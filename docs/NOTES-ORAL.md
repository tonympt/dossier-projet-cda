# Notes pour l'oral — Points à maîtriser

> Ce fichier recense les concepts et questions potentielles du jury identifiés pendant la rédaction du dossier.

## Architecture

### Architecture 3 tiers
- Couche présentation (Nginx + SPA React dans le navigateur)
- Couche métier (API REST NestJS)
- Couche données (PostgreSQL + Redis)
- Chaque couche ne communique qu'avec la suivante

### Fonctionnement d'une SPA
- Le navigateur reçoit un `index.html` quasi vide + des fichiers JS (React compilé par Vite)
- Le navigateur exécute le JS → React génère le HTML dynamiquement
- Le routage est géré côté client (TanStack Router), sans rechargement de page
- Les seules requêtes serveur sont les appels API (/api)

### SPA (CSR) vs SSR vs Serverless — ne pas confondre
- **SPA / CSR** (Client-Side Rendering) : le navigateur reçoit un HTML quasi vide + un bundle JS → React construit le DOM. GreenRoots = CSR. Attention : avec le code splitting, le JS n'est **pas** chargé en une seule fois — seul le chunk de la route demandée est chargé, les autres en lazy loading
- **SSR** (Server-Side Rendering) : le serveur génère le HTML **complet** à chaque requête → le navigateur l'affiche immédiatement (pas de page blanche). Puis le JS se charge et **hydrate** le HTML (= attache les event listeners pour rendre la page interactive). Frameworks : Next.js, Nuxt
- **Serverless** : modèle d'**infrastructure** (AWS Lambda, Vercel Functions, Cloudflare Workers) — pas de serveur à gérer. Le code s'exécute **à la demande** : une requête arrive → la fonction démarre, s'exécute, s'éteint. 0 requête = 0 coût. **≠ mode de rendu** — on peut faire du SSR en serverless (Next.js sur Vercel)
- **Deux axes différents** : rendu (CSR vs SSR) et infrastructure (serveur classique vs serverless). Ne pas mélanger
- **GreenRoots** : CSR (SPA React) + serveur classique (VPS Hetzner avec Docker)
- Si le jury demande : "GreenRoots est une SPA en client-side rendering sur un serveur classique. SSR c'est le rendu côté serveur avec hydratation (Next.js). Serverless c'est un modèle d'infrastructure à la demande — c'est un axe différent du rendu."

### Rôle de Nginx
- Double rôle : sert les fichiers statiques de React ET redirige les appels /api vers NestJS
- Tout passe par Nginx, le navigateur ne parle jamais directement au backend

### Vite — pas juste un builder
- Vite = **outil de développement complet** : serveur de dev avec HMR (Hot Module Replacement) ultra-rapide + bundler de production (basé sur Rollup)
- CRA (Create React App) est déprécié, Vite l'a remplacé comme standard
- En dev : pas de bundle, les modules ES sont servis directement au navigateur → démarrage quasi instantané
- En prod : Rollup optimise le bundle (tree-shaking, code splitting, manual chunks)
- Si le jury demande "Vite c'est un bundler ?" → "C'est plus qu'un bundler : c'est un outil de dev complet avec un serveur HMR en dev et Rollup en prod."

### TanStack Router vs React Router — pourquoi TanStack
- **React Router** : routes en strings non vérifiées (`<Link to="/ctalogue">` → typo, pas d'erreur TS). `useParams()` retourne `string | undefined` — tout est non typé
- **TanStack Router** : routes **type-safe** — vérifiées à la compilation. Route inconnue = erreur TS. Params typés automatiquement (`$id` → TypeScript sait que c'est un string). Autocomplétion des routes dans l'IDE
- Aussi : code splitting natif via `lazyRouteComponent` + `import()` dynamique
- Si le jury demande : "TanStack Router est type-safe — routes, params et search params vérifiés à la compilation. Une typo dans une route casse au build, pas au runtime."

### Next.js vs SPA — pourquoi pas Next.js
- Next.js = framework fullstack React (mélange frontend + backend : Server Components, API routes)
- GreenRoots = **architecture découplée** : SPA React pure + API REST NestJS séparée
- On ne voulait pas un framework qui mélange les responsabilités — on voulait un frontend pur qui communique avec une API indépendante
- Argument principal : séparation des responsabilités + architecture qu'on maîtrisait déjà
- Si le jury demande : "On a choisi une SPA pure pour avoir une architecture découplée — le frontend et le backend évoluent indépendamment, chacun avec son framework dédié."

### Architecture hexagonale vs modulaire NestJS
- Hexagonale = ports & adapters, interfaces intermédiaires strictes entre métier et infrastructure
- GreenRoots = architecture modulaire NestJS classique (Controller → Service → Prisma directement)
- Pas d'interfaces intermédiaires = pas hexagonale au sens strict, et c'est OK

### Injection de dépendances
- Ne pas instancier soi-même ses dépendances — le conteneur IoC de NestJS s'en charge
- `constructor(private readonly prisma: PrismaService)` → NestJS fournit l'instance
- Même principe que `@Autowired` dans Spring Boot
- Avantage : découplage, testabilité (on peut mocker)

## Injection de dépendances / Modules NestJS

### Conteneur IoC (Inversion of Control)
- **Sans IoC** : c'est toi qui crées les dépendances (`new OrdersService(new PrismaService()...)`) → couplage fort
- **Avec IoC** : NestJS crée et injecte les instances. Tu déclares ce dont tu as besoin dans le constructeur, NestJS résout
- `@Injectable()` sur un service = "NestJS peut m'instancier et m'injecter"
- Au démarrage, NestJS lit les modules → instancie les providers (singleton par défaut) → injecte dans les constructeurs via le type

### Les 4 champs de `@Module()`
- **`imports`** : les modules dont tu dépends (pour accéder à leurs exports). Les modules `@Global()` n'ont pas besoin d'être importés
- **`controllers`** : les controllers qui gèrent les routes HTTP du module
- **`providers`** : les services `@Injectable()` instanciés et disponibles **dans ce module** (singleton par défaut)
- **`exports`** : les services partagés avec les modules qui importent le tien. **Encapsulation** — ce qui n'est pas exporté reste privé
- Si le jury demande : "Le module est une unité d'encapsulation. Les providers sont les services en singleton. Les exports contrôlent ce qui est visible. Les imports accèdent aux exports des autres modules."

## Design Patterns

### Singleton
- PrismaService, RedisService : une seule instance partagée dans toute l'app
- Comportement par défaut des providers NestJS (pas de scope spécifié)

### Factory (`useFactory` NestJS) — pas exactement le DP classique
- `useFactory` = mécanisme NestJS pour **fabriquer une instance** au démarrage de l'application
- C'est une **fonction** qui reçoit des dépendances injectées (ex: `ConfigService`) et retourne l'objet construit
- Exemple StripeModule : `useFactory: (config: ConfigService) => new Stripe(config.get('STRIPE_SECRET_KEY'))` → crée une instance Stripe avec la clé lue depuis les variables d'environnement
- Le résultat est un **singleton** : une seule instance Stripe partagée dans toute l'app
- **`@Global()`** : rend le module visible dans tous les autres modules sans l'importer explicitement. Sans ça, il faudrait écrire `imports: [StripeModule]` dans OrdersModule, CheckoutModule, PaymentModule…
- En résumé : `useFactory` = "voici comment construire l'objet", `@Global()` = "tout le monde peut l'utiliser"
- Différence avec le DP Factory classique : ici c'est un mécanisme du conteneur IoC NestJS, pas un pattern orienté objet avec une interface abstraite
- Si le jury demande : "Le useFactory permet de construire l'instance Stripe au démarrage en injectant la configuration. Le @Global évite de l'importer dans chaque module — c'est un singleton accessible partout."

### Strategy (Passport) — ≠ Factory, ne pas confondre
- **Pattern Strategy** = plusieurs implémentations **interchangeables** d'un même contrat (comportement). Le choix se fait au runtime
- **≠ Factory** = crée des **objets** différents selon un paramètre. Factory = création, Strategy = comportement
- **Dans GreenRoots** : Passport implémente Strategy avec 2 stratégies d'authentification :
  - `LocalStrategy` → validate(email, password) → vérifie email/mdp en BDD + bcrypt → utilisé sur `/login` via `AuthGuard('local')`
  - `JwtStrategy` → validate(req, payload) → vérifie JWT cookie + blacklist + rôle BDD → utilisé sur les routes protégées via `JwtAuthGuard`
- Les deux héritent de `PassportStrategy` et implémentent `validate()` avec une logique différente
- Le guard sur la route détermine quelle stratégie est appelée — le controller ne sait pas laquelle tourne
- **Principe ouvert/fermé (SOLID)** : pour ajouter OAuth Google → créer `GoogleStrategy`, aucun code existant modifié
- **Résumé rapide** : Factory (Stripe) = comment créer l'objet, Strategy (Passport) = comment s'authentifier
- Si le jury demande : "Passport implémente le pattern Strategy — deux stratégies interchangeables partageant le même contrat validate(). Le guard détermine laquelle est appelée. Pour ajouter une 3e méthode d'auth, on crée une nouvelle stratégie sans toucher au code existant."

### State machine — définition
- Un système qui peut être dans **un seul état à la fois**, et qui passe d'un état à un autre via des **transitions déclenchées par des événements**
- 3 éléments : états (les situations possibles), transitions (les passages), événements (ce qui déclenche)
- Exemple concret : une commande est PENDING, le webhook Stripe arrive → transition vers CONFIRMED. Impossible de passer directement de PENDING à PLANTED.
- Si le jury demande : "Une state machine garantit que l'objet ne peut passer que par des états autorisés, dans un ordre défini."

### State machine — explicite vs implicite
- **Explicite (backend)** : `ORDER_STATUS_TRANSITIONS` dans `orders.constants.ts` — une constante qui liste les transitions autorisées, le code vérifie avant chaque changement
- **Implicite (frontend)** : `PaymentState` dans `Checkout.tsx` — pas de constante, c'est le flux du code (`setPaymentState()`) qui gère les transitions (IDLE → INITIALIZING → READY ou ERROR)
- L'explicite est plus robuste (impossible de faire une transition non déclarée), l'implicite est plus simple pour des cas linéaires
- Différent de l'enum en BDD qui ne fait que contraindre les **valeurs** possibles (pas les transitions)

### PaymentState — flux frontend checkout
- **IDLE** : formulaire éditable, pas de Stripe visible (`clientSecret: null`)
- **INITIALIZING** : clic "Valider l'achat" → validation Zod → appel `initCheckout` → loader
- **READY** : API répond `clientSecret` + `orderId` → formulaire verrouillé (`isFormLocked = clientSecret !== null`) → Stripe PaymentElement apparaît
- **ERROR** : appel échoue → retour formulaire éditable, l'utilisateur peut réessayer
- Fichiers : `Checkout.tsx` (lignes ~29-148), `checkout.type.ts`, `StripePaymentSection.tsx`, `checkout.api.ts`

### Decorator — 3 types différents dans GreenRoots
- **Decorator de classe/méthode** : `@Roles('ADMIN')` → pose des métadonnées sur la route, lues par le `RolesGuard` au runtime
- **Decorator de paramètre** : `@CurrentUser()` → extrait `request.user` (posé par JwtAuthGuard) et l'injecte directement dans le paramètre du controller → `updateOrder(@CurrentUser() user)`
- **Decorator de propriété (DTO)** : `@PriceToDecimal()` → appliqué sur une propriété du DTO, transforme la valeur lors de la sérialisation (prix centimes → euros)
- Si le jury demande : "Les decorators ajoutent du comportement déclaratif — ils posent des métadonnées lues par les guards, interceptors ou le sérialiseur au runtime."

### Serialize Interceptor — pourquoi custom et comment ça marche
- **Problème** : Prisma retourne des **plain objects** (objets JS bruts sans classe), pas des instances → le `ClassSerializerInterceptor` standard de NestJS ne fonctionne pas (il attend des instances pour appliquer `@Exclude()`/`@Expose()`)
- **Solution** : interceptor custom qui utilise `plainToInstance(DTO, data, { excludeExtraneousValues: true })` de class-transformer → convertit le plain object en instance du DTO
- **`excludeExtraneousValues: true`** = seuls les champs avec `@Expose()` sont gardés. Tout le reste est supprimé
- **`@Serialize(TreeResponseDto)`** = sucre syntaxique qui appelle `@UseInterceptors(new SerializeInterceptor(dto))`
- **Flux complet** :
  1. Controller retourne le plain object Prisma
  2. Interceptor intercepte la réponse (RxJS `.pipe(map(...))`)
  3. `plainToInstance()` crée une instance du DTO
  4. `@Expose()` filtre les champs, `@Transform()` transforme les valeurs, `@PriceToDecimal()` convertit centimes → euros
  5. Le client reçoit un JSON propre et filtré
- Gère aussi les réponses paginées `{ data: [], total, page, limit }` — transforme uniquement `data`
- Si le jury demande : "Prisma retourne des plain objects, pas des instances — le sérialiseur standard NestJS ne marche pas. Notre interceptor custom utilise plainToInstance pour convertir, puis excludeExtraneousValues ne garde que les @Expose. Ça garantit qu'on ne fuit jamais de données sensibles."
  5. Le client reçoit des euros (décimal)
- Justification : prix stockés en INT pour la précision (standard Stripe, pas de floating point), mais exposés en décimal pour le frontend
- Si le jury demande : "L'interceptor transforme automatiquement les réponses via le DTO. Le decorator PriceToDecimal convertit les centimes en euros — la BDD garde la précision, l'API expose un format lisible."

### Guards RBAC — ordre d'exécution et composition
- **Deux niveaux de `@UseGuards`** dans le controller orders :
  - **Niveau classe** : `@UseGuards(JwtAuthGuard)` → s'applique à **toutes** les routes du controller
  - **Niveau route** : `@UseGuards(RolesGuard)` + `@Roles('ADMIN', 'SUPERADMIN')` → uniquement sur les routes admin
- NestJS exécute d'abord les guards de classe, puis ceux de la route

### JwtAuthGuard — étapes détaillées (1er guard)
1. Étend `AuthGuard('jwt')` de Passport → c'est un **guard factory**, Passport fait le travail
2. Passport appelle `JwtStrategy` qui extrait le JWT depuis le **cookie HttpOnly** (`extractJwtFromCookie`)
3. Vérifie la **signature** du JWT avec `JWT_SECRET` + vérifie l'**expiration** (`ignoreExpiration: false`)
4. Si valide → appelle `validate(req, payload)` avec le payload décodé
5. `validate()` effectue **4 vérifications** :
   - Token **blacklisté** dans Redis ? (logout) → 401
   - Utilisateur **existe toujours** en BDD ? → 401
   - Email **vérifié** ? → 401
   - Récupère le **rôle actuel en BDD** (pas celui du JWT !)
6. `validate()` retourne `{ userId, email, role }` → **Passport pose automatiquement** cet objet dans `request.user` (code invisible de la librairie, pas du code applicatif)
7. Ensuite : `@CurrentUser()` lit `request.user`, le RolesGuard lit `request.user.role` — tout vient de ce que `validate()` a retourné
8. Si JWT absent/invalide/expiré → 401 Unauthorized, `validate()` n'est jamais appelé
- **Les étapes 2-3 sont du code Passport** (dans `node_modules/passport-jwt`), pas du code applicatif. On fournit la config via `super()`, Passport exécute. Notre seule responsabilité = `validate()`
- **Point clé** : le rôle dans le JWT n'est jamais utilisé pour l'autorisation. La strategy va chercher le rôle **frais en BDD** à chaque requête → un changement de rôle par un admin est effectif immédiatement

### RolesGuard — étapes détaillées (2e guard)
1. Implémente `CanActivate` → méthode `canActivate()` retourne `true` ou throw `ForbiddenException`
2. Utilise le **Reflector** pour lire `@Roles()` sur la route : `reflector.getAllAndOverride(ROLES_KEY, [handler, class])`
3. Si aucun `@Roles()` → retourne `true` (route sans restriction de rôle)
4. Récupère `request.user.role` (posé par JwtAuthGuard juste avant)
5. Si `SUPER_ADMIN` → bypass, retourne `true` quoi qu'il arrive
6. `requiredRoles.some(role => role === userRole)` → au moins un rôle correspond ? → `true` sinon 403

### Décorateur `@Roles()` — 3 lignes
- `SetMetadata('roles', ['ADMIN', 'SUPERADMIN'])` → pose juste des métadonnées sur la route
- Ne fait rien par lui-même — c'est le RolesGuard qui lit et agit

### `@CurrentUser()` — décorateur de paramètre
- Extrait `request.user` et l'injecte dans le controller → `updateOrder(@CurrentUser() user)`
- Évite `@Req() req` puis `req.user` → plus propre et typé

- Si le jury demande : "Le JwtAuthGuard vérifie le JWT et récupère le rôle **actuel** en BDD — pas celui du token. Le RolesGuard compare ce rôle avec les métadonnées @Roles() via le Reflector. L'ordre est garanti : identité d'abord, permissions ensuite."

## Modèle de données

### Identifier le type de relation — astuce "Combien ?"
- Poser la question dans les **deux sens** : "Un [A] peut avoir combien de [B] ?" et inversement
- Combiner les deux réponses :

| A → B | B → A | Relation |
|-------|-------|----------|
| un | un | One-to-One |
| plusieurs | un | Many-to-One |
| un | plusieurs | One-to-Many |
| plusieurs | plusieurs | Many-to-Many |

- **Exemples GreenRoots** :
  - User/Role : un User → un Role, un Role → plusieurs Users = **Many-to-One**
  - User/Cart : un User → un Cart, un Cart → un User = **One-to-One**
  - User/Order : un User → plusieurs Orders, un Order → un User = **One-to-Many**
  - Order/Tree : un Order → plusieurs Trees, un Tree → plusieurs Orders = **Many-to-Many** → table de jonction `OrderItem`
- **Où mettre la FK ?** Toujours du côté qui répond "un seul" :
  - One-to-One → FK du côté le plus dépendant (Cart porte userId)
  - Many-to-One / One-to-Many → FK du côté Many (User porte roleId)
  - Many-to-Many → table de jonction (OrderItem avec orderId + treeId)

### Enum vs Table séparée (Role)
- **Enum Prisma** (ex: OrderStatus) : liste fixe, pas de jointure, mais ajouter une valeur = migration + redéploiement
- **Table séparée** (ex: Role) : extensible sans migration (un INSERT suffit), peut porter des métadonnées (description, permissions)
- GreenRoots : table Role avec one-to-many. Un enum aurait suffi pour le MVP (3 rôles, pas de métadonnées), mais la table est prête pour une évolution (many-to-many si multi-rôles, ajout de permissions)
- Si le jury demande : "La table Role permet d'ajouter des rôles sans migration et pourrait porter des métadonnées. Un enum aurait suffi pour le MVP mais la table offre plus d'extensibilité."

### Prix en centimes (INT)
- Pas d'erreurs d'arrondi (0.1 + 0.2 ≠ 0.3 en float)
- Standard Stripe (attend des centimes)
- Decorator `@PriceToDecimal()` pour la conversion à l'affichage

### PostGIS
- Extension PostgreSQL pour coordonnées géographiques
- Permet requêtes spatiales (distance, rayon)
- Prévu pour l'évolution carte interactive du CDC

### B-tree — ne pas confondre
- **B-tree** = structure de données utilisée par PostgreSQL pour les **index** de base de données (accélérer les recherches). Ce n'est PAS un arbre de décision ni une table de transitions
- **Table de transitions** (state machine) = dictionnaire qui dit "depuis tel état, tu peux aller vers tels états" (ex : `ORDER_STATUS_TRANSITIONS`)
- **Arbre de décision** = suite de conditions if/then pour arriver à un résultat
- Ne pas mélanger ces trois concepts à l'oral

### Séparation Tree / TreeStock
- L'identité d'un arbre ≠ sa disponibilité commerciale
- Permet d'évoluer indépendamment (seuils d'alerte, historique)
- Séparation des responsabilités

### Snapshot des données de commande
- OrderAddress.countryCode = string, pas de FK vers Country
- unitPrice figé dans OrderItem au moment de l'achat
- Préserve l'intégrité historique si les données de référence changent

### Cart/CartItem en BDD
- Tables existantes mais non utilisées dans le MVP
- Panier géré côté client (Zustand + localStorage)
- Prévues pour synchronisation serveur en évolution future

### Droits d'accès applicatifs vs BDD
- Guards NestJS (JWT + rôles) plutôt que rôles PostgreSQL
- Prisma = connexion unique, pas de multi-utilisateur BDD
- Pattern standard des applications REST modernes

## Sécurité

### Prisma $queryRaw — tagged template vs unsafe
- Dans GreenRoots, il y a du SQL brut dans `findRandomTrees` (`ORDER BY RANDOM()` n'existe pas dans l'API Prisma)
- **Tagged template literal** (backticks) : `` $queryRaw`SELECT id FROM trees LIMIT ${limit}` `` → Prisma **paramètre automatiquement** les variables → **sécurisé** contre l'injection
- **$queryRawUnsafe** (string) : `$queryRawUnsafe("SELECT ... " + limit)` → concaténation directe → **vulnérable** à l'injection SQL
- La différence est subtile mais critique : même syntaxe JS (template literal), mais Prisma intercepte les variables dans la version tagged
- Si le jury demande : "On utilise $queryRaw avec des tagged templates — Prisma paramètre automatiquement les variables, c'est sécurisé. Ce serait dangereux uniquement avec $queryRawUnsafe qui fait de la concaténation."

### JWT vs Cookie — ce n'est pas la même chose
- **JWT** = le **contenu** (un jeton signé contenant `{ userId, role, exp }`). C'est juste une chaîne de caractères. Il pourrait être stocké n'importe où
- **Cookie** = le **conteneur de transport**. Mécanisme du protocole HTTP : le serveur dit au navigateur "stocke cette donnée et renvoie-la moi à chaque requête"
- Analogie : le JWT c'est une **lettre** (message signé), le cookie c'est une **enveloppe scellée** que seul le navigateur peut manipuler
- Avant : JWT en localStorage → accessible par JavaScript → volable par XSS
- Maintenant : JWT dans un cookie HttpOnly → invisible pour JavaScript → protégé contre le vol XSS
- **Le cookie ressemble à quoi ?** C'est un header HTTP :
  - Réponse serveur : `Set-Cookie: jwt=eyJhbG...; HttpOnly; Secure; SameSite=Lax; Max-Age=86400`
  - Requête navigateur (automatique) : `Cookie: jwt=eyJhbG...`
- Un cookie peut stocker n'importe quelle petite donnée texte (max ~4KB)
- Si le jury demande : "Le JWT c'est le jeton d'identité, le cookie c'est l'enveloppe de transport. HttpOnly empêche JavaScript de lire l'enveloppe — même en cas de XSS, le token ne peut pas être volé."

### Flags de cookie — instructions au navigateur
- Un "flag" = un **attribut/option** posé par le serveur au moment de créer le cookie
- `HttpOnly` → JavaScript ne peut pas lire ce cookie (ni `document.cookie`, ni aucune API JS)
- `Secure` → envoyé uniquement sur HTTPS (pas en clair sur HTTP)
- `SameSite=Lax` → pas envoyé depuis un autre site sauf liens GET (protection CSRF)
- `Max-Age=86400` → durée de vie du cookie (ici 24h)
- `Path=/` → envoyé pour toutes les routes du domaine
- Ce n'est pas du code, c'est une **instruction dans le header HTTP** de réponse. Le navigateur les lit et applique les restrictions
- Si le jury demande : "Les flags sont des instructions au navigateur qui contrôlent quand et comment le cookie est envoyé — HttpOnly pour la sécurité XSS, Secure pour HTTPS, SameSite pour le CSRF."

### JWT HttpOnly — pourquoi la migration
- Sprint initial : JWT en localStorage → simple à implémenter, mais vulnérable au XSS
- Sprint 3 (audit sécurité) : migration vers cookies HttpOnly+Secure+SameSite=Lax
- Le token reste valide jusqu'à expiration → blacklist Redis nécessaire pour le logout
- HttpOnly = protection côté client (vol impossible), blacklist = invalidation côté serveur — complémentaires

### Défense en profondeur (organisation sécurité par couches)
- Principe ANSSI/OWASP : chaque couche a ses propres mesures de sécurité, indépendantes
- Si une couche est compromise, les autres tiennent
- Dans GreenRoots : couche réseau (Helmet, CORS) → couche application (ValidationPipe, ThrottlerGuard, Guards) → couche données (Prisma paramétré, transactions)
- Ce n'est PAS lié au SPA — c'est un principe universel de sécurité
- Si le jury demande : "On applique la défense en profondeur — chaque couche a ses protections indépendantes, la compromission d'une couche ne donne pas accès aux autres."

### Hachage vs Chiffrement — ne JAMAIS confondre
- **Hachage** (hashing) = **irréversible**. Entrée → empreinte fixe. Impossible de retrouver l'original. Usage : mots de passe, tokens, intégrité
- **Chiffrement** (encryption) = **réversible**. On peut déchiffrer avec la clé. Usage : HTTPS, JWT (signature), données confidentielles en transit
- On ne "décrypte" jamais un hash — on hash la tentative et on **compare** les hash
- Dans GreenRoots : **bcrypt** (hash lent, mots de passe) + **SHA-256** (hash rapide, tokens) + **HTTPS** (chiffrement, transit)
- Si le jury demande : "Le hachage est irréversible — on ne stocke jamais un mot de passe, on stocke son hash. Pour vérifier, on hash la tentative et on compare. Le chiffrement est réversible, c'est pour protéger les données en transit."

### bcrypt vs SHA-256 — deux usages différents dans GreenRoots
- **bcrypt** (mots de passe) : volontairement lent (cost factor 10 = 2^10 tours ≈ 100ms). Les mots de passe humains ont une faible entropie → il faut ralentir le brute-force
- **SHA-256** (tokens reset/vérification email) : rapide. Les tokens sont générés par `crypto.randomBytes(32)` = 256 bits d'entropie → déjà impossible à brute-forcer, pas besoin de ralentir
- **Le cost factor `10`** ≠ le salt. Cost factor = nombre de tours (2^10). Salt = chaîne aléatoire unique par utilisateur, intégrée dans le hash
- Si le jury demande : "bcrypt pour les mots de passe car faible entropie — le cost factor ralentit le brute-force. SHA-256 pour les tokens à haute entropie — la vitesse n'est pas un risque."

### Bcrypt — hash, salt et rounds (cycle complet)
- **Pourquoi hasher ?** Si la BDD est compromise, un attaquant qui voit les mots de passe en clair peut les utiliser directement. Avec du hash, il ne voit que des chaînes inexploitables
- **Le salt** = une chaîne aléatoire unique, générée pour chaque utilisateur, ajoutée au mot de passe **avant** le hash
  - Sans salt : même mot de passe → même hash (vulnérable aux rainbow tables = dictionnaires de hash pré-calculés)
  - Avec salt : même mot de passe + salt différent → hash complètement différent → rainbow tables inutiles
- **Les "10 rounds" (cost factor)** = concept séparé du salt. Bcrypt applique l'algorithme **2^10 = 1024 fois en boucle**
  - 10 rounds ≈ 100ms par hash. 11 rounds ≈ 200ms, 12 ≈ 400ms → **exponentiel**
  - Assez lent pour décourager le brute force, assez rapide pour ne pas bloquer l'utilisateur au login
- **Cycle à la création du compte** :
  1. Utilisateur envoie "MonMotDePasse123"
  2. Bcrypt génère un salt aléatoire unique
  3. Bcrypt combine salt + mot de passe → applique l'algorithme 1024 fois → produit le hash
  4. Bcrypt stocke **tout dans une seule chaîne** : `$2b$10$salt22caractères...hash_final...`
     - `$2b$` = version bcrypt, `10$` = rounds, 22 chars = salt, le reste = hash
  5. On enregistre cette chaîne unique en BDD (une seule colonne)
- **Cycle à l'authentification** :
  1. Utilisateur envoie "MonMotDePasse123"
  2. Bcrypt lit le hash stocké en BDD → **extrait le salt** (les 22 caractères après `$10$`)
  3. Bcrypt applique **le même salt** + le mot de passe proposé → recalcule le hash
  4. Compare le hash recalculé avec le hash stocké → match ou pas
- **Le salt est-il aléatoire à chaque fois ?** À la création : oui, unique par utilisateur. À la vérification : non, bcrypt **réutilise le salt stocké** dans le hash. Le salt est figé pour ce hash, mais chaque utilisateur a le sien
- Élégance de bcrypt : tout dans une seule chaîne → pas de colonne séparée pour le salt
- Si le jury demande : "Bcrypt génère un salt aléatoire unique par utilisateur et l'intègre dans le hash. Les 10 rounds ralentissent volontairement le calcul pour contrer le brute force. À la vérification, il extrait le salt du hash stocké et recalcule — c'est transparent."

### Tokens hashés en BDD — reset password et vérification email
- **Avant (sprint initial)** : les tokens de reset password et vérification email étaient stockés **en clair** en BDD (colonnes `emailVerificationToken`, `passwordResetToken` en VARCHAR 255)
- **Le risque** : si la BDD est compromise, l'attaquant a les tokens en clair → il peut vérifier n'importe quel compte ou réinitialiser n'importe quel mot de passe
- **Après correction (sprint 3, audit OWASP)** : tokens hashés en **SHA-256** avant stockage → colonnes renommées `emailVerificationTokenHash` / `passwordResetTokenHash` (VARCHAR 64)
- **Le flow** : token brut généré (`crypto.randomBytes(32)`) → hashé → stocké en BDD. Le token brut est envoyé par email. À la vérification : on hashe le token reçu et on compare avec le hash stocké
- **Pourquoi SHA-256 et pas bcrypt ?** Les tokens ont déjà 256 bits d'entropie (générés par `crypto.randomBytes`). Bcrypt c'est pour les mots de passe humains (faible entropie, besoin de ralentir le brute force). SHA-256 suffit pour des tokens déjà cryptographiquement aléatoires
- Bon exemple de cycle **audit OWASP → identification du risque → correction concrète**
- Si le jury demande : "Les tokens étaient en clair, l'audit OWASP l'a identifié comme risque. On les hashe maintenant en SHA-256 avant stockage. SHA-256 et pas bcrypt parce que les tokens ont déjà 256 bits d'entropie — pas besoin de ralentir le calcul."

### CSRF (Cross-Site Request Forgery) — l'attaque et la protection
- **Le scénario d'attaque** :
  1. Tu es connecté sur GreenRoots (ton navigateur a le cookie JWT)
  2. Tu visites un site malveillant
  3. Ce site contient un formulaire caché qui fait un POST vers `greenroots.com/api/orders/123/cancel`
  4. Le navigateur envoie **automatiquement** le cookie JWT avec cette requête (comportement par défaut des cookies)
  5. Le backend reçoit un JWT valide → exécute l'annulation → ta commande est annulée sans ton consentement
- C'est le **prix des cookies** : le navigateur les envoie automatiquement, même depuis un autre site
- **SameSite=Lax protège comment ?**
  - Même domaine → cookie envoyé (toutes méthodes) ✅
  - Autre domaine + lien GET (clic sur un lien GreenRoots depuis Slack) → cookie envoyé (bonne UX) ✅
  - Autre domaine + POST/AJAX → cookie **non envoyé** (protection CSRF) 🛡️
- **Pourquoi Lax et pas Strict ?**
  - Strict = cookie jamais envoyé si tu viens d'un autre site, même en cliquant un lien → tu es déconnecté si tu cliques un lien GreenRoots depuis Slack → mauvaise UX
  - Lax = compromis : liens GET autorisés (UX), POST/AJAX bloqués (sécurité)

### SameSite vs CORS — complémentaires, pas identiques
- **CORS** = contrôle côté **serveur** : "quels domaines ont le droit d'appeler mon API ?" Protège l'**API**
- **SameSite** = contrôle côté **navigateur** : "est-ce que j'envoie le cookie avec cette requête ?" Protège le **cookie**
- Ils sont complémentaires (défense en profondeur) :
  - Requête AJAX depuis un site malveillant → CORS la bloque (origin non autorisé)
  - Formulaire POST caché (pas du AJAX, CORS ne s'applique pas) → SameSite=Lax bloque l'envoi du cookie
- Si le jury demande : "CORS protège l'API en filtrant les origines, SameSite protège le cookie en filtrant les contextes d'envoi. Les deux ensemble couvrent des vecteurs d'attaque différents."

### Authentification vs Autorisation vs Ownership
- **Authentification** = "**Qui es-tu ?**" → vérifier l'identité. JwtAuthGuard vérifie le JWT et extrait userId/email/role
- **Autorisation** = "**As-tu le droit ?**" → vérifier les permissions. RolesGuard compare le rôle avec `@Roles()` sur la route
- **Ownership** = "**Est-ce ta ressource ?**" → vérification métier dans le service. `order.userId !== userId` → ForbiddenException
- L'ownership est dans le **service** (pas dans un guard) parce que ça dépend de la donnée elle-même, pas d'un rôle générique
- Ordre d'exécution : authentification (guard 1) → autorisation (guard 2) → ownership (service)
- Si le jury demande : "Trois niveaux de contrôle : qui es-tu (JWT), quel est ton rôle (RBAC), est-ce ta donnée (ownership). Le dernier est dans le service parce qu'il dépend de la ressource demandée."

### Helmet
- Middleware Express qui ajoute automatiquement des **headers HTTP de sécurité** aux réponses
- Exemples : `X-Content-Type-Options: nosniff` (pas de devinette de type fichier), `X-Frame-Options` (pas d'iframe → anti-clickjacking)
- Une seule ligne : `app.use(helmet())` → couvre une dizaine de headers d'un coup
- Si le jury demande : "Helmet sécurise les headers HTTP automatiquement — c'est une protection côté réseau qu'on active en une ligne."

### CORS (Cross-Origin Resource Sharing)
- Par défaut, un navigateur **interdit** les requêtes vers un domaine différent de la page
- Le CORS autorise explicitement un domaine précis : `origin: FRONTEND_URL`
- Sans ça, n'importe quel site pourrait appeler ton API depuis le navigateur d'un utilisateur connecté
- Si le jury demande : "Le CORS restreint l'accès à notre API au seul domaine du frontend — aucun autre site ne peut appeler l'API depuis un navigateur."

### ValidationPipe (NestJS) — whitelist + forbidNonWhitelisted
- **Pipe** NestJS (pas un middleware) : couche de validation appliquée automatiquement à chaque requête, configurée globalement dans `main.ts`
- Les deux options font des choses **différentes et complémentaires** :
  - `whitelist: true` → **supprime silencieusement** les propriétés qui ne sont pas décorées dans le DTO. Ex : DTO a `@IsString() name` et `@IsEmail() email`, quelqu'un envoie `{ name, email, role: "ADMIN" }` → le champ `role` est supprimé avant d'arriver au contrôleur
  - `forbidNonWhitelisted: true` → au lieu de supprimer silencieusement, **rejette la requête avec une erreur 400**. Combiné avec whitelist : "non seulement je ne garde pas les champs inconnus, mais en plus je te dis que c'est interdit"
- `transform: true` → convertit le payload en **instance de classe DTO** (pas un objet JS brut), permet à class-transformer de fonctionner
- `enableImplicitConversion: true` → convertit les types selon la déclaration TypeScript (ex : query param `"2"` → `2` si `page: number`) — sans ça il faudrait `@Type(() => Number)` sur chaque propriété
- Première ligne de défense contre l'injection de données malveillantes — le code du contrôleur ne voit jamais les champs non autorisés
- Si le jury demande : "Le ValidationPipe est global, il filtre toutes les entrées. whitelist supprime les champs inconnus, forbidNonWhitelisted va plus loin en rejetant la requête. transform + enableImplicitConversion gèrent la conversion de types. C'est la première ligne de défense avant même que le code métier ne s'exécute."

### ThrottlerGuard (rate limiting)
- Limite le **nombre de requêtes par IP** par unité de temps (ex : 100 requêtes/min)
- Si dépassement → 429 Too Many Requests
- Protège contre les attaques par saturation (DDoS léger, brute force)
- ≠ guards JWT/Roles qui gèrent les **droits d'accès** — le ThrottlerGuard gère le **volume**
- Si le jury demande : "Le ThrottlerGuard protège contre l'abus de requêtes — c'est du rate limiting, pas de la gestion de droits."

### Audit OWASP — comment en parler
- L'OWASP Top 10 est un référentiel des 10 failles de sécurité les plus courantes (injection, XSS, broken access control…)
- Tony a initié la démarche d'audit en sprint 3, utilisé l'IA comme outil d'assistance pour structurer l'analyse
- L'important pour le jury = la **démarche** (avoir pensé à auditer, connaître le référentiel) + les **correctifs appliqués**
- Si le jury demande : "J'ai initié un audit basé sur le référentiel OWASP Top 10:2025 en sprint 3. J'ai utilisé l'IA pour structurer l'analyse, puis j'ai analysé les résultats et appliqué les correctifs."

### Stripe — données bancaires
- Les données carte ne transitent jamais par notre API
- Le composant PaymentElement communique directement avec Stripe via le clientSecret
- Backend crée un PaymentIntent → reçoit clientSecret → frontend le passe à Stripe

### Signature HMAC — comment le backend vérifie les webhooks Stripe
- **Le problème** : l'endpoint `/webhook` est une URL publique. N'importe qui pourrait envoyer un faux POST pour simuler un paiement réussi → déclencher une livraison sans payer
- **La solution : signature HMAC-SHA256**
- **Comment ça marche concrètement** :
  1. Configuration : dans le dashboard Stripe, on récupère un **webhook secret** (clé secrète partagée, stockée en variable d'env `STRIPE_WEBHOOK_SECRET`)
  2. À chaque webhook, Stripe prend le body brut + un **timestamp**, calcule un **hash HMAC-SHA256** avec le secret, et met le résultat dans le header `Stripe-Signature`
  3. Le backend reçoit la requête et fait **le même calcul** : body brut + timestamp + même secret → HMAC-SHA256
  4. Si les deux hash correspondent → la requête vient bien de Stripe (seul Stripe a le secret). Sinon → rejetée
- **Dans le code** : `stripe.webhooks.constructEvent(body, signature, secret)` fait tout ça automatiquement et lève une exception si la signature est invalide
- **HMAC vs hash simple** : un hash SHA256 classique, n'importe qui peut le calculer (il suffit de connaître le message). Un HMAC intègre une **clé secrète** dans le calcul → sans la clé, impossible de produire le bon hash
- **Le timestamp** empêche les "replay attacks" : même si un attaquant intercepte une vraie requête Stripe, il ne peut pas la rejouer plus tard car le timestamp sera trop vieux (>5 min par défaut)
- **Analogie** : un sceau de cire sur une lettre. Seul l'expéditeur (Stripe) a le tampon (le secret). Tu vérifies le sceau avant d'ouvrir la lettre
- Si le jury demande : "Le webhook est protégé par une signature HMAC-SHA256. Le backend recalcule le hash avec le secret partagé et compare — si ça ne correspond pas, la requête est rejetée. Le timestamp empêche le rejeu."

### Anti-spoofing webhook — pourquoi vérifier le paymentIntentId en plus de la signature
- La signature HMAC prouve que le webhook vient bien de Stripe. Mais elle ne prouve pas que le `paymentIntentId` dans le body correspond à **la bonne commande**
- Scénario : la signature est valide (c'est Stripe), mais le paymentIntentId correspond à un autre compte ou une autre commande → sans vérification, on confirmerait une commande avec un paiement qui ne lui appartient pas
- La vérification `order.paymentIntentId !== paymentIntent.id` est une couche supplémentaire : on vérifie que le paiement est bien **celui qu'on attend pour cette commande**
- ≠ idempotence (qui empêche le double traitement) : l'anti-spoofing empêche le **mauvais** traitement
- Si le jury demande : "La signature vérifie l'expéditeur (Stripe), le paymentIntentId vérifie le contenu (le bon paiement pour la bonne commande). Ce sont deux contrôles complémentaires."

### Idempotence du webhook — protection contre le double traitement
- Stripe peut renvoyer le même événement (retry après timeout réseau)
- Le code vérifie le `paymentStatus` avant de continuer : si déjà `PAID` → return, aucune modification
- **3 protections complémentaires** sur le webhook : signature HMAC (expéditeur), paymentIntentId (contenu), statut PAID (idempotence)
- **Idempotent** = même résultat quel que soit le nombre d'appels — essentiel en e-commerce
- Si le jury demande : "Le webhook est idempotent — si le statut est déjà PAID, un deuxième appel n'a aucun effet. Combiné avec la signature HMAC et la vérification du paymentIntentId, ça fait trois couches de protection."

### Upload signé Cloudinary — même principe que le webhook
- **Le problème** : sans protection, n'importe qui pourrait uploader n'importe quoi sur le compte Cloudinary (images inappropriées, fichiers énormes pour exploser la facture)
- **La solution** : upload signé en 3 étapes (même principe de signature que le webhook Stripe) :
  1. Le frontend demande une signature au backend : `POST /cloudinary/signature` avec le dossier cible. **Route protégée par JwtAuthGuard** → seul un utilisateur authentifié peut demander une signature
  2. Le backend génère la signature : dossier + timestamp → hash SHA-1 avec l'`API_SECRET` Cloudinary (secret **côté backend uniquement**, jamais exposé au frontend). Renvoie : signature + API key (publique) + timestamp
  3. Le frontend upload directement vers Cloudinary avec la signature. Cloudinary refait le calcul de son côté → si la signature est valide, l'upload est accepté
- **Sécurités cumulées** :
  - Authentification obligatoire (JwtAuthGuard)
  - Whitelist de dossiers autorisés : `trees`, `locations`, `avatars` (un DTO avec `@IsIn` valide le dossier)
  - Le secret API ne quitte jamais le serveur → le frontend ne connaît que l'identifiant (API key), pas le mot de passe (API secret)
- **API Key vs API Secret** : la clé API est publique (identifie le compte), le secret est privé (signe les requêtes). C'est comme un login/mot de passe : le login peut être connu, le mot de passe jamais
- Si le jury demande : "Les uploads Cloudinary sont signés. Le backend génère une signature avec le secret API, le frontend l'utilise pour uploader directement. Le secret ne quitte jamais le serveur."

### Cloudinary — pourquoi le backend ne reçoit jamais l'image
- Le frontend upload **directement** vers Cloudinary avec la signature — le fichier ne transite pas par notre API
- **Avantages** : pas de consommation de bande passante/mémoire côté serveur pour des fichiers potentiellement lourds, pas de risque de fichier malveillant stocké sur notre infra
- Le backend ne fait que signer : "oui, cet utilisateur authentifié a le droit d'uploader dans ce dossier"
- C'est aussi un argument **éco-conception** : on évite un transfert réseau supplémentaire (frontend → backend → Cloudinary serait deux transferts au lieu d'un)
- Si le jury demande : "Le backend ne touche jamais l'image. Il signe une autorisation, le frontend upload directement vers Cloudinary. Ça économise la bande passante serveur et réduit la surface d'attaque."

## UML — Diagramme de cas d'utilisation

### Notation de base
- **Acteur** (bonhomme bâton) : quelqu'un qui interagit avec le système (Visiteur, Utilisateur, Admin)
- **Cas d'utilisation** (ellipse) : une action que l'acteur peut faire ("Consulter catalogue", "Passer commande")
- **Héritage entre acteurs** : l'Utilisateur hérite des droits du Visiteur, l'Admin hérite de tout. Dans le code GreenRoots, c'est implémenté via les guards NestJS (pas d'héritage POO)

### `<<include>>` vs `<<extend>>`
- **`<<include>>`** = toujours exécuté. Ex : "Passer commande" inclut toujours "Vérifier stock" — on ne peut pas commander sans vérifier le stock
- **`<<extend>>`** = optionnel, sous condition. Ex : "Gérer commande" peut étendre vers "Rembourser via Stripe" seulement si annulation
- Moyen mnémotechnique : include = obligatoire, extend = optionnel

## UML — Diagrammes de séquence et d'états

### Diagramme de séquence — c'est quoi
- Montre l'ordre des échanges (messages) entre plusieurs acteurs/systèmes dans le temps
- Axe vertical = temps (de haut en bas), axe horizontal = les participants
- Flèche pleine (`->>`) = appel/requête, flèche pointillée (`-->>`) = réponse
- Dans GreenRoots : utilisé pour modéliser le checkout (5 participants) et l'auth (5 participants)

### Diagramme d'états (stateDiagram) — c'est quoi
- Montre les différents états possibles d'un objet et les transitions entre eux
- Chaque transition a un déclencheur (ex : "webhook succeeded" fait passer PENDING → CONFIRMED)
- Dans GreenRoots : OrderStatus et PaymentStatus évoluent en parallèle, pilotés par le webhook Stripe
- Différent de la state machine dans le code (`ORDER_STATUS_TRANSITIONS`) qui est l'implémentation de ce diagramme
- Si le jury demande : "Le diagramme d'états documente les transitions autorisées, la constante dans le code les applique — c'est la même logique, modélisation vs implémentation."

### Communication asynchrone (webhook + polling)
- Le paiement Stripe est **asynchrone** : le frontend envoie le paiement, mais la confirmation arrive plus tard via webhook
- Le frontend ne sait pas quand le webhook a été traité → il fait du **polling** (interroge le backend en boucle)
- Polling adaptatif dans GreenRoots : interroge toutes les X secondes jusqu'à obtenir CONFIRMED ou un timeout
- Si le jury demande : "Le webhook est serveur-à-serveur, le frontend n'en a pas connaissance, donc il interroge le backend en boucle pour savoir si la commande est confirmée."

### SSE vs Polling — évolution prévue
- **Polling** (actuel) : le frontend envoie une requête toutes les 2s → beaucoup de requêtes inutiles si le webhook n'est pas encore arrivé
- **SSE (Server-Sent Events)** : le client ouvre **une seule connexion**, le serveur envoie un événement quand le webhook est traité → notification instantanée, zéro requête inutile
- NestJS supporte SSE nativement avec le decorator `@Sse()` sur un endpoint
- **WebSockets** = bidirectionnel (client ↔ serveur), **SSE** = unidirectionnel (serveur → client) — SSE suffit pour ce cas
- Évolution prévue dans GreenRoots : remplacer le polling par du SSE (éco-conception : moins de requêtes)
- Si le jury demande : "Le polling fonctionne pour le MVP. En évolution, le SSE permettrait une notification instantanée avec une seule connexion — plus efficace et plus éco-responsable."

### Cron de réconciliation
- Un cron horaire (`OrdersCleanupService`) gère deux cas limites :
  - Commandes abandonnées (PENDING depuis trop longtemps → stock libéré)
  - Webhooks perdus (PaymentStatus PROCESSING mais Stripe dit PAID → réconciliation via API Stripe)
- Filet de sécurité : le webhook gère 99% des cas, le cron rattrape le reste
- Si le jury demande : "C'est un mécanisme de réconciliation — on interroge directement l'API Stripe pour les commandes bloquées, au cas où un webhook serait perdu."

## Checkout & Paiement

### Checkout en deux étapes (init → confirm)
- `init` crée la commande + le PaymentIntent mais **ne touche pas au stock**
- `confirm` vérifie et réserve le stock **juste avant** le paiement Stripe (transaction atomique)
- Pourquoi : si on réserve à l'init, on bloque du stock pendant 30 min pour quelqu'un qui peut abandonner. Un autre client ne peut pas acheter ce stock bloqué
- Ça évite aussi de devoir gérer un panier côté serveur synchronisé avec le stock réservé — le panier reste en localStorage/Zustand
- Si le jury demande : "C'est au moment le plus critique — le paiement — qu'on vérifie le stock. Ça minimise la fenêtre de blocage et simplifie l'architecture."

### Timeout 30 minutes et réutilisation des commandes PENDING
- `initCheckout` cherche une commande PENDING existante (< 30 min) avant d'en créer une nouvelle → **mise à jour** au lieu de création
- Pourquoi 30 min : même sans blocage de stock, ce seuil reste utile :
  - Le PaymentIntent Stripe a un montant associé → au-delà de 30 min, mieux vaut repartir propre
  - Limite le nombre de **commandes zombies** en base (sans seuil, 10 allers-retours = 10 commandes PENDING)
  - Cohérence avec le `OrdersCleanupService` (cron) qui utilise le même seuil pour nettoyer
  - 30 min = standard Stripe pour l'expiration d'un checkout session — bon benchmark
- Si le jury demande : "On réutilise la même commande tant que l'utilisateur est actif dans les 30 minutes. Ça évite de polluer la base avec des commandes fantômes et c'est cohérent avec le timeout Stripe."

### Webhook Stripe comme source de vérité
- Le retour de `stripe.confirmPayment()` côté frontend n'est PAS fiable : l'utilisateur peut fermer son navigateur, perdre sa connexion
- Principe fondamental : **on ne fait jamais confiance au client**, qui peut être manipulé. La seule source de vérité protégée, c'est la communication serveur-à-serveur entre Stripe et le backend
- Le webhook est signé HMAC → vérifie que c'est bien Stripe qui envoie l'événement
- Le backend ne met à jour la commande (CONFIRMED + PAID) que sur réception du webhook
- Si le jury demande : "On ne fait jamais confiance au client. Le webhook est du serveur-à-serveur signé — c'est le seul canal fiable, indépendant du navigateur."

### Cron de réconciliation — pourquoi c'est nécessaire
- Le `OrdersCleanupService` (cron horaire) répond à **3 problèmes concrets** :
  1. **Données propres** : des commandes dans des statuts non valides doivent être corrigées pour un monitoring sain
  2. **Stock bloqué** : sans nettoyage, le stock réservé par des commandes abandonnées reste bloqué indéfiniment → un autre client ne peut pas acheter
  3. **Résilience inter-infrastructures** : le paiement implique **deux serveurs différents** (Stripe et notre backend) avec deux infrastructures distinctes — il existe une probabilité que l'un des deux tombe ou ait un problème réseau → le webhook ne passe jamais
- Le cron est un **filet de sécurité** : le webhook gère 99% des cas, le cron rattrape le 1% restant
- Si le jury demande : "Le cron gère la résilience entre deux infrastructures indépendantes — si le webhook se perd entre Stripe et notre serveur, le cron réconcilie en interrogeant directement l'API Stripe."

### State machine explicite (`ORDER_STATUS_TRANSITIONS`) — pourquoi pas un if/else
- La constante `ORDER_STATUS_TRANSITIONS` est un **dictionnaire** : pour chaque état, la liste des états autorisés
- Avantages par rapport à un if/else ou switch :
  1. **Lisibilité** : on voit d'un coup d'œil toutes les transitions autorisées
  2. **Extensibilité** : ajouter un nouveau statut = ajouter une ligne, pas réécrire des blocs
  3. **Maintenabilité** : la logique de transition est centralisée, pas dispersée dans le code
- Si 20 statuts → la constante reste un tableau lisible, le if/else devient un spaghetti
- Si le jury demande : "On a centralisé les transitions dans une constante plutôt que des if/else — c'est plus lisible, plus facile à étendre, et ça garantit qu'aucune transition non déclarée ne passe."

### Annulation commande — refund Stripe avant transaction BDD (fail early)
- Quand l'admin annule une commande payée : refund Stripe → puis transaction BDD (restauration stock + CANCELLED + REFUNDED)
- Le refund Stripe est fait **avant** la transaction BDD volontairement : si Stripe refuse le remboursement → exception throw → la transaction BDD n'a jamais lieu
- Pattern **fail early** : si on ne peut pas rembourser, on n'annule pas la commande — elle reste dans son état actuel
- L'inverse serait dangereux : annuler en BDD puis échouer au refund → commande annulée mais client jamais remboursé
- Feature anticipée (pas de cas concret en dev) — l'objectif était d'être production-ready : un utilisateur doit pouvoir annuler sa commande avant que l'arbre soit planté
- Si le jury demande : "Le refund Stripe est en premier parce que c'est le point de défaillance le plus probable. Si le remboursement échoue, on préfère ne rien toucher plutôt que d'avoir une commande annulée sans remboursement."

### Reset `stockReservedAt = null` sur payment_failed (lien avec réutilisation)
- Quand le webhook `payment_failed` arrive, le code remet `stockReservedAt = null` + restaure le stock
- Pourquoi : en remettant le flag à null, la commande redevient éligible à `findReusableOrder` dans `initCheckout`
- Résultat : l'utilisateur peut **réessayer de payer** sur la même commande sans qu'on en crée une nouvelle
- C'est le lien entre le webhook failed et la réutilisation intelligente des commandes PENDING
- Si le jury demande : "Le reset du flag permet à la commande d'être réutilisée — l'utilisateur peut réessayer sans créer de doublon."

### Fil conducteur §7.2 — écosystème d'intégrité des données
- Tous les composants métier du tunnel de vente forment un **écosystème** dont l'objectif commun est de **garantir l'intégrité des données**
- Chaque composant est un maillon : init (création), confirm (réservation stock), webhook (source de vérité paiement), cleanup (réconciliation), updateStatus (state machine)
- L'enjeu : que les commandes, le stock et les paiements restent **cohérents** à chaque étape, même en cas de défaillance (double-clic, webhook perdu, paiement échoué)
- Si le jury demande : "Chaque composant métier garantit l'intégrité des données à son niveau — ensemble ils forment un écosystème résilient où une défaillance à une étape est rattrapée par une autre."

### Idempotence confirmCheckout (`stockReservedAt`)
- Le flag `stockReservedAt` sert de **garde d'idempotence** dans `confirmCheckout`
- Scénario : l'utilisateur double-clique sur "Payer" → le front appelle `confirmCheckout` deux fois
- Sans le guard : le stock serait décrémenté **deux fois** (ex : -3 chênes × 2 = -6 au lieu de -3)
- Avec le guard : le 2e appel voit `stockReservedAt` non null → retourne `{ success: true }` sans rien toucher
- Même principe qu'une **clé d'idempotence** Stripe : vérifier si l'opération a déjà été faite avant de la refaire
- Idempotent = "même résultat quel que soit le nombre d'appels" — essentiel en e-commerce (opérations financières)
- Si le jury demande : "Le flag stockReservedAt garantit qu'on ne décrémente le stock qu'une seule fois, même si le frontend appelle deux fois — c'est une garde d'idempotence."

### Protection brute force (Redis)
- LoginAttemptsService : compteur Redis (INCR) avec TTL de 15 min
- 5 tentatives max avant blocage, le compteur s'efface tout seul grâce au TTL Redis
- Pattern classique : Redis est idéal pour ça (opérations atomiques, TTL natif, rapide)
- Même pour un email inexistant, on enregistre la tentative (protection contre le timing attack)
- Si le jury demande : "Redis gère le compteur avec un TTL automatique — pas besoin de cron pour nettoyer, c'est natif."

## Conteneurisation & DevOps

### Multi-stage Docker — Frontend (Dockerfile.prod)
- **Stage 1 (node:22-alpine)** : `npm ci` (toutes deps) → variables `VITE_*` injectées comme `ARG` (brûlées dans le bundle au build) → `npx vite build` → produit `dist/` (HTML + CSS + JS minifiés)
- **Stage 2 (nginx:alpine)** : `COPY --from=builder /app/dist` → fichiers statiques uniquement. Nginx configuré avec : proxy `/api` → `http://backend:3000`, cache assets 1 an (immutable), SPA fallback `try_files $uri /index.html`
- **Résultat** : image finale = Nginx + fichiers statiques. Pas de Node.js, pas de node_modules

### Multi-stage Docker — Backend (Dockerfile.prod)
- **Stage 1 (node:22-alpine)** : `npm ci` → `npx prisma generate` → `npm run build` → compile TS → JS dans `dist/`
- **Stage 2 (node:22-alpine)** : `npm ci --only=production` (deps prod uniquement) → `prisma generate` (regénère le client) → `COPY --from=builder dist/` + `src/generated/`
- **Sécurité** : `USER nestjs` (utilisateur non-root, principe du moindre privilège)
- **CMD** : `prisma migrate deploy` (applique les migrations) puis `node dist/src/main.js`
- **Résultat** : image sans devDeps (TypeScript, Jest, ESLint), code compilé, user non-root

### Database — pas de build
- Image prête depuis Docker Hub : `imresamu/postgis:16-3.4` (PostgreSQL 16 + PostGIS)
- Pas de Dockerfile, Docker télécharge et lance directement
- `volumes: postgres-data` → persiste les données sur le disque hôte (survit au redémarrage du conteneur)

- Si le jury demande : "Le multi-stage sépare le build du run. Le frontend produit des fichiers statiques servis par Nginx. Le backend compile le TypeScript et tourne en user non-root. La base utilise une image prête avec un volume persistant."

### Utilisateur non-root dans les conteneurs (principe du moindre privilège)
- Par défaut, un processus dans un conteneur Docker tourne en **root du conteneur** (distinct du root de la machine hôte, mais c'est une bonne pratique de ne pas prendre le risque)
- Dans le Dockerfile prod du backend : `USER nestjs` → le processus NestJS tourne en **utilisateur limité**
- **Pourquoi c'est important** : si un attaquant exploite une faille dans l'application et exécute du code dans le conteneur, il n'a que les droits de l'utilisateur `nestjs` — pas les droits root. Il ne peut pas installer de logiciels, modifier la config système, ou accéder aux fichiers d'autres services
- C'est le **principe du moindre privilège** : ne donner que les droits strictement nécessaires au fonctionnement
- En dev : on reste souvent en root pour le confort (volumes montés, permissions fichiers). En prod : toujours non-root
- Si le jury demande : "Le Dockerfile prod utilise `USER nestjs` pour que le processus tourne en utilisateur non-root — c'est le principe du moindre privilège appliqué aux conteneurs."

### Différence Docker Compose dev vs prod (7 vs 3 services)
- **Dev (7 services)** : frontend (Vite HMR), backend (NestJS watch), database, redis, nginx, stripe-cli (webhooks locaux), prisma-studio (explorer la BDD)
- **Prod (3 services)** : frontend (build statique + Nginx), backend (compilé), database
- En dev : volumes montés (hot reload), ports exposés, outils de confort (Prisma Studio, Stripe CLI)
- En prod : `expose` au lieu de `ports` (pas d'accès direct), images multi-stage, pas d'outils de dev
- Si le jury demande : "En dev on a besoin d'outils de confort pour développer efficacement. En prod on épure au maximum — moins de surface d'attaque, images plus légères."

### Docker Compose — c'est quoi, à quoi ça sert
- Docker Compose **orchestre plusieurs containers** à partir d'un seul fichier (`compose.yml`)
- Chaque service = un container isolé (frontend, backend, BDD, Redis…) avec sa propre image
- Un seul `docker compose up` démarre tout l'environnement — plus besoin d'installer PostgreSQL, Redis, etc. en local
- Chaque développeur a le **même environnement** peu importe son OS
- Si le jury demande : "Docker Compose orchestre tous nos services dans des containers isolés. Un seul `make dev` et tout l'environnement est prêt."

### Architecture prod — comment les conteneurs communiquent
- **3 conteneurs isolés** sur un **réseau Docker interne** (`network: coolify`)
- **Aucun service n'expose de port vers internet** — que des `expose` (visibles uniquement entre conteneurs)
- **Coolify** (reverse proxy sur le serveur Hetzner) = **seul point d'entrée HTTPS** depuis internet
- Flux : `Internet → Coolify (HTTPS) → réseau Docker → frontend:80 (Nginx) → fichiers statiques OU /api → backend:3000 → database:5432`
- Les conteneurs se trouvent par **nom de service** comme hostname : Nginx appelle `http://backend:3000`, le backend appelle `database:5432` via DATABASE_URL — Docker résout ces noms en IPs internes

### Variables d'environnement — deux moments différents
- **Au build (frontend)** : variables `VITE_*` passées en `ARG` Docker → Vite les incruste dans le JS au build → **figées** dans le bundle
- **Au runtime (backend)** : variables injectées par Docker au démarrage (`environment:`) → lues par `ConfigService` NestJS (validées par Joi — fail-fast). Configurées dans **Coolify** (interface web)
- **Au runtime (database)** : `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` → lues par PostgreSQL au **premier démarrage** pour créer l'utilisateur et la base

### Lien backend ↔ database via Prisma
- `DATABASE_URL=postgresql://root:password@database:5432/greenroot` — `database` = hostname Docker du service
- Au démarrage du backend : `prisma migrate deploy` (applique les migrations SQL) → puis `node dist/src/main.js` (NestJS démarre, PrismaService se connecte)
- Le client Prisma (généré au build) traduit les appels TypeScript en requêtes SQL envoyées à PostgreSQL via cette connexion
- Si le jury demande : "En prod, 3 conteneurs communiquent via un réseau Docker privé. Aucun n'est exposé sur internet — Coolify est le seul point d'entrée HTTPS. Le frontend proxy les /api vers le backend. Le backend se connecte à PostgreSQL via le DATABASE_URL qui utilise le hostname Docker."

### Volumes montés (hot reload en dev)
- Un **volume** lie un dossier local (ton code) à un dossier dans le container
- Quand tu modifies un fichier sur ta machine, le container le voit immédiatement → hot reload (Vite HMR / NestJS watch)
- En prod : pas de volumes montés, le code est copié dans l'image au build → immuable et sécurisé
- Si le jury demande : "Les volumes montés permettent le hot reload en dev — le container reflète en temps réel les changements de code sans reconstruire l'image."

### Healthchecks et depends_on
- **Healthcheck** = une commande qui vérifie que le service est prêt (ex : `pg_isready` pour PostgreSQL, `redis-cli ping` pour Redis)
- **depends_on + condition: service_healthy** = le service attend que sa dépendance soit *vraiment* prête avant de démarrer
- Sans ça : le backend pourrait démarrer avant que PostgreSQL soit prêt → crash
- Dans GreenRoots : le backend dépend de database (healthy) + redis (healthy) → démarrage ordonné garanti
- Si le jury demande : "Les healthchecks vérifient que chaque service est opérationnel, et depends_on garantit l'ordre de démarrage."

### `expose` vs `ports` (sécurité prod)
- **`ports: "3000:3000"`** = mappe le port du conteneur vers la machine hôte. **Accessible depuis l'extérieur** (navigateur, autres machines, internet)
- **`expose: "3000"`** = rend le port visible **uniquement aux autres conteneurs** du même réseau Docker. **Invisible depuis l'extérieur**
- **Preuve concrète dans GreenRoots** :
  - `compose.dev.yml` : `ports: "3000:3000"` (backend), `ports: "5432:5432"` (BDD), `ports: "6379:6379"` (Redis) → tout ouvert pour débugger facilement
  - `compose.prod.yml` : **que des `expose`**, zéro `ports` → rien n'est accessible directement depuis internet. Seul Coolify (reverse proxy) atteint les services via le réseau Docker interne (`network: coolify`)
- La BDD PostgreSQL n'est **jamais exposée** sur internet en production → un attaquant ne peut pas s'y connecter directement
- Si le jury demande : "En dev, les ports sont mappés pour le confort de développement. En prod, tout passe par `expose` — seul le reverse proxy Coolify est le point d'entrée, les services internes sont isolés dans le réseau Docker."

### Validation Joi — fail-fast au démarrage
- **Joi** = bibliothèque de validation de schéma (comme Zod côté frontend). Utilisé dans `ConfigModule.forRoot()` de NestJS pour valider **toutes les variables d'environnement au démarrage**
- **Le problème sans Joi** : le serveur démarre, tout semble OK… puis 3h plus tard, un utilisateur essaie de se connecter, le code cherche `JWT_SECRET`, ne le trouve pas → **crash en production**. Pire : si le secret existe mais fait 3 caractères → JWT signé avec un secret faible = faille de sécurité silencieuse
- **Avec Joi** : le serveur **refuse de démarrer** si :
  - `DATABASE_URL` absent → pas de connexion BDD possible
  - `JWT_SECRET` < 32 caractères → `.min(32)` rejette un secret trop faible
  - `NODE_ENV` pas dans `['development', 'production', 'staging']` → valeur incohérente
  - `FRONTEND_URL` pas une URI valide → `.uri()` vérifie le format → CORS cassé sinon
  - `STRIPE_SECRET_KEY`, `CLOUDINARY_API_SECRET`, etc. absents → intégrations cassées
- **REDIS_URL est `.optional()`** → cohérent avec la dégradation gracieuse : Redis est optionnel, l'app fonctionne sans
- **Pattern fail-fast** : mieux vaut crasher immédiatement avec un message d'erreur clair que tourner dans un état cassé silencieusement. Principe général de sécurité : ne jamais laisser un système fonctionner dans un état dégradé silencieux
- **Validation aussi de type** : `PORT: Joi.number().default(3000)` → si quelqu'un met `PORT=abc`, rejeté. `.default()` fournit une valeur par défaut si absente
- Si le jury demande : "Joi valide toutes les variables d'environnement au démarrage. Si une variable critique manque ou est invalide, l'application refuse de démarrer — c'est le pattern fail-fast. Ça évite de planter silencieusement en production sur une config manquante."

### CI/CD — Les 3 workflows GitHub Actions (+ Husky en amont)

**4 niveaux de qualité** : Husky (pre-commit) → pr-checks (feature branch) → develop-ci (intégration) → production-deploy (production)

### pr-checks — feedback rapide (~5 min)
- **Déclencheur** : push/PR sur **toutes les branches sauf main et develop** (feature, fix, chore)
- **2 jobs en parallèle** :
  - Backend : `npm audit --audit-level=critical` + lint + tests unitaires **sans BDD** (mocks)
  - Frontend : `npm audit` + lint + **build complet** (vérifie que TypeScript compile)
- **Pas de BDD, pas de coverage** — c'est un check rapide
- Si le jury demande : "pr-checks donne un retour rapide en ~5 min : audit sécurité, lint et tests unitaires sans base de données."

### develop-ci — suite complète avec vraie BDD
- **Déclencheur** : push/PR sur `develop`
- **Différence clé** : lance un **service PostgreSQL** dans GitHub Actions (vrai conteneur PostGIS)
- **3 jobs** :
  - Backend : `prisma migrate deploy` (vérifie les migrations) → seed → lint → tests avec vraie BDD → **coverage** (lignes, branches, fonctions)
  - Frontend : lint + build
  - **Test idempotence seed** : lance le seed 2 fois de suite → si le 2e échoue, le seed n'est pas idempotent
- Si le jury demande : "develop-ci va plus loin avec une vraie PostgreSQL — on vérifie les migrations, le seed idempotent, et on mesure la couverture."

### production-deploy — déploiement en production
- **Déclencheur** : push sur `main` (uniquement si fichiers app/, compose, nginx ou workflows modifiés) + déclenchement manuel possible
- **3 jobs séquentiels** :
  1. **test-and-lint** : mêmes tests qu'en develop-ci (avec BDD) — on ne déploie jamais sans retester
  2. **deploy** : trigger webhook Coolify → Coolify pull le code, rebuild les images, redémarre les conteneurs → attente 30s → **health check API** (10 tentatives, 10s entre chaque) → health check frontend
  3. **notify-failure** : notification Slack si échec (optionnel, si webhook configuré)
- Si le jury demande : "Le déploiement refait tous les tests puis trigger Coolify via webhook. Des health checks vérifient que l'API et le frontend répondent après déploiement."

### Seed idempotent — pourquoi c'est testé en CI
- Le seed garantit un **état de départ connu et reproductible** — chaque run de CI repart des mêmes données
- Idempotent = on peut le lancer N fois, même résultat (pas de doublons, pas d'erreurs)
- Testé explicitement en develop-ci : 2 runs consécutifs, le 2e ne doit pas échouer

### Husky + lint-staged (pre-commit hook)
- **Husky** intercepte les commandes Git (hook `pre-commit`) — avant chaque `git commit`, il exécute un script
- **lint-staged** exécute ESLint + Prettier **uniquement sur les fichiers modifiés** (pas tout le projet → rapide)
- Si le lint échoue → le commit est **bloqué**
- Résultat : du code mal formaté ne rentre jamais dans le repo — qualité garantie dès le développeur
- Si le jury demande : "Husky bloque le commit si le code n'est pas propre. Ça garantit la qualité en amont, avant même la CI."

### Swagger — pourquoi désactivé en prod
- Swagger génère une **documentation interactive complète** de l'API (toutes les routes, paramètres, types de réponse)
- En dev : très utile pour tester et documenter les endpoints
- En prod : c'est un **risque de sécurité** — ça donne à un attaquant la cartographie détaillée de toutes les routes (rétro-ingénierie facilitée)
- Désactivé via un simple `if (NODE_ENV !== 'production')` dans `main.ts`
- Si le jury demande : "Swagger est un outil de développement, pas de production. En prod il exposerait la structure complète de notre API."

## Facturation (InvoiceService)

### `@react-pdf/renderer` — génération PDF côté serveur
- Bibliothèque qui utilise la **syntaxe React** (composants, props) pour construire des documents PDF
- Au lieu de rendre du HTML → rend des éléments PDF (`Document`, `Page`, `View`, `Text`)
- `renderToBuffer()` transforme le composant React en Buffer binaire PDF → envoyé au client via un endpoint
- Pourquoi pas Puppeteer ? Puppeteer = lancer Chrome headless pour "imprimer" du HTML → lourd (Chrome en mémoire), lent, fragile. `@react-pdf/renderer` = génération native PDF, pas de navigateur, léger
- Cohérence stack : l'équipe connaît déjà React → la syntaxe est familière

### Notation `h = React.createElement` (pas de JSX en backend)
- Dans NestJS (backend), il n'y a **pas de compilateur JSX** (pas de Babel/Vite côté serveur)
- Donc au lieu d'écrire `<Text>Bonjour</Text>`, on écrit `h(Text, null, 'Bonjour')` — c'est **strictement la même chose**, juste sans le sucre syntaxique JSX
- `h` est une convention (alias de `createElement`, utilisé aussi dans Preact/Vue)
- Si le jury demande : "Le JSX n'est pas disponible côté backend, donc on utilise React.createElement directement — c'est ce que le compilateur JSX produit normalement."

### Numéro de facture (`generateInvoiceNumber`)
- Format `FA-2026-000001` — séquentiel par année, avec reset au 1er janvier
- Appelé **dans une transaction Prisma** → garantit pas de doublon même si deux webhooks arrivent simultanément
- Cherche le dernier numéro avec le préfixe de l'année courante, incrémente de 1
- **Obligation légale** : chaque facture doit avoir un numéro unique et séquentiel (pas de trous)

### Mentions légales (CGI art. 289)
- Obligation française : toute facture doit mentionner SIRET, TVA intracommunautaire, capital social du vendeur
- Dans le code : constante `SELLER` avec des valeurs fictives (contexte projet de formation)
- Le service inclut aussi : adresse client, détail des items (désignation, qté, PU HT, TVA, total HT), totaux (HT, TVA, TTC), date de vente, mode de règlement
- Si le jury demande : "Les mentions légales sont obligatoires sur toute facture en France. On les a intégrées comme constantes dans le service — en production, elles seraient alimentées par la configuration de l'entreprise."

## Accès aux données (Prisma / SQL / Redis)

### Transactions Prisma — atomicité et verrouillage
- Transaction = ensemble d'opérations liées exécutées en tout-ou-rien. Si une échoue → rollback automatique, les précédentes sont annulées
- Avantage supplémentaire : une seule connexion BDD pour toutes les requêtes de la transaction (pas un client par requête)
- **Verrouillage** : PostgreSQL pose automatiquement un **row-level lock** sur les lignes modifiées par un `UPDATE` (niveau d'isolation par défaut : Read Committed). Pas besoin de `SELECT FOR UPDATE` explicite
- Dans `confirmCheckout` : le `decrement` sur `treeStock` verrouille la ligne → une deuxième transaction concurrente attend que la première committe
- **Pourquoi pas tout en transaction** : une transaction maintient une connexion ouverte + potentiellement des verrous. Plus elle est longue, plus elle bloque. C'est pour ça que dans le cleanup, l'appel Stripe est **hors** transaction (un appel réseau lent ne doit pas maintenir un verrou BDD)
- Si le jury demande : "Les transactions garantissent l'atomicité et PostgreSQL gère le verrouillage automatiquement sur les UPDATE. On ne met en transaction que les opérations qui doivent être cohérentes entre elles."

### Include Prisma — pas des JOIN SQL
- Prisma traduit les `include` en **requêtes SQL séparées** (pas un vrai JOIN dans la plupart des cas)
- Exemple : `include: { items: { include: { tree: true } } }` → 1 requête orders + 1 requête order_items (WHERE orderId IN (...)) + 1 requête trees (WHERE id IN (...))
- C'est du **batching automatique** (pattern "data loader"), pas un JOIN unique
- Risque avec includes profonds (3-4 niveaux) : plus de requêtes sous le capot + beaucoup de données en mémoire
- Dans GreenRoots : acceptable (une commande a rarement > 10 items)
- **Alternative si performance insuffisante** : `$queryRaw` pour écrire du SQL pur avec de vrais JOIN optimisés — Prisma le supporte nativement
- **`TREE_INCLUDES`** : constante réutilisable qui centralise les includes (stock, category, location → country) → un seul endroit à modifier si on ajoute une relation
- Si le jury demande : "Prisma fait du batching, pas des JOIN. Pour notre volume c'est suffisant, mais on pourrait passer en raw SQL si les performances l'exigeaient."

### Promise.all — paralléliser des requêtes indépendantes
- `Promise.all([promesse1, promesse2])` exécute les promesses **en parallèle** et retourne les résultats dans le même ordre
- Temps = max(p1, p2) au lieu de p1 + p2 — on gagne le temps de la plus lente
- **≠ Transaction Prisma** : Promise.all = performance (requêtes indépendantes en parallèle), Transaction = atomicité (écritures liées en tout-ou-rien avec rollback)
- Si le jury demande : "Promise.all parallélise des lectures indépendantes. Une transaction garantit l'atomicité d'écritures liées. Deux problèmes différents."

### Pagination offset/limit
- Pattern : `Promise.all([findMany({ skip, take }), count()])` — les deux requêtes en parallèle
- `skip` = OFFSET SQL, `take` = LIMIT SQL, `orderBy` = ORDER BY
- Retour structuré : `{ data, total, page, limit, totalPages }` — contrat API prêt pour le front
- **Frontend** : pagination non implémentée côté front (volume actuel ne le justifie pas). Le backend est prêt, le contrat API est en place — évolution prévue
- Si le jury demande : "Le backend pagine et renvoie les métadonnées. Le front n'affiche pas encore la pagination parce que le volume ne le justifie pas, mais le contrat API est prêt."
- Alternative possible : **cursor-based pagination** (plus performant sur gros volumes, pas de "saut" de pages) — non nécessaire ici

### Pattern cache-aside (lazy loading) — comment ça marche
- Cache-aside = le cache est **à côté** de la base, pas devant. C'est l'application qui gère la lecture/écriture du cache (Redis ne se synchronise pas tout seul avec PostgreSQL)
- Flux en 3 étapes :
  1. **GET /categories** → le service regarde d'abord dans Redis (clé `categories:list`)
  2. **Cache hit** (clé existe) → donnée retournée directement depuis la mémoire, pas de requête PostgreSQL
  3. **Cache miss** (clé absente ou expirée via TTL) → requête PostgreSQL, stockage du résultat dans Redis avec un TTL (1h pour catégories, 24h pour countries), puis retour de la donnée
- **Invalidation sur écriture** : quand un admin crée/modifie/supprime une catégorie → `invalidateCache()` supprime la clé Redis. Le prochain GET trouvera un cache miss et reconstruira le cache avec les données fraîches
- Dans le code : `categories.service.ts` lignes 35-49 (findAll avec cache) + lignes 158-166 (invalidateCache)
- Si le jury demande : "Le cache-aside vérifie Redis en premier, va en base uniquement si le cache est vide, et invalide le cache à chaque écriture pour garantir la fraîcheur des données."

### Dégradation gracieuse Redis — l'API continue sans cache
- Implémentée dans `redis.service.ts` avec un double filet de sécurité :
  1. **Flag `isAvailable()`** : chaque méthode (`get`, `set`, `incr`, `del`) commence par `if (!this.isAvailable()) return null/false/0`. Si Redis est down → valeur neutre retournée immédiatement
  2. **try/catch** : si Redis plante pendant une opération → log de l'erreur + retour valeur neutre (pas d'exception propagée)
- **Concrètement** : si Redis tombe, `GET /categories` → cache miss (le `get` retourne `null`) → la requête va directement en PostgreSQL. L'API continue de fonctionner, juste plus lente (pas de cache)
- Le flag `isConnected` est mis à jour automatiquement par les listeners Redis (`connect` → true, `close`/`error` → false)
- **Brute force en "fail open"** : si Redis est down, `isLocked()` retourne `false` → on ne bloque personne. Compromis assumé : mieux vaut laisser passer les connexions que bloquer tous les utilisateurs
- C'est un choix d'architecture : Redis est un **accélérateur**, pas une dépendance critique. L'application fonctionne sans lui, juste moins bien
- Si le jury demande : "Chaque appel Redis est protégé par un try/catch et un check de disponibilité. Si Redis tombe, l'API continue normalement via PostgreSQL — Redis est un accélérateur, pas une dépendance bloquante."

### Redis plutôt que PostgreSQL pour brute force + JWT blacklist — pourquoi
- **3 raisons** de préférer Redis à PostgreSQL pour ces cas :
  1. **Données éphémères** : les tentatives de login (TTL 15 min) et les tokens blacklistés (TTL = durée de vie du JWT) sont temporaires. En BDD relationnelle, il faudrait un cron pour nettoyer les vieilles entrées → complexité inutile
  2. **Performance** : Redis est en mémoire (~0.1ms par lecture) vs PostgreSQL sur disque (~1-5ms). Le brute force est vérifié à **chaque login**, la blacklist à **chaque requête authentifiée** → la vitesse compte
  3. **TTL natif** : Redis supprime automatiquement les clés expirées. Le compteur de brute force s'efface **tout seul** après 15 min. Pas de cron, pas de maintenance
- **Comment marche le brute force** (`login-attempts.service.ts`) :
  - Clé Redis : `login_attempts:email@example.com`
  - À chaque échec de login → `INCR` sur la clé (atomique) + TTL 15 min posé au premier échec
  - Si compteur ≥ 5 → compte verrouillé (`isLocked()` retourne true)
  - Après 15 min → la clé expire automatiquement → compteur reset à 0
  - Login réussi → `clearAttempts()` supprime la clé immédiatement
  - Même pour un email inexistant, on enregistre la tentative (protection timing attack)
- **Comment marche la JWT blacklist** :
  - Au logout → on stocke le token JWT dans Redis avec un TTL = durée de vie restante du token
  - À chaque requête authentifiée → on vérifie si le token est dans la blacklist Redis
  - Si présent → requête rejetée (token invalidé)
  - Le TTL fait que la clé disparaît automatiquement quand le JWT aurait expiré de toute façon
- Si le jury demande : "Redis est idéal pour les données éphémères à accès fréquent : TTL natif pour le nettoyage automatique, opérations atomiques pour le compteur, et temps de réponse sub-milliseconde."

### INCR Redis — opération atomique
- `INCR` incrémente un compteur de 1 en une seule opération **atomique** (pas de read-then-write)
- Même si 10 requêtes arrivent en même temps → chaque INCR voit la bonne valeur, pas de race condition
- C'est ce qui rend le compteur de brute force fiable en concurrence
- En PostgreSQL, il faudrait un `UPDATE SET count = count + 1` avec verrouillage → plus complexe et plus lent
- Si le jury demande : "INCR est atomique — pas besoin de transaction ni de verrou pour garantir la cohérence du compteur."

## RGPD — Protection des données personnelles

### Suppression de compte : anonymisation vs suppression pure
- **Le conflit juridique** : le RGPD (Article 17) donne le **droit à l'effacement** ("droit à l'oubli"). Mais le Code Général des Impôts (Article L123-22) impose de **conserver les documents comptables pendant 6 ans** minimum. Deux obligations légales contradictoires
- **La solution : anonymisation des données nominatives + conservation des données comptables**
- **Ce qui se passe concrètement** quand un utilisateur supprime son compte (tout dans une seule **transaction Prisma**) :
  1. **Anonymise les adresses de commande** : `firstName` → "Utilisateur supprimé", `street` → "Adresse supprimée", `phone` → null, etc. Les données nominatives disparaissent
  2. **Anonymise l'email de contact** sur les commandes → `deleted@anonymized.local`
  3. **Supprime le panier** et ses items (plus besoin)
  4. **Supprime l'utilisateur** (compte, email, mot de passe hashé — tout disparaît de la table `user`)
- **Ce qui reste** : les commandes (montants, statuts, numéros de facture, date) et les items commandés (quantité, prix unitaire, TVA). Ce sont les données comptables obligatoires — elles ne permettent plus d'identifier personne puisque les noms/adresses/emails sont anonymisés
- **Pourquoi pas tout supprimer ?** Si un contrôle fiscal arrive, on doit pouvoir justifier les transactions. Une commande à 150€ avec facture n°GR-2025-0042 doit rester traçable comptablement. Mais le fisc n'a pas besoin de savoir que c'est Jean Dupont au 12 rue des Lilas — juste le montant et la TVA
- **Pourquoi une transaction** : si l'anonymisation des adresses réussit mais la suppression de l'utilisateur échoue → état incohérent (données anonymisées mais compte toujours actif). La transaction garantit le tout-ou-rien
- Si le jury demande : "On anonymise les données nominatives (noms, adresses, email) mais on conserve les données comptables (montants, factures) — obligation légale de 6 ans. Le tout dans une transaction pour garantir la cohérence."

### Export de données (RGPD Article 15 — droit d'accès)
- L'utilisateur peut demander **toutes ses données personnelles** en format structuré (JSON)
- Implémenté dans `exportUserData()` : profil complet, commandes avec détails, panier, adresses
- **`safeSelect`** : le select exclut les champs sensibles (mot de passe hashé, tokens) — on ne renvoie **jamais** les données sensibles, même à l'utilisateur lui-même
- C'est l'autre face du RGPD : le droit à l'effacement (Article 17) ET le droit d'accès (Article 15) sont complémentaires
- Si le jury demande : "L'export inclut toutes les données personnelles sauf les champs de sécurité (mot de passe, tokens). L'utilisateur peut exercer son droit d'accès via un endpoint dédié."

### Bannière cookies — pourquoi on n'en a pas
- GreenRoots n'utilise **aucun cookie de tracking** ni analytics (pas de Google Analytics, pas de pixels de suivi)
- Le seul cookie est le **JWT HttpOnly** = cookie **strictement nécessaire** au fonctionnement de l'authentification
- La directive ePrivacy (et le RGPD) **exemptent** les cookies strictement nécessaires du consentement
- Si le jury demande : "Pas de bannière cookies parce qu'on n'a que le cookie d'authentification, qui est strictement nécessaire et exempté de consentement par la directive ePrivacy."

## Tests

### Comment tester un webhook Stripe sans appeler Stripe (mocking)
- On crée un **mock** : un faux objet Stripe qui simule le comportement réel
- Au lieu d'appeler `stripe.webhooks.constructEvent()` pour de vrai, le test remplace cette méthode par une fonction contrôlée
- Cas testés : signature invalide → BadRequestException, `payment_intent.succeeded` → commande passe en PAID+CONFIRMED
- **Le principe** : on isole le code qu'on teste de ses dépendances externes (Stripe, BDD) pour tester uniquement la logique métier
- Si le jury demande : "On utilise des mocks pour simuler les appels Stripe. Le test vérifie que notre code réagit correctement à chaque type d'événement, sans dépendre de l'infrastructure externe."

### Comment tester un store Zustand (cartStore)
- Pas besoin de monter un composant React — on accède directement au store via `useCartStore.getState()` et `useCartStore.setState()`
- On appelle les actions (`addItem`, `syncWithStock`) et on vérifie l'état résultant
- C'est comme tester une fonction pure : entrée → action → vérification sortie
- `beforeEach` : reset du store pour isoler chaque test
- **Cas de test `syncWithStock()`** — 4 scénarios :
  1. **Stock suffisant** → items inchangés, `itemsChangeStock` vide
  2. **Stock insuffisant** → quantité ajustée au stock serveur, `itemsChangeStock` trace le changement (productId, ancienne qté, nouveau stock)
  3. **Stock à 0** → quantité mise à 0, item conservé (pas supprimé)
  4. **Produit absent de la réponse serveur** → item gardé tel quel (cas défensif)
- Si le jury demande : "Le store se teste comme une fonction pure. Pour syncWithStock, on couvre le happy path (stock OK), l'ajustement (stock insuffisant), et les cas limites (stock 0, produit absent)."

### Comment tester un composant React (Testing Library)
- On **rend** le composant dans un environnement simulé (jsdom, pas un vrai navigateur)
- On simule des interactions : `userEvent.click()` sur un bouton, saisie de texte
- On vérifie le résultat dans le DOM : `screen.getByRole("spinbutton")`, `screen.getByLabelText()`
- On vérifie aussi l'accessibilité : attributs ARIA (`aria-valuemin`, `aria-valuemax`, `aria-valuenow`)
- **Philosophie Testing Library** : tester du point de vue de l'utilisateur (ce qu'il voit, ce qu'il clique), pas le code interne du composant
- Si le jury demande : "On teste les composants comme un utilisateur les utiliserait — on clique, on vérifie ce qui s'affiche. Testing Library encourage cette approche plutôt que de tester l'implémentation interne."

### Pyramide de tests — pourquoi cette répartition
- **Base large** : tests unitaires (rapides, isolés, nombreux) — services backend, stores frontend
- **Milieu** : tests d'intégration (moins nombreux, plus lents) — dans GreenRoots : CI avec BDD réelle sur `develop-ci`
- **Sommet** : tests E2E (peu, coûteux) — absents dans GreenRoots, axe d'amélioration identifié
- La priorité a été mise sur les **tests de logique métier critique** (checkout, paiement, panier) plutôt que sur la couverture exhaustive
- Si le jury demande : "On a priorisé les tests unitaires sur la logique métier critique — checkout, paiement, gestion du stock. C'est pragmatique pour 4 sprints d'une semaine."

### Jest vs Vitest — pourquoi deux frameworks
- **Jest** (backend) : standard de l'écosystème NestJS, intégré par défaut avec `@nestjs/testing`
- **Vitest** (frontend) : compatible avec la config Vite (même résolution de modules, même transpilation), plus rapide que Jest pour du code ESM/TypeScript
- Les deux ont une API quasi identique (`describe`, `it`, `expect`, `vi.fn()` / `jest.fn()`)
- Si le jury demande : "Jest pour le backend parce que c'est le standard NestJS, Vitest pour le frontend parce qu'il partage la config Vite — pas de double configuration."

## React / State management

### Zustand vs Context API — pourquoi Zustand pour le panier
- **Context API** : nécessite un Provider wrapper → tout changement de state re-rend **tous** les enfants du Provider
- **Zustand** : store JS **externe** à React (simple objet en mémoire créé par `create()`)
- **3 avantages** :
  1. **Re-render granulaire** : chaque composant souscrit via un sélecteur (`state => state.items`). Zustand compare ancien/nouveau résultat (`===`) → re-rend **uniquement** si le résultat du sélecteur a changé
  2. **Testabilité** : store JS pur → `useCartStore.getState()` / `.setState()` sans monter de composant React
  3. **Persistance native** : middleware `persist` synchronise automatiquement avec localStorage. `partialize` permet de ne persister que certaines propriétés (items + createdAt, pas les données temporaires)
- **Comment ça marche** : `create()` crée un objet JS en mémoire. Le hook `useCartStore` utilise `useSyncExternalStore` (React 18) pour souscrire le composant. Quand `set()` est appelé → state mis à jour → sélecteurs réévalués → re-render ciblé
- Si le jury demande : "Zustand est un store JS externe avec des sélecteurs granulaires — seuls les composants dont la donnée sélectionnée change re-rendent. Le middleware persist gère localStorage nativement. Et c'est testable sans React."

### Panier — flux réhydratation + synchronisation stock
- **Au chargement** : le middleware `persist` lit `localStorage["cart-storage"]` et réhydrate le state (items + createdAt) automatiquement
- **Le store ne contient que `productId` + `quantity`** — jamais les prix/noms/images (évite les données obsolètes en localStorage)
- **`useCartProducts()`** enrichit les items locaux avec les données serveur :
  1. Lit les items du store Zustand
  2. `useQuery` → `POST /cart/batch-details` avec les IDs et quantités → retourne prix, stock, détails
  3. Au retour : cache chaque arbre dans TanStack Query (`setQueryData`) + **`syncWithStock()`** ajuste les quantités si stock insuffisant
  4. Merge items locaux + données serveur → `cartProducts` avec prix et total calculé
- **`syncWithStock()`** : si stock serveur (2) < quantité locale (5) → ajuste à 2 dans Zustand + trace le changement dans `itemsChangeStock` pour notifier l'utilisateur
- **Rafraîchissement** : `refetchOnWindowFocus: true` + `refetchInterval: 5min` → stock toujours frais
- **`enabled: items.length > 0`** → pas de requête si panier vide
- Si le jury demande : "Le panier localStorage ne stocke que les IDs et quantités. Au montage, un hook enrichit via l'API et synchronise le stock — si un arbre n'est plus disponible, la quantité est ajustée automatiquement."

## Éco-conception
- Cohérence mission reforestation + démarche technique

### Code splitting — comment ça marche concrètement
- **Au build** : Vite voit les `import()` dynamiques et découpe le bundle en **chunks séparés** (un par route). Le build produit tous les fichiers, mais le navigateur ne les télécharge pas tous d'un coup
- **Dans le code** : `lazyRouteComponent(() => import("@/features/home/Home"))` — l'import dynamique dit à Vite "crée un chunk séparé pour ce composant"
- **Au runtime** : quand l'utilisateur navigue vers une route, TanStack Router appelle l'`import()` → le navigateur **télécharge le chunk** de cette route uniquement → React rend le composant
- **Cache Nginx** : `Cache-Control: public, immutable` + `expires 1y` sur les assets → le chunk n'est téléchargé **qu'une seule fois**, mis en cache navigateur
- Sans code splitting : 1 gros fichier JS → tout téléchargé au premier chargement. Avec : chunks à la demande → démarrage rapide
- Si le jury demande : "Vite découpe le bundle en chunks au build via les imports dynamiques. Le navigateur ne télécharge que le chunk de la route visitée. Le cache Nginx immutable évite les re-téléchargements."

### Autres mesures éco-conception
- Vite = build léger vs Webpack
- Tailwind = purge CSS (seules les classes utilisées sont incluses)
- shadcn/ui = import sélectif (pas de bibliothèque entière chargée)
- Redis = réduit les requêtes SQL redondantes
- Police Montserrat hébergée en local (pas de requête externe Google Fonts)
- Images Cloudinary : transformations côté CDN (pas de traitement serveur)
- Rendu conditionnel JS (`useMediaQuery`) plutôt que CSS hidden → composants lourds non montés sur mobile

---

## Questions jury — Points identifiés lors de l'audit de cohérence

> Questions potentielles du jury sur des points techniques du dossier, avec réponses préparées.

### Transactions stock — "C'est du verrouillage pessimiste ?"
- **Non.** GreenRoots utilise des **opérations atomiques** dans une transaction Prisma (`$transaction` + `decrement`), pas du verrouillage pessimiste (pas de `SELECT ... FOR UPDATE`)
- L'opération `decrement` est atomique au niveau SQL — pas de race condition sur le décrement lui-même
- La transaction garantit que la validation du stock et le décrement sont exécutés dans la même unité de travail
- En cas de concurrence très élevée, une fenêtre existe entre la lecture du stock et le décrement. Pour un MVP e-commerce, c'est acceptable. Pour un site à très fort trafic, un verrouillage pessimiste (`FOR UPDATE`) ou un système de queue serait préférable
- Si le jury demande : "On utilise des transactions atomiques Prisma avec l'opérateur decrement, qui est atomique côté SQL. Ce n'est pas du verrouillage pessimiste au sens strict, mais c'est suffisant pour le volume du MVP."

### SameSite=Lax et CSRF — "Votre protection CSRF est suffisante ?"
- **SameSite=Lax** empêche l'envoi du cookie lors de requêtes cross-origin initiées par des formulaires ou des scripts (form submissions POST, fetch/XHR)
- Il autorise l'envoi du cookie pour les navigations top-level (clic sur un lien GET) — c'est la différence avec Strict
- **CORS restrictif** (origin unique = FRONTEND_URL, credentials: true) bloque toute requête AJAX depuis un autre domaine
- Combinaison SameSite=Lax + CORS = protection CSRF suffisante dans une SPA où toutes les mutations passent par des appels API (POST/PUT/DELETE via fetch/axios), pas par des formulaires HTML classiques
- Un token CSRF classique (champ caché dans un formulaire) n'est pas nécessaire dans cette architecture
- **Limite** : navigateurs très anciens ne supportant pas SameSite — mais on cible les 2 dernières versions majeures, donc pas de risque
- Si le jury demande : "On s'appuie sur SameSite=Lax pour le cookie et sur le CORS restrictif côté serveur. Dans une SPA, les mutations passent toujours par des appels API, jamais par des formulaires HTML classiques — un token CSRF n'apporte pas de protection supplémentaire dans ce contexte."

### Fail-open Redis — "Que se passe-t-il si Redis tombe ?"
- Si Redis est indisponible, la protection brute force login **ne s'applique pas** (fail-open : on autorise le login plutôt que de bloquer tout le monde)
- C'est un choix délibéré : **disponibilité > protection temporaire**. Refuser tous les logins quand Redis est down (fail-closed) serait pire pour les utilisateurs
- **Le ThrottlerGuard global reste actif** indépendamment de Redis — il limite les requêtes par IP sur toute l'API, ce qui constitue un premier rempart même sans Redis
- Le cache API (countries, categories) est aussi en dégradation gracieuse : si Redis est down, les requêtes passent directement à PostgreSQL
- En production, un monitoring Redis avec alertes permettrait de réagir rapidement. Redis est un accélérateur et une couche de protection, pas une dépendance bloquante
- Si le jury demande : "On a choisi une stratégie fail-open : si Redis tombe, on perd le rate limiting spécifique au login, mais le ThrottlerGuard global continue de protéger l'API. C'est un compromis disponibilité vs sécurité temporaire — en production, une alerte sur la santé de Redis permettrait d'intervenir rapidement."

### Architecture hexagonale — "Prisma est votre repository ?"
- Les services injectent directement `PrismaService` — **pas d'interface d'abstraction** (port) entre le service métier et l'ORM
- Dans une architecture hexagonale stricte (ports & adapters), on définirait une interface `OrderRepository` que Prisma implémenterait, permettant de changer d'ORM sans toucher au service
- GreenRoots utilise l'architecture **modulaire NestJS classique** : Controller → Service → PrismaService, avec l'injection de dépendances du framework
- Ce n'est pas hexagonal au sens strict, mais la séparation des responsabilités est claire et la testabilité est assurée grâce à l'injection de dépendances (on peut mocker PrismaService dans les tests)
- C'est un **compromis pragmatique** pour un MVP : le coût d'abstraction supplémentaire n'est pas justifié quand on n'envisage pas de changer d'ORM
- Si le jury demande : "Les services dépendent directement de PrismaService, donc ce n'est pas hexagonal au sens strict — il faudrait des interfaces intermédiaires. Mais l'injection de dépendances NestJS assure le découplage pour les tests, et sur un MVP, l'abstraction supplémentaire n'apporte pas de valeur concrète."

### Panier client-side — "Pourquoi des tables Cart/CartItem en base si le panier est côté client ?"
- Le panier est géré côté client (Zustand + localStorage) pour la rapidité d'interaction et le fonctionnement offline
- Le backend expose un endpoint stateless `POST /cart/batch-details` qui enrichit les items du panier avec les données serveur (stock actuel, prix, détails arbre) — c'est un lookup, pas de persistence
- Les modèles Cart et CartItem existent en base (schéma Prisma) mais **ne sont pas utilisés pour la persistence du panier actuellement**
- Ils ont été modélisés en anticipation d'une évolution future : synchronisation multi-appareil (l'utilisateur retrouve son panier sur un autre device)
- Si le jury demande : "Le panier vit côté client pour la réactivité. Le backend sert uniquement à enrichir les données (stock, prix). Les tables Cart/CartItem sont prêtes pour une synchro serveur en évolution future, mais pas utilisées actuellement."

### Prisma et les JOINs — "Prisma fait des requêtes séparées ou des JOINs ?"
- Prisma utilise des **JOINs SQL** (ou des requêtes `IN(...)` latérales) pour résoudre les `include` imbriqués dans une même requête
- Ce n'est PAS "une requête séparée par relation" — c'est optimisé côté moteur Prisma
- Pour un `findUnique` avec `include: { items: { include: { tree: { include: { stock: true } } } } }`, Prisma génère des JOINs ou des sous-requêtes batchées selon le cas
- La nuance : Prisma n'utilise pas toujours un seul `SELECT ... JOIN` classique — il peut découper en requêtes `SELECT ... WHERE id IN (...)` pour les relations 1-N, ce qui est souvent plus performant que des JOINs massifs
- Pour le volume de GreenRoots, la différence est négligeable. Pour des jeux de données volumineux, Prisma supporte `$queryRaw` pour du SQL natif optimisé
- Si le jury demande : "Prisma optimise les includes en combinant JOINs et requêtes batchées IN(...) selon la relation. Pour notre volume, c'est transparent. Si on avait besoin de SQL très optimisé, Prisma expose $queryRaw."

### Prisma — comment fonctionne le chargement des relations
- Par défaut, un `include` génère **des requêtes séparées batchées avec `WHERE ... IN (...)`**, pas des JOINs
- Exemple : `findMany({ include: { posts: true } })` produit 2 requêtes SQL :
  1. `SELECT * FROM "User" WHERE 1=1`
  2. `SELECT * FROM "Post" WHERE "authorId" IN ($1, $2, $3, $4)`
- C'est **une requête par niveau de relation** : arbres → stocks → catégories → localisations = 4 requêtes
- Ce comportement résout le problème N+1 (pas une requête par arbre, mais une seule requête batchée pour tous les stocks)
- Depuis Prisma 5.9+, il existe `relationLoadStrategy: "join"` (opt-in) qui utilise des LATERAL JOINs (PostgreSQL) ou des sous-requêtes corrélées (MySQL) — une seule requête SQL
- GreenRoots utilise le **comportement par défaut** (requêtes batchées), suffisant pour le volume du projet
- Les requêtes sont toujours **paramétrées** (pas de concaténation SQL → pas d'injection)
- Si le jury demande "pourquoi pas des JOINs ?" : "Prisma utilise par défaut des requêtes batchées IN(...) plutôt que des JOINs. C'est suffisant pour notre volume. Depuis Prisma 5.9, une option relationLoadStrategy: join permet d'utiliser des LATERAL JOINs si nécessaire."

### Factures — "Votre SIRET est à zéros, c'est conforme ?"
- C'est un projet de formation — les données vendeur (SIRET, TVA intracommunautaire, capital social) sont **fictives et placeholder**
- La **structure du PDF** respecte les mentions obligatoires du CGI art. 289 : identification vendeur, numéro de facture séquentiel unique (FA-YYYY-NNNNNN), date de facturation, détail des articles avec TVA (HT + taux + montant TVA + TTC)
- En production, ces constantes seraient alimentées par la configuration réelle de l'entreprise
- L'important pour le jury : la démarche est complète (calcul TVA, stockage en base, génération PDF, endpoint téléchargement, bouton frontend) — les données sont fictives par nécessité
- Si le jury demande : "Les données vendeur sont fictives — c'est un projet de formation. Mais la structure est conforme : numéro séquentiel atomique, mentions obligatoires, TVA calculée et stockée en base. En production, on remplacerait les constantes par les données réelles de l'entreprise."

### RGPD anonymisation — "Les noms des arbres commandés restent visibles ?"
- L'anonymisation porte sur les **données personnelles** : nom, prénom, adresse, email, téléphone dans OrderAddress et Order
- Les données de commande (arbres, quantités, montants) sont des **données comptables et produit**, pas des données personnelles au sens du RGPD
- Le nom d'un arbre ("Chêne sessile") n'identifie pas une personne, même indirectement
- Les montants et articles sont conservés pour l'obligation légale de conservation comptable (6 ans, art. L123-22 Code de commerce)
- Si le jury demande : "Les noms d'arbres sont des données produit, pas des données personnelles — les anonymiser n'aurait pas de sens juridique. On anonymise ce qui identifie la personne (nom, adresse, email) tout en conservant les données comptables, comme l'exige la loi."

### Lighthouse — "Vous annoncez 95+ en performance ?"
- Les scores ont été mesurés sur un **build de preview** (production build servi localement)
- Performance 95+, Accessibilité 100, SEO 100 — captures disponibles pour démonstration
- Ces scores sont favorisés par : code splitting (35+ routes lazy-loaded), CSS critique inline (plugin Critters), polices locales préchargées, images Cloudinary optimisées, manual chunks Vite
- **Limite honnête** : un score en preview ne reflète pas exactement la production (latence réseau, serveur, CDN). En production réelle, le score dépendrait aussi de l'infrastructure
- Si le jury demande : "Les scores ont été mesurés en build preview. Les optimisations sont dans le code — code splitting, CSS critique, polices locales. En production, les scores dépendraient aussi de l'infrastructure serveur."
