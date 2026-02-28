# Audit Éco-Conception - GreenRoots

> **Date de l'audit** : 2 février 2026
> **Version application** : Branch `feature/eco-conception`
> **Référentiels utilisés** : WSG (W3C), RGESN 2024, RWEB GreenIT v5

---

## Table des matières

1. [Contexte et enjeux](#1-contexte-et-enjeux)
2. [Référentiels d'éco-conception](#2-référentiels-déco-conception)
3. [État des lieux - Frontend](#3-état-des-lieux---frontend)
4. [État des lieux - Backend](#4-état-des-lieux---backend)
5. [Analyse par catégorie WSG](#5-analyse-par-catégorie-wsg)
6. [Conformité RGESN](#6-conformité-rgesn)
7. [Métriques et benchmarks](#7-métriques-et-benchmarks)
8. [Plan d'action priorisé](#8-plan-daction-priorisé)
9. [Ressources et outils](#9-ressources-et-outils)

---

## 1. Contexte et enjeux

### 1.1 Impact environnemental du numérique

Le numérique représente aujourd'hui **4% des émissions mondiales de gaz à effet de serre**, soit l'équivalent du secteur aérien. Les projections indiquent que les technologies de communication émettront plus de carbone d'ici 2025 que n'importe quel pays à l'exception de la Chine, l'Inde et les États-Unis.

**Statistiques clés (Web Almanac 2024)** :
- Page médiane : **2.5 MB** (desktop), **2 MB** (mobile)
- 90e percentile : **8 MB** (desktop), **7.2 MB** (mobile)
- Émissions moyennes : **0.37g CO2** par visite (50e percentile)
- Émissions élevées : **1.47g CO2** par visite (90e percentile)

**Objectif recommandé** : Pages < 1 MB, idéalement ~500 KB

### 1.2 Composition typique du poids d'une page

| Ressource | Part du poids total | 90e percentile |
|-----------|---------------------|----------------|
| **Images** | ~50% | 4,910 KB |
| **JavaScript** | ~20% | 1,834 KB |
| **Fonts** | ~5% | 458 KB |
| **CSS** | ~3% | 269 KB |
| **HTML** | ~2% | 146 KB |

### 1.3 Code inutilisé (problème majeur)

| Type | 90e percentile Desktop | 90e percentile Mobile |
|------|------------------------|----------------------|
| **CSS inutilisé** | 225 KB | 212 KB |
| **JavaScript inutilisé** | 907 KB | 812 KB |

---

## 2. Référentiels d'éco-conception

### 2.1 Web Sustainability Guidelines (WSG) - W3C

Les [Web Sustainability Guidelines](https://sustainablewebdesign.org/guidelines/) sont développées par le W3C Sustainable Web Interest Group. Publication officielle prévue en 2026.

**Structure en 4 catégories (94 recommandations)** :

| Catégorie | Guidelines | Focus |
|-----------|------------|-------|
| **UX Design** | 2.1 → 2.21 | Parcours utilisateur, contenu, accessibilité |
| **Web Development** | 3.1 → 3.20 | Performance, code, assets |
| **Hosting & Infrastructure** | 4.1 → 4.12 | Serveurs, données, énergie |
| **Business Strategy** | 5.1 → 5.27 | Gouvernance, mesure, transparence |

### 2.2 RGESN 2024 - France

Le [Référentiel Général d'Écoconception de Services Numériques](https://ecoresponsable.numerique.gouv.fr/publications/referentiel-general-ecoconception/) est élaboré par l'ARCEP, l'Arcom, l'ADEME et la DINUM.

**78 critères répartis en catégories** :
- Stratégie
- Spécifications
- Architecture
- UX/UI
- Contenus
- Frontend
- Backend
- Hébergement
- **Algorithmie/IA** (nouveau en 2024)

**3 niveaux de priorité** : Prioritaire, Recommandé, Modéré

### 2.3 RWEB GreenIT v5 (2025)

Le [référentiel RWEB](https://rweb.greenit.fr/) propose **119 bonnes pratiques** organisées selon le cycle de vie projet :

1. **Spécifications** - Définir les besoins réels
2. **Conception** - Architecturer la solution
3. **Réalisation** - Développer/fabriquer
4. **Production** - Mettre en service
5. **Utilisation** - Phase d'exploitation
6. **Support/maintenance** - Évolutions et corrections
7. **Fin de vie** - Déclassement

> **Fait notable** : 45% des fonctionnalités demandées ne sont jamais utilisées (source : RWEB)

---

## 3. État des lieux - Frontend

### 3.1 Stack technique

| Technologie | Version | Évaluation éco |
|-------------|---------|----------------|
| React | 19.2.0 | Moderne, performant |
| Vite | 7.2.4 | ESM natif, build rapide |
| TanStack Router | 1.139.0 | Lazy loading natif |
| TanStack Query | 5.90.10 | Cache intelligent |
| Zustand | 5.0.9 | Ultra léger (2.3 KB) |
| Tailwind CSS | 4.1.17 | JIT, purge automatique |

### 3.2 Images - ✅ EXCELLENT

**Implémentation actuelle** :

```
CloudinaryImage.tsx + cloudinary.ts
├── Format auto (WebP/AVIF)
├── Quality auto
├── srcSet responsive (0.5x, 1x, 1.5x, 2x)
├── Lazy loading par défaut
├── Eager pour LCP (Above-the-fold)
└── CDN edge delivery
```

**Conformité** :
- [x] WSG 2.11 - Optimize media ✅
- [x] RGESN - Optimiser les images ✅
- [x] RWEB - Choisir des formats d'image adaptés ✅

**Amélioration possible** :
- [ ] Ajouter LQIP (Low Quality Image Placeholder) pour réduire CLS

### 3.3 Fonts - ✅ BON

**Implémentation actuelle** :
- Format WOFF2 (optimal)
- 5 variantes Montserrat (~90 KB total)
- 2 fonts critiques preloadées
- `font-display: fallback`

**Conformité** :
- [x] WSG 2.13 - Use optimized typography ✅
- [x] RWEB - Utiliser uniquement les portions nécessaires 🟡

**Amélioration** :
- [ ] Réduire à 3 variantes (400, 600, 700) - économie ~36 KB
- [ ] Considérer le subsetting des caractères non utilisés

### 3.4 JavaScript - ✅ BON

**Implémentation actuelle** :

```javascript
// vite.config.ts - Manual chunks
manualChunks: {
  'react-vendor': ['react', 'react-dom'],      // ~64 KB gzip
  'query-vendor': ['@tanstack/react-query'],   // ~37 KB
  'router-vendor': ['@tanstack/react-router'], // ~45 KB
  'utils-vendor': ['axios', 'clsx', '...'],    // ~20 KB
}
```

**Routes lazy-loaded** : Toutes les routes utilisent `lazyRouteComponent()`

**Bundle estimé initial** : ~300-400 KB (acceptable)

**Conformité** :
- [x] WSG 3.x - Code splitting ✅
- [x] RWEB - Architecture modulaire ✅
- [x] RGESN - Limiter le poids des ressources ✅

**Points de vigilance** :
| Dépendance | Taille | Chargement |
|------------|--------|------------|
| @stripe/stripe-js | 80 KB | Lazy (checkout) ✅ |
| lucide-react | ~30 KB | Tree-shakeable ✅ |
| @tanstack/react-table | 30 KB | Lazy (admin) ✅ |

### 3.5 CSS - ✅ EXCELLENT

**Implémentation actuelle** :
- Critical CSS inline (Critters plugin)
- Tailwind JIT (purge automatique)
- Variables CSS oklch() (moderne)
- Dark mode préparé (variables `.dark`)

**Conformité** :
- [x] WSG 3.x - Minimize CSS ✅
- [x] RWEB - Éviter le CSS inutilisé ✅

### 3.6 Caching côté client - 🟡 AMÉLIORABLE

**Implémentation actuelle** :

```typescript
// QueryClient defaults
staleTime: 1 * 60 * 1000,      // 1 minute
refetchOnWindowFocus: true,

// Données statiques
countries: staleTime 1h
```

**Persistance locale** :
- Cart : localStorage (24h expiration)
- Auth : localStorage

**Manquant** :
- [ ] Service Worker (cache offline)
- [ ] Stratégie cache-first pour assets statiques

---

## 4. État des lieux - Backend

### 4.1 Stack technique

| Technologie | Version | Évaluation éco |
|-------------|---------|----------------|
| NestJS | 11.0.1 | Architecture modulaire |
| Prisma | 7.0.0 | ORM optimisé |
| PostgreSQL | + PostGIS | Robuste |
| SWC | 1.10.7 | Compilation rapide |

### 4.2 Base de données - ✅ BON

**Patterns implémentés** :

```typescript
// Eager loading systématique
const TREE_INCLUDES = {
  stock: true,
  category: true,
  location: { include: { country: true } },
} as const;

// Select sécurisé (exclut password)
private readonly safeSelect = {
  id: true, email: true, firstName: true,
  // password EXCLUDED
};

// Requêtes parallèles
const [data, total] = await Promise.all([
  this.prisma.order.findMany({...}),
  this.prisma.order.count({...}),
]);
```

**Conformité** :
- [x] RWEB - Éviter les requêtes N+1 ✅
- [x] RGESN - Optimiser les requêtes ✅

**Améliorations** :
- [ ] Ajouter indices Prisma explicites

```prisma
model Order {
  @@index([userId])
  @@index([paymentStatus])
}

model Tree {
  @@index([categoryId])
  @@index([locationId])
}

model Location {
  @@index([coordinates])  // Index spatial PostGIS
}
```

### 4.3 API - 🟡 AMÉLIORABLE

**Implémentation actuelle** :
- Pagination : `skip/take` + count parallèle ✅
- Validation : DTOs stricts + ValidationPipe ✅
- Rate limiting : 60 req/min global ✅

**Manquant critique** :

```typescript
// ❌ Pas de headers Cache-Control
// ❌ Pas de compression explicite
// ❌ Pas d'ETags
```

### 4.4 Uploads - ✅ EXCELLENT

**Architecture Cloudinary** :
- Upload client-side avec signature backend
- Zéro bande passante serveur pour les images
- Transformations à la volée

---

## 5. Analyse par catégorie WSG

### 5.1 User Experience Design

| Guideline | Description | État | Détail |
|-----------|-------------|------|--------|
| 2.1 | Examine external factors | ✅ | Contexte projet défini |
| 2.4 | Minimize non-essential content | ✅ | Design épuré |
| 2.5 | Structure navigation | ✅ | TanStack Router, structure claire |
| 2.7 | Avoid manipulation | ✅ | Pas de dark patterns |
| 2.9 | Use design systems | ✅ | Shadcn/UI + Tailwind |
| 2.10 | Clear, inclusive content | ✅ | Textes concis |
| 2.11 | Optimize media | ✅ | Cloudinary optimisé |
| 2.12 | Proportionate animation | ✅ | Animations minimales |
| 2.13 | Optimized typography | 🟡 | Réduire variantes fonts |
| 2.15 | Minimal web forms | ✅ | Formulaires concis |

### 5.2 Web Development

| Guideline | Description | État | Détail |
|-----------|-------------|------|--------|
| 3.1 | Set performance goals | 🟡 | Métriques à définir |
| 3.2 | Use native features | ✅ | ES2020+, pas de polyfills |
| 3.3 | Code splitting | ✅ | Routes lazy, manual chunks |
| 3.4 | Minimize dependencies | ✅ | Zustand vs Redux |
| 3.5 | Tree shaking | ✅ | ESM natif |
| 3.6 | Minification | ✅ | esbuild |
| 3.7 | Compression | 🔴 | Backend non configuré |
| 3.8 | HTTP caching | 🔴 | Headers manquants |
| 3.9 | Service Worker | 🔴 | Non implémenté |
| 3.10 | Image optimization | ✅ | Cloudinary excellent |

### 5.3 Hosting & Infrastructure

| Guideline | Description | État | Détail |
|-----------|-------------|------|--------|
| 4.1 | Sustainable hosting | ❓ | À vérifier |
| 4.2 | Optimize server efficiency | 🟡 | Pas de Redis cache |
| 4.3 | Use CDN | 🟡 | Images oui, assets JS/CSS non |
| 4.4 | Optimize data transfer | 🔴 | Compression manquante |
| 4.5 | Efficient data storage | ✅ | Prisma optimisé |

### 5.4 Business Strategy

| Guideline | Description | État | Détail |
|-----------|-------------|------|--------|
| 5.1 | Ethical strategy | ✅ | Projet éco-responsable |
| 5.5 | Measure impact | 🔴 | Pas de mesure CO2 |
| 5.6 | Report sustainability | 🔴 | Pas de reporting |

---

## 6. Conformité RGESN

### 6.1 Critères prioritaires

| # | Critère | État | Action |
|---|---------|------|--------|
| 1.1 | Évaluer la pertinence du service | ✅ | Besoin réel identifié |
| 2.1 | Quantifier les besoins métier | ✅ | Fonctionnalités définies |
| 4.1 | Favoriser un design simple | ✅ | UI minimaliste |
| 5.1 | Optimiser les images | ✅ | Cloudinary |
| 5.2 | Lazy loading médias | ✅ | Implémenté |
| 6.1 | Minifier les ressources | ✅ | Vite build |
| 6.2 | Compresser les transferts | 🔴 | À implémenter |
| 6.3 | Mettre en cache | 🔴 | Headers manquants |
| 7.1 | Optimiser requêtes BDD | ✅ | Prisma patterns |
| 8.1 | Hébergement éco-responsable | ❓ | À vérifier |

### 6.2 Score estimé

| Catégorie | Conforme | Partiel | Non conforme |
|-----------|----------|---------|--------------|
| Stratégie | 3/3 | 0 | 0 |
| Spécifications | 4/5 | 1 | 0 |
| Architecture | 3/5 | 2 | 0 |
| UX/UI | 6/7 | 1 | 0 |
| Contenus | 4/5 | 1 | 0 |
| Frontend | 8/12 | 2 | 2 |
| Backend | 6/10 | 2 | 2 |
| Hébergement | 2/6 | 2 | 2 |

**Score global estimé : ~65-70%** (bon point de départ)

---

## 7. Métriques et benchmarks

### 7.1 Objectifs de performance

| Métrique | Actuel (estimé) | Cible | Outil |
|----------|-----------------|-------|-------|
| Lighthouse Performance | ~75-85 | > 90 | Chrome DevTools |
| First Contentful Paint | ~1.5-2s | < 1.8s | WebPageTest |
| Largest Contentful Paint | ~2-3s | < 2.5s | WebPageTest |
| Total Blocking Time | ~200-400ms | < 200ms | Lighthouse |
| Cumulative Layout Shift | ~0.05-0.1 | < 0.1 | Lighthouse |
| Page Weight (initial) | ~400-600 KB | < 500 KB | Network tab |
| Page Weight (total) | ~1-1.5 MB | < 1 MB | Network tab |

### 7.2 Objectifs carbone

| Métrique | Actuel (estimé) | Cible | Outil |
|----------|-----------------|-------|-------|
| CO2 par page | ~0.4-0.6g | < 0.5g | websitecarbon.com |
| Énergie par visite | ~0.3-0.5 kWh | < 0.3 kWh | Ecograder |
| Green hosting | ❓ | ✅ | Green Web Foundation |

### 7.3 Objectifs techniques

| Métrique | Actuel | Cible |
|----------|--------|-------|
| Cache Hit Rate | N/A | > 80% |
| Requêtes par page | ~15-25 | < 15 |
| CSS inutilisé | ~50 KB | < 20 KB |
| JS inutilisé | ~100 KB | < 50 KB |

---

## 8. Plan d'action priorisé

### 8.1 Priorité 1 - Impact élevé, effort faible

#### A. Ajouter la compression serveur

```bash
# Installation
npm install compression @types/compression
```

```typescript
// main.ts
import compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { rawBody: true });

  // Compression gzip
  app.use(compression({
    threshold: 1024,  // Compresser si > 1KB
    level: 6,         // Niveau de compression
  }));

  // ...
}
```

**Impact** : Réduction 60-80% taille transferts

---

#### B. Ajouter les headers Cache-Control

```typescript
// src/common/interceptors/cache-headers.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class CacheHeadersInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();

    // Données très statiques (pays, catégories)
    if (request.url.match(/\/(countries|categories)/) && request.method === 'GET') {
      response.setHeader('Cache-Control', 'public, max-age=86400, stale-while-revalidate=43200');
    }
    // Catalogue arbres (change peu)
    else if (request.url.match(/\/trees/) && request.method === 'GET') {
      response.setHeader('Cache-Control', 'public, max-age=300, stale-while-revalidate=600');
    }
    // Données utilisateur (privées)
    else if (request.url.match(/\/(orders|users|cart)/)) {
      response.setHeader('Cache-Control', 'private, no-cache');
    }

    return next.handle();
  }
}
```

```typescript
// app.module.ts
import { APP_INTERCEPTOR } from '@nestjs/core';

providers: [
  {
    provide: APP_INTERCEPTOR,
    useClass: CacheHeadersInterceptor,
  },
]
```

**Impact** : Réduction drastique requêtes réseau

---

#### C. Réduire les variantes de fonts

```css
/* Supprimer dans index.css ou public/fonts */
/* GARDER : 400 (regular), 600 (semibold), 700 (bold) */
/* SUPPRIMER : 300 (light), 500 (medium) */
```

**Économie** : ~36 KB (40% des fonts)

---

### 8.2 Priorité 2 - Impact moyen

#### D. Service Worker pour cache offline

```bash
npm install -D vite-plugin-pwa
```

```typescript
// vite.config.ts
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    // ...
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,woff2}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/res\.cloudinary\.com\/.*/i,
            handler: 'CacheFirst',
            options: {
              cacheName: 'cloudinary-images',
              expiration: {
                maxEntries: 100,
                maxAgeSeconds: 60 * 60 * 24 * 30, // 30 jours
              },
            },
          },
          {
            urlPattern: /\/api\/(countries|categories)/,
            handler: 'StaleWhileRevalidate',
            options: {
              cacheName: 'api-static',
              expiration: {
                maxEntries: 10,
                maxAgeSeconds: 60 * 60 * 24, // 24h
              },
            },
          },
        ],
      },
    }),
  ],
});
```

---

#### E. Ajouter LQIP (Low Quality Image Placeholder)

```typescript
// src/lib/cloudinary.ts

export function getLqipUrl(publicId: string): string {
  return cld
    .image(publicId)
    .resize(scale().width(20))
    .quality('auto:low')
    .effect(blur().strength(10))
    .format('auto')
    .toURL();
}

export function getImageWithLqip(publicId: string, size: ImageSize) {
  return {
    src: getCloudinaryUrl(publicId, size),
    lqip: getLqipUrl(publicId),
    srcSet: getOptimizedSrcSet(publicId, size),
  };
}
```

```tsx
// CloudinaryImage.tsx - Ajouter support LQIP
const [loaded, setLoaded] = useState(false);

<div className="relative">
  {!loaded && lqip && (
    <img
      src={lqip}
      className="absolute inset-0 blur-sm"
      aria-hidden="true"
    />
  )}
  <img
    src={src}
    onLoad={() => setLoaded(true)}
    className={cn(!loaded && 'opacity-0')}
    // ...
  />
</div>
```

---

#### F. Indices Prisma

```prisma
// prisma/schema.prisma

model Order {
  // ... fields

  @@index([userId])
  @@index([paymentStatus])
  @@index([createdAt])
}

model Tree {
  // ... fields

  @@index([categoryId])
  @@index([locationId])
  @@index([name])
}

model Location {
  // ... fields

  @@index([countryId])
  // Pour requêtes géospatiales
  @@index([coordinates], type: Gist)
}

model CartItem {
  // ... fields

  @@index([cartId])
}

model OrderItem {
  // ... fields

  @@index([orderId])
}
```

```bash
npx prisma migrate dev --name add_performance_indices
```

---

### 8.3 Priorité 3 - Optimisations avancées

#### G. Dark mode UI

```tsx
// src/components/atoms/ThemeToggle.tsx
import { Moon, Sun } from 'lucide-react';
import { useEffect, useState } from 'react';

export function ThemeToggle() {
  const [isDark, setIsDark] = useState(false);

  useEffect(() => {
    const saved = localStorage.getItem('theme');
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    setIsDark(saved === 'dark' || (!saved && prefersDark));
  }, []);

  useEffect(() => {
    document.documentElement.classList.toggle('dark', isDark);
    localStorage.setItem('theme', isDark ? 'dark' : 'light');
  }, [isDark]);

  return (
    <button
      onClick={() => setIsDark(!isDark)}
      className="p-2 rounded-md hover:bg-muted"
      aria-label={isDark ? 'Activer le mode clair' : 'Activer le mode sombre'}
    >
      {isDark ? <Sun size={20} /> : <Moon size={20} />}
    </button>
  );
}
```

**Bénéfice** : Réduction 30-60% consommation écran OLED

---

#### H. Redis cache (scaling)

```bash
npm install @nestjs/cache-manager cache-manager cache-manager-redis-store redis
```

```typescript
// app.module.ts
import { CacheModule } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true,
      useFactory: async () => ({
        store: await redisStore({
          socket: {
            host: process.env.REDIS_HOST,
            port: parseInt(process.env.REDIS_PORT),
          },
          ttl: 300, // 5 minutes default
        }),
      }),
    }),
  ],
})
```

```typescript
// trees.service.ts
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager';

@Injectable()
export class TreesService {
  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,
    private prisma: PrismaService,
  ) {}

  async findAll(query: GetTreesDto) {
    const cacheKey = `trees:${JSON.stringify(query)}`;

    // Check cache
    const cached = await this.cache.get(cacheKey);
    if (cached) return cached;

    // Query DB
    const result = await this.prisma.tree.findMany({...});

    // Cache result
    await this.cache.set(cacheKey, result, 300000); // 5 min

    return result;
  }
}
```

---

#### I. Préchargement intelligent des routes

```tsx
// src/components/molecules/TreeCard.tsx
import { useRouter } from '@tanstack/react-router';

function TreeCard({ tree }) {
  const router = useRouter();

  const handleMouseEnter = () => {
    // Précharger la page détail au survol
    router.preloadRoute({
      to: '/trees/$id',
      params: { id: tree.id.toString() },
    });
  };

  return (
    <Link
      to="/trees/$id"
      params={{ id: tree.id.toString() }}
      onMouseEnter={handleMouseEnter}
    >
      {/* ... */}
    </Link>
  );
}
```

---

### 8.4 Récapitulatif des priorités

| Priorité | Action | Impact CO2 | Effort | Délai |
|----------|--------|------------|--------|-------|
| **P1** | Compression serveur | -30% transferts | Faible | 1h |
| **P1** | Cache-Control headers | -50% requêtes | Faible | 2h |
| **P1** | Réduire fonts | -36 KB | Faible | 30min |
| **P2** | Service Worker | -40% revisites | Moyen | 4h |
| **P2** | LQIP images | Meilleur UX | Moyen | 3h |
| **P2** | Indices Prisma | -30% latence BDD | Faible | 1h |
| **P3** | Dark mode | -30% écran OLED | Moyen | 2h |
| **P3** | Redis cache | Scaling | Élevé | 8h |
| **P3** | Préchargement routes | Meilleur UX | Faible | 1h |

---

## 9. Ressources et outils

### 9.1 Outils de mesure

| Outil | Usage | URL |
|-------|-------|-----|
| **Website Carbon** | Empreinte carbone | [websitecarbon.com](https://www.websitecarbon.com/) |
| **Ecograder** | Audit complet | [ecograder.com](https://ecograder.com/) |
| **Digital Beacon** | Analyse détaillée | [digitalbeacon.co](https://digitalbeacon.co/) |
| **Greenoco** | Audit bulk (75 pages) | [greenoco.io](https://greenoco.io/) |
| **Lighthouse** | Performance | Chrome DevTools |
| **WebPageTest** | Métriques avancées | [webpagetest.org](https://www.webpagetest.org/) |
| **Green Web Check** | Hébergement vert | [thegreenwebfoundation.org](https://www.thegreenwebfoundation.org/green-web-check/) |

### 9.2 Référentiels officiels

| Référentiel | Critères | URL |
|-------------|----------|-----|
| **WSG W3C** | 94 guidelines | [sustainablewebdesign.org/guidelines](https://sustainablewebdesign.org/guidelines/) |
| **RGESN 2024** | 78 critères | [ecoresponsable.numerique.gouv.fr](https://ecoresponsable.numerique.gouv.fr/publications/referentiel-general-ecoconception/) |
| **RWEB GreenIT** | 119 bonnes pratiques | [rweb.greenit.fr](https://rweb.greenit.fr/) |
| **Sustainable Web Manifesto** | 6 principes | [sustainablewebmanifesto.com](https://www.sustainablewebmanifesto.com/) |

### 9.3 Documentation technique

- [MDN - Introduction to Web Sustainability](https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/)
- [HTTP Archive - Web Almanac 2024 Sustainability](https://almanac.httparchive.org/en/2024/sustainability)
- [GitHub - cnumr/best-practices](https://github.com/cnumr/best-practices)

### 9.4 Formations et certifications

- [Formation GreenIT - Certification RGESN/RWEB](https://www.greenit.fr/formations/)
- [Opquast - Certification qualité web](https://www.opquast.com/)

---

## Annexes

### A. Checklist rapide pré-déploiement

```markdown
## Images
- [ ] Format WebP/AVIF avec fallback
- [ ] Lazy loading activé
- [ ] srcSet responsive
- [ ] Compression quality auto

## Fonts
- [ ] Format WOFF2
- [ ] Preload fonts critiques
- [ ] Maximum 3-4 variantes
- [ ] font-display: swap/fallback

## JavaScript
- [ ] Code splitting par route
- [ ] Tree shaking activé
- [ ] Dépendances lourdes en lazy
- [ ] DevTools exclus en prod

## CSS
- [ ] Critical CSS inline
- [ ] Purge CSS inutilisé
- [ ] Minification activée

## Backend
- [ ] Compression gzip/brotli
- [ ] Headers Cache-Control
- [ ] Pagination implémentée
- [ ] Requêtes BDD optimisées

## Hébergement
- [ ] CDN pour assets statiques
- [ ] HTTP/2 ou HTTP/3
- [ ] Hébergeur éco-responsable vérifié
```

### B. Commandes utiles

```bash
# Analyser le bundle
npx vite-bundle-visualizer

# Audit Lighthouse CLI
npx lighthouse https://greenroots.com --output=json --output-path=./lighthouse.json

# Tester la compression
curl -H "Accept-Encoding: gzip" -I https://api.greenroots.com/api/trees

# Vérifier les headers cache
curl -I https://api.greenroots.com/api/countries

# Analyser le CSS inutilisé
npx purgecss --css dist/assets/*.css --content dist/**/*.html --output purged/
```

---

**Document généré le 2 février 2026**
**Dernière mise à jour : v1.0**

---

## Sources

- [Web Sustainability Guidelines - W3C](https://sustainablewebdesign.org/guidelines/)
- [RGESN 2024 - Gouvernement français](https://ecoresponsable.numerique.gouv.fr/publications/referentiel-general-ecoconception/)
- [RWEB GreenIT v5](https://rweb.greenit.fr/)
- [HTTP Archive - Web Almanac 2024 Sustainability](https://almanac.httparchive.org/en/2024/sustainability)
- [GreenIT - 115 bonnes pratiques](https://www.greenit.fr/2025/06/23/le-collectif-green-it-publie-la-5eme-edition-du-referentiel-ecoconception-web-les-115-bonnes-pratiques/)
- [Mightybytes - Sustainable Web Design](https://www.mightybytes.com/insights/sustainable-web-design/)
- [MDN - Introduction to Web Sustainability](https://developer.mozilla.org/en-US/blog/introduction-to-web-sustainability/)
