# Feature 06 — Cache Redis API (pays et catégories)

## Priorité : P2 | Estimation : ~2h

## Contexte
Redis était déjà utilisé dans GreenRoots pour la **blacklist de tokens JWT** et la **protection brute force** (compteur de tentatives de login). Mais il n'y avait aucun cache applicatif sur les routes API : chaque requête allait systématiquement en base de données, même pour des données qui changent très rarement.

Les pays (facturation et plantation) et les catégories d'arbres sont des données quasi-statiques, idéales pour un cache Redis.

## Pattern utilisé : Cache-aside (Lazy loading)

Le **cache-aside** est le pattern de cache le plus classique. Le code vérifie d'abord Redis, puis va en base si le cache est vide, et remplit le cache au passage :

```
1. Le client appelle GET /api/countries/billing
2. Le service vérifie Redis : redis.get("countries:billing")
   → Si trouvé (cache hit) : retourne les données depuis Redis (pas de requête SQL)
   → Si absent (cache miss) : requête SQL → stocke le résultat dans Redis → retourne
3. Les requêtes suivantes sont servies depuis Redis (beaucoup plus rapide)
```

### Pourquoi cache-aside et pas cache-through ?
- **Simplicité** : le code contrôle explicitement quand lire/écrire le cache
- **Granularité** : on peut choisir précisément quelles données cacher et avec quel TTL
- **Pas de middleware complexe** : pas besoin d'un `CacheInterceptor` NestJS qui cacherait tout aveuglément

## Modifications techniques

### `app/backend/src/countries/countries.service.ts`

#### Constantes
```typescript
const CACHE_TTL = 86400; // 24 hours
const CACHE_KEY_BILLING = 'countries:billing';
const CACHE_KEY_PLANTING = 'countries:planting';
```

#### Injection de RedisService
- `RedisService` ajouté dans le constructeur (injection de dépendances NestJS)
- `Logger` NestJS pour tracer les cache hit/miss en mode debug

#### `findBillingCountries()`
1. Vérifie `redis.get(CACHE_KEY_BILLING)`
2. Si cache hit : `JSON.parse()` et retour immédiat (log debug "Cache hit")
3. Si cache miss : requête Prisma → `redis.set(key, JSON.stringify(data), TTL)` → retour (log debug "Cache miss — cached for 24h")

#### `findPlantingCountries()`
- Même pattern que `findBillingCountries()` avec la clé `countries:planting`

#### Pourquoi pas d'invalidation pour les pays ?
Les pays ne changent (quasiment) jamais. Il n'y a pas de CRUD admin sur les pays — ils sont chargés via le seed Prisma. Le TTL de 24h suffit : si on ajoute un pays au seed, il sera visible sous 24h maximum.

### `app/backend/src/categories/categories.service.ts`

#### Constantes
```typescript
const CACHE_TTL = 3600; // 1 hour
const CACHE_KEY_LIST = 'categories:list';
const CACHE_KEY_PREFIX = 'categories:';
```

#### TTL plus court (1h vs 24h pour les pays)
Les catégories sont modifiables par l'admin (CRUD complet). Un TTL plus court réduit la fenêtre d'incohérence si l'invalidation échouait pour une raison quelconque.

#### Cache sur la lecture
- `findAll()` : cache la liste complète (`categories:list`)
- `findOne(id)` : cache chaque catégorie individuellement (`categories:42`)

#### Invalidation sur l'écriture
Méthode privée `invalidateCache(id?: number)` appelée après chaque écriture :
- `create()` → invalide la liste (le nouvel élément doit y apparaître)
- `update(id)` → invalide la liste + la catégorie spécifique
- `remove(id)` → invalide la liste + la catégorie spécifique

```typescript
private async invalidateCache(id?: number) {
  await this.redis.del(CACHE_KEY_LIST);       // Toujours invalider la liste
  if (id) {
    await this.redis.del(`${CACHE_KEY_PREFIX}${id}`);  // Invalider l'item si applicable
  }
}
```

### Dégradation gracieuse
Les appels Redis (`get`, `set`, `del`) sont déjà wrappés dans le `RedisService` existant avec des try/catch. Si Redis est indisponible :
- `get()` retourne `null` → le code va en base (cache miss)
- `set()` échoue silencieusement → la donnée n'est pas cachée mais le service fonctionne
- `del()` échoue silencieusement → le cache expirera naturellement via le TTL

L'application ne crashe jamais à cause de Redis — elle fonctionne simplement sans cache.

## Tests mis à jour

### `countries.service.spec.ts`
- Ajout du mock `RedisService` avec `get: jest.fn().mockResolvedValue(null)` pour simuler un cache miss
- Les tests existants continuent de fonctionner (le cache retourne toujours null → le code va toujours en base)

### `categories.service.spec.ts`
- Même ajout du mock `RedisService`
- Tests existants (findAll, findOne, create, conflits) fonctionnent toujours

## Ce qui n'a PAS été caché (et pourquoi)

| Route | Raison |
|---|---|
| `GET /trees` (catalogue) | Les arbres ont du stock variable, des filtres utilisateur — le cache serait invalidé trop souvent pour être utile |
| `GET /orders` | Données personnelles et dynamiques, pas de bénéfice |
| `GET /auth/me` | Profil utilisateur, change à chaque modification |
| `GET /cart` | Panier personnel et très dynamique |

## Branche
`feature/redis-cache-api`

## Fichiers modifiés
- `app/backend/src/countries/countries.service.ts`
- `app/backend/src/countries/countries.service.spec.ts`
- `app/backend/src/categories/categories.service.ts`
- `app/backend/src/categories/categories.service.spec.ts`

## Sections du dossier CDA concernées
- §7.3 (accès aux données — cache Redis NoSQL clé/valeur, pattern cache-aside)
- §6.2 (choix techniques — Redis comme cache applicatif, justification des TTL)
- §5.2 (architecture — Redis dans l'architecture 3 tiers, cache layer)
- §8.6 (éco-conception — réduction des requêtes SQL redondantes)
