# GreenRoots - Récap technique pour l'oral

> Vue globale de l'application avec **extraits de code réels** et références
> `chemin/fichier.ts:ligne`. Objectif : avoir l'image du code en tête et pouvoir
> dérouler la logique de bout en bout devant le jury.
>
> Tous les chemins backend sont relatifs à `projet-greenroots/app/backend/src/`,
> les chemins frontend à `projet-greenroots/app/frontend/src/`.

---

## 1. En une phrase

**GreenRoots** est une application e-commerce de vente d'arbres pour la reforestation :
un visiteur parcourt un catalogue d'arbres rattachés à des forêts géolocalisées, remplit
un panier, paie par carte (Stripe), reçoit une facture PDF ; un back-office admin gère le
catalogue et fait évoluer les commandes jusqu'à la plantation.

```
┌───────────────────────┐        HTTP/JSON (cookie httpOnly)      ┌──────────────────────────┐
│   FRONTEND (SPA)       │  ────────────────────────────────────► │   BACKEND (API REST)      │
│   React 19 + Vite      │  ◄────────────────────────────────────  │   NestJS 11 + Prisma 7    │
│   TanStack Router/Query│                                          └──────────┬───────────────┘
│   Zustand              │                                                     │
└───────────────────────┘        ┌──────────┬─────────┬──────────┬────────────┼─────────┐
                              ┌───▼────┐ ┌───▼───┐ ┌───▼────┐ ┌───▼─────┐ ┌────▼─────┐
                              │Postgres│ │ Redis │ │ Stripe │ │Cloudinary│ │  Resend │
                              │+PostGIS│ │cache/ │ │paiement│ │ images   │ │  emails  │
                              │        │ │lockout│ │        │ │          │ │          │
                              └────────┘ └───────┘ └────────┘ └──────────┘ └──────────┘
```

Le backend est le **seul** à parler à la base, à Stripe, à Cloudinary, à Resend.
Le frontend ne fait que consommer l'API.

---

## 2. Le backend NestJS

### 2.1 Le vocabulaire (à maîtriser)

NestJS structure le code en **modules**. Chaque module regroupe :

- **Controller** = porte d'entrée HTTP : déclare les **routes**, applique les **guards**,
  délègue au service. Pas de logique métier.
- **Service** = la logique métier (le cerveau), qui parle à la base via Prisma.
- **DTO** = la forme attendue des données, validée automatiquement.
- **Module** = câble controller + services + dépendances (injection de dépendances).

Analogie : le controller = le serveur en salle (prend la commande, vérifie le droit d'entrée),
le service = la cuisine (prépare le plat), Prisma = le passe-plat vers le garde-manger (la base).

### 2.2 Le démarrage global (`main.ts`)

```ts
// main.ts:13-47
const app = await NestFactory.create<NestExpressApplication>(AppModule, {
  logger: ['error', 'warn', 'log', 'debug', 'verbose'],
  rawBody: true,                       // ← corps brut requis pour la signature du webhook Stripe
});

app.setGlobalPrefix('api');            // ← toutes les routes sous /api
app.use(cookieParser());               // ← pour lire le cookie qui contient le JWT
app.use(helmet({ hsts: process.env.NODE_ENV === 'production' }));

app.enableCors({
  origin: configService.get<string>('FRONTEND_URL'),   // ← seul le front est autorisé
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type'],
  credentials: true,                   // ← indispensable pour que le cookie circule
});

app.useGlobalPipes(new ValidationPipe({
  whitelist: true,                     // ← supprime les champs non déclarés dans le DTO
  forbidNonWhitelisted: true,          // ← rejette une requête avec un champ inconnu
  transform: true,                     // ← convertit les types (ex: "3" → 3)
  transformOptions: { enableImplicitConversion: true },
}));
```

**Ce qu'il faut voir** : la validation des entrées est **globale et automatique** (aucun DTO
ne peut recevoir un champ non prévu), le CORS est **restreint** au domaine du front, et
`rawBody: true` sert exclusivement à Stripe (§5).

### 2.3 La config validée + les guards globaux (`app.module.ts`)

```ts
// app.module.ts:27-54  — validation des variables d'environnement au démarrage
ConfigModule.forRoot({
  isGlobal: true,
  validationSchema: Joi.object({
    DATABASE_URL: Joi.string().required(),
    JWT_SECRET: Joi.string().min(32).required(),   // ← refuse de démarrer si secret faible
    JWT_EXPIRATION: Joi.string().required(),
    RESEND_API_KEY: Joi.string().required(),
    FRONTEND_URL: Joi.string().uri().required(),
    // ... Cloudinary, REDIS_URL (optionnel)
  }),
}),
// Rate limiting global : 60 req/min/IP
ThrottlerModule.forRoot({ throttlers: [{ ttl: 60000, limit: 60 }] }),
```

```ts
// app.module.ts:73-80  — le SEUL guard appliqué globalement
providers: [
  AppService,
  { provide: APP_GUARD, useClass: ThrottlerGuard },   // ← rate limiting sur TOUTES les routes
],
```

**Ce qu'il faut voir** : si une variable d'environnement manque, **l'app ne démarre pas**.
Le seul guard global est le rate limiter ; JwtAuthGuard et RolesGuard sont appliqués
route par route.

### 2.4 Les modules et leurs routes (vue synthétique)

Chemins préfixés par `/api`. Accès : **Public** (aucun guard), **Auth** (JwtAuthGuard),
**Admin** (RolesGuard + rôle requis).

| Module | Routes principales | Accès |
|---|---|---|
| **auth** | `POST /auth/login`, `/auth/register`, `GET /auth/me`, `PATCH /auth/me`, `DELETE /auth/me`… | mixte (§3) |
| **users** | `GET /users`, `GET /users/:id` | Admin |
| **trees** | `GET /trees`, `GET /trees/:id` (public) ; `POST/PATCH/DELETE` (admin) | Public + Admin |
| **categories** | `GET` (public, cache Redis) ; `POST/PATCH/DELETE` (admin) | Public + Admin |
| **countries** | `GET /countries/billing`, `/countries/planting` | Public (cache) |
| **locations** | `GET` (public) ; `POST/PATCH/DELETE` (admin) — PostGIS | Public + Admin |
| **cart** | `POST /cart/batch-details` (enrichit le panier, lecture seule) | Public |
| **checkout** | `POST /checkout/init`, `POST /checkout/confirm` | Auth |
| **payment** | `POST /payment/webhook` | Public (signé Stripe) |
| **orders** | `GET /orders/me`, `GET /orders/:id/invoice`, `GET /orders` (admin), `PATCH /orders/:id` (admin) | Auth + Admin |
| **cloudinary** | `POST /cloudinary/signature` | Auth |
| **stripe / database / redis** | modules `@Global()`, pas de route | — |

**À savoir dire** : le **panier n'est PAS en base**. Il vit dans le navigateur (localStorage).
Les tables `carts`/`cart_items` ont même été **supprimées par migration** — on a fait marche
arrière. Le back ne connaît le panier qu'au checkout, où il devient une commande (`Order`).

**Exemple concret de déclaration de routes** (le controller ne fait que router + garder) :

```ts
// orders/orders.controller.ts:33-59
@ApiTags('orders')
@UseGuards(JwtAuthGuard)          // ← guard au niveau CLASSE : toutes les routes exigent d'être connecté
@Controller('orders')
export class OrdersController {

  @Get('me')                      // GET /api/orders/me
  @Serialize(OrderResponseDto)    // ← filtre la réponse via un DTO
  findMyOrders(@CurrentUser() user, @Query() query) {
    return this.ordersService.findUserOrders(user.userId, query);
  }

  @Get()                          // GET /api/orders
  @UseGuards(RolesGuard)          // ← guard SUPPLÉMENTAIRE au niveau MÉTHODE
  @Roles('ADMIN')                 // ← rôle requis
  findAll(@Query() query) {
    return this.ordersService.findAll(query);
  }
}
```

**Ce qu'il faut voir** : les guards se cumulent (classe + méthode) et le controller ne
contient **aucune** logique métier — il délègue tout au service.

---

## 3. Authentification et guards (le point que le jury adore)

### 3.1 JWT dans un cookie httpOnly

```ts
// auth/utils/cookie.util.ts  (fichier complet)
export const COOKIE_NAME = 'access_token';

export function getCookieOptions(isProduction: boolean): CookieOptions {
  return {
    httpOnly: true,                 // ← le JavaScript du navigateur NE PEUT PAS lire le cookie (anti-XSS)
    secure: isProduction,           // ← HTTPS uniquement en production
    sameSite: 'lax',                // ← limite le CSRF
    path: '/',
    maxAge: 24 * 60 * 60 * 1000,    // ← 1 jour, aligné sur JWT_EXPIRATION
  };
}
```

À la connexion, le token est déposé dans ce cookie ; le front n'a **rien** à gérer :

```ts
// auth/auth.controller.ts:55-78
@UseGuards(LocalAuthGuard)                          // ← vérifie email + mot de passe
@Throttle({ default: { limit: 5, ttl: 60000 } })    // ← durcit : 5 tentatives/min sur le login
@Post('login')
async login(@Request() req, @Res({ passthrough: true }) res: Response) {
  const { token, message } = await this.authService.login(req.user);
  res.cookie(COOKIE_NAME, token, getCookieOptions(this.isProduction));  // ← dépose le JWT en cookie
  return { message };
}
```

> Phrase clé : « Le JWT est stocké dans un cookie httpOnly, pas dans le localStorage, pour se
> protéger du vol de token via XSS. En contrepartie j'active `sameSite: lax` et un CORS restreint
> pour limiter le CSRF. Un seul token d'accès, pas de refresh token : à l'expiration (1 jour),
> l'utilisateur se reconnecte. »

### 3.2 Passport : deux stratégies

**LocalStrategy** (à la connexion) → délègue à `validateUser` :

```ts
// auth/auth.service.ts:36-84 (extrait)
async validateUser(email, pass) {
  if (await this.loginAttemptsService.isLocked(email)) {   // ← compte verrouillé ?
    throw new HttpException('Compte temporairement bloqué...', HttpStatus.TOO_MANY_REQUESTS);
  }
  const user = await this.usersService.findOneByEmailForAuth(email);
  if (!user) {
    await this.loginAttemptsService.recordFailure(email);  // ← compte l'échec même si l'email n'existe pas
    return null;
  }
  const match = await bcrypt.compare(pass, user.password); // ← comparaison bcrypt
  if (!match) {
    const attempts = await this.loginAttemptsService.recordFailure(email);
    if (attempts >= 5) throw new HttpException('...bloqué 15 minutes.', 429);
    return null;
  }
  if (!user.isEmailVerified) throw new UnauthorizedException('Email non vérifié.');
  await this.loginAttemptsService.clearAttempts(email);    // ← reset des compteurs
  const { password, ...result } = user;                    // ← ne jamais renvoyer le hash
  return result;
}
```

Puis `login` signe le token :

```ts
// auth/auth.service.ts:86-101
async login(user) {
  const payload = { email: user.email, sub: user.id, role: user.role?.name || 'USER' };
  return { token: await this.jwtService.signAsync(payload), message: 'Connexion réussie.' };
}
```

**JwtStrategy** (toutes les autres requêtes) → extrait le token du cookie et le valide :

```ts
// auth/strategies/jwt.strategy.ts:43-92 (extrait)
async validate(req, payload) {
  const token = req.cookies?.[COOKIE_NAME];
  if (token && await this.tokenBlacklistService.isBlacklisted(token)) {  // 1) token révoqué (logout) ?
    throw new UnauthorizedException('Token révoqué - veuillez vous reconnecter');
  }
  const user = await this.prisma.user.findUnique({          // 2) l'utilisateur existe-t-il ENCORE ?
    where: { id: payload.sub },
    select: { id: true, email: true, isEmailVerified: true, role: { select: { name: true } } },
  });
  if (!user) throw new UnauthorizedException('Utilisateur introuvable');
  if (!user.isEmailVerified) throw new UnauthorizedException('Email non vérifié');

  return { userId: user.id, email: user.email, role: user.role.name.toUpperCase() };  // 3) rôle LU EN BASE
}
```

> Détail malin à dire : « Je relis le rôle **depuis la base** à chaque requête plutôt que de faire
> confiance au rôle inscrit dans le token. Si un admin est rétrogradé, l'effet est immédiat, sans
> attendre l'expiration du token. »

### 3.3 Les guards (les videurs à l'entrée)

Un guard répond à une question : « cette requête a-t-elle le droit de passer ? ». Il s'exécute
**avant** le controller.

| Guard | Rôle | Où |
|---|---|---|
| **ThrottlerGuard** | 60 req/min/IP | global |
| **LocalAuthGuard** | email + mot de passe | seulement `POST /auth/login` |
| **JwtAuthGuard** | cookie JWT valide → injecte `req.user` | routes « Auth » |
| **RolesGuard** | `req.user.role` dans les rôles autorisés | routes « Admin » |

Le `JwtAuthGuard` est minimal (il branche la stratégie Passport `'jwt'`) :

```ts
// auth/guards/jwt-auth.guard.ts (fichier complet)
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```

Le `RolesGuard` lit les rôles requis via les métadonnées posées par `@Roles(...)` :

```ts
// auth/guards/roles.guard.ts:14-51 (extrait)
canActivate(context: ExecutionContext): boolean {
  const requiredRoles = this.reflector.getAllAndOverride<string[]>(
    ROLES_KEY, [context.getHandler(), context.getClass()],   // ← lit @Roles sur méthode OU classe
  );
  if (!requiredRoles || requiredRoles.length === 0) return true;   // ← aucune restriction → passe

  const user = context.switchToHttp().getRequest().user;    // ← utilisateur injecté par JwtAuthGuard
  if (!user) throw new ForbiddenException('Utilisateur non authentifié');

  const userRole = user.role?.toUpperCase() || 'USER';     // ← RBAC à 2 rôles : USER ou ADMIN
  const hasRole = requiredRoles.some((r) => r.toUpperCase() === userRole);
  if (!hasRole) throw new ForbiddenException(`Accès refusé. Rôle requis: ${requiredRoles.join(' ou ')}`);
  return true;
}
```

> **RBAC à 2 rôles** (`user` / `admin`) : c'est la version passée en production (commit
> « release: RBAC à 2 rôles » sur `main`). Il n'y a **pas** de rôle super-admin. Le rôle
> opérationnel qui donne accès au back-office est **`admin`**. Les rôles sont une **table**
> (`Role.name`), pas un enum : au seed, deux rôles sont créés — `admin` et `user`.
> L'attribution d'un rôle se fait **au seed** ; il n'y a pas d'endpoint de changement de rôle
> (la route `PATCH /users/:id/role`, gardée par un rôle inexistant et donc inaccessible à tous,
> a été supprimée lors du nettoyage RBAC).

**Ordre d'enchaînement** sur une route admin : `@UseGuards(JwtAuthGuard, RolesGuard)`.
D'abord JwtAuthGuard **authentifie** (et remplit `req.user`), **ensuite** RolesGuard **autorise**.
L'ordre compte : sans JwtAuthGuard avant, RolesGuard n'aurait pas d'utilisateur à contrôler.

### 3.4 Les décorateurs maison

```ts
// auth/decorators/roles.decorator.ts (fichier complet)
export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);  // ← pose des métadonnées
```

- **`@Roles('ADMIN')`** : liste les rôles autorisés (lue par RolesGuard).
- **`@CurrentUser()`** : injecte `req.user` dans un paramètre de méthode.
- **`@Serialize(Dto)`** : filtre la réponse (`excludeExtraneousValues`) → jamais de champ sensible exposé.
- **`@Throttle(5, 60s)`** : durcit le rate limiting sur login / forgot-password / reset.

### 3.5 Les protections « bonus » (à citer côté sécurité)

- **Anti-brute-force** (`auth/services/login-attempts.service.ts`, via Redis) : 5 échecs →
  verrou 15 min. Clé `login_attempts:{email}`.
- **Blacklist de tokens** (`auth/services/token-blacklist.service.ts`, via Redis) : au logout, le
  token est blacklisté jusqu'à son expiration. Clé `blacklist:{empreinte}`.
- **Tokens email/reset hachés en SHA-256** avant stockage (`common/utils/hash-token.util.ts`) —
  inutilisables si la base fuite. Les mots de passe, eux, sont hachés en **bcrypt (coût 10)**.
- **Anti-énumération** : `forgot-password` renvoie toujours le même message générique
  (`auth.service.ts:352-357`), qu'un compte existe ou non.
- **RGPD** : export (`GET /auth/me/export`, art. 15) et suppression avec anonymisation des
  commandes (`DELETE /auth/me`, `auth.service.ts:285-314`).

> Robustesse : si Redis tombe, lockout et blacklist « fail open » (ils laissent passer plutôt que
> de bloquer l'app). Ce sont des couches de défense, pas le socle de l'auth.

---

## 4. La base de données (PostgreSQL + Prisma + PostGIS)

### 4.1 Prisma, l'ORM

On décrit les tables dans `prisma/schema.prisma`, Prisma génère un client typé (au lieu d'écrire
du SQL). Les évolutions passent par des **migrations** (24 à ce jour). Accès via `PrismaService`
(module `@Global()`).

### 4.2 Les 11 modèles et leurs relations

```
Role  ──1─N──►  User  ──1─N──►  Order  ──1─N──►  OrderItem  ──N─1──►  Tree
                                  │                                     │
                                  └─1─N──► OrderAddress          Tree ──1─1──► TreeStock
                                                                        ▲
Country ──1─N──► Location ──1─N──► Tree ◄──N─1── Category ───────────────┘
```

- **Role → User** (1-N). Les rôles sont une **table** (`admin`, `user`), pas un enum.
- **User → Order** (1-N).
- **Order ↔ Tree** : un **N-N** résolu par **`OrderItem`**, qui porte `quantity`, `unitPrice`
  (**prix figé au moment de l'achat**) et `vatRate`.
  → À dire : « Je stocke le prix unitaire dans la ligne de commande, pas juste une référence
  produit, pour qu'une commande garde son prix même si le catalogue change ensuite. »
- **Order → OrderAddress** (1-N, mais 1 seule `BILLING` + 1 seule `DELIVERY` par commande,
  contrainte unique). Suppression en **cascade**.
- **Tree ↔ TreeStock** (1-1) : chaque arbre a une ligne de stock (quantité + seuil d'alerte).
  Cascade : supprimer un arbre supprime son stock.
- **Country → Location → Tree** : un pays contient des forêts, chaque forêt contient des arbres.
  Le pays porte `isBillingAvailable` et `isPlantingAvailable`.

### 4.3 Les enums de statut (cœur du tunnel)

```prisma
enum OrderStatus   { PENDING  CONFIRMED  PLANTED  CANCELLED }
enum PaymentStatus { PENDING  PROCESSING  PAID  FAILED  REFUNDED }
enum AddressType   { BILLING  DELIVERY }
```

> **Deux statuts distincts** : un pour le **paiement** (état de l'argent), un pour la **commande**
> (état logistique). C'est cette séparation qui rend le tunnel robuste (§5 et §6).

### 4.4 PostGIS (géolocalisation des forêts)

`Location.coordinates` est de type **`geometry`** (point GPS). Prisma ne gère pas ce type
(`Unsupported("geometry")`), donc les lectures/écritures se font en **SQL brut** dans
`locations/locations.service.ts` :

- lecture : `ST_Y(coordinates)` = latitude, `ST_X(coordinates)` = longitude ;
- écriture : `ST_GeomFromText('POINT(lng lat)', 4326)` — **SRID 4326** = le système GPS standard (WGS84).

3 forêts géolocalisées au seed : Fontainebleau (FR), Aletsch (CH), Rold Skov (DK).

> « Pourquoi PostGIS et pas deux colonnes lat/lng ? » → PostGIS permettrait des requêtes
> spatiales (arbres à moins de X km d'un point). C'est un choix d'extensibilité ; ici on n'exploite
> encore que le stockage du point.

### 4.5 L'argent en centimes

Tous les montants sont des **entiers en centimes**, jamais des flottants (pour éviter
`0.1 + 0.2 ≠ 0.3`). La TVA est en points de base : `vatRate = 2000` = 20,00 %
(`common/constants/billing.constants.ts`).

---

## 5. Le tunnel de vente de bout en bout (LE fil rouge)

```
 PANIER            CHECKOUT INIT         CHECKOUT CONFIRM        PAIEMENT          WEBHOOK
(navigateur)  ─►  POST /checkout/init ─► POST /checkout/confirm ─► Stripe (front) ─► POST /payment/webhook
                  crée la commande       réserve le stock         carte bancaire    finalise
                  + PaymentIntent        (décrémente)                               PAID + CONFIRMED
                  PENDING/PENDING        PROCESSING                                 + facture + email
```

### Étape 1 — Panier
Panier **client-side** (Zustand, persisté). `POST /cart/batch-details` (public, **lecture seule**, aucun écrit en base) renvoie prix + stock **à jour** pour les arbres du panier. Il ne décrémente **rien** : la décrémentation viendra à l'étape 3.

Le câblage front (`features/cart/hooks/useCartProducts.tsx`) : un `useQuery` TanStack appelle `cartApi.getBatchDetails`, puis passe la réponse à l'action Zustand `syncWithStock` (ce n'est donc **pas** `syncWithStock` qui fait l'appel HTTP, elle **reçoit** le stock et réconcilie le panier local). La requête se relance :
- au montage / quand le contenu du panier change (le `productIds` est dans la `queryKey`),
- **toutes les 5 min** (`refetchInterval`) et **au retour sur l'onglet** (`refetchOnWindowFocus`),
- **au clic sur « Valider »** (`Cart.handlePurchase` force un `refetch()` avant d'ouvrir le checkout).

**La notification « le stock a changé »** : si `syncWithStock` voit un stock serveur < quantité du panier, il ajuste la quantité et empile l'écart dans `itemsChangeStock` → le composant affiche le dialog `CartAlertDialogChangeStock` (`out-of-stock` / `quantity-reduced`). Elle peut donc apparaître **en simple consultation du panier** (refetch 5 min / focus), pas seulement au moment de payer.

### Étape 2 — Init (`POST /checkout/init`, authentifié)

```ts
// checkout/checkout.service.ts:53-121 (extrait)
async initCheckout(userId, items, contactEmail, billingAddress) {
  return this.prisma.$transaction(async (tx) => {
    const { currency } = await this.validateCountry(tx, billingAddress.countryCode); // pays actif ?
    const existingOrder = await this.findReusableOrder(tx, userId);   // réutilise une commande PENDING < 30 min

    let orderId, totalAmount;
    if (existingOrder) { /* met à jour les items + totaux */ }
    else { /* createNewOrder : items, totaux HT/TVA/TTC, adresse */ }

    // Le STOCK N'EST PAS décrémenté ici (on ne bloque pas de stock pour des paniers abandonnés)
    const clientSecret = await this.handleStripePaymentIntent(tx, orderId, totalAmount);
    return { orderId, clientSecret };   // ← le clientSecret est le laissez-passer pour payer
  });
}
```

Le PaymentIntent Stripe porte l'`orderId` en métadonnée (clé de la réconciliation ensuite) :

```ts
// checkout/checkout.service.ts:460-465
const paymentIntent = await this.stripe.paymentIntents.create({
  amount,
  currency: order!.currency.toLowerCase(),
  metadata: { orderId: orderId.toString() },   // ← lien Stripe ↔ commande
  payment_method_types: ['card'],
});
```

### Étape 3 — Confirm (`POST /checkout/confirm`, authentifié)
Appelé **juste avant** le paiement. C'est ici — et seulement ici — que le stock est réservé.

```ts
// checkout/checkout.service.ts:133-238 (extrait)
async confirmCheckout(userId, orderId) {
  return this.prisma.$transaction(async (tx) => {
    const order = await tx.order.findUnique({ where: { id: orderId }, include: { items: { include: { tree: { include: { stock: true } } } } } });
    if (order.userId !== userId) throw new ForbiddenException(...);       // sécurité : c'est bien sa commande
    if (order.paymentStatus === PaymentStatus.PAID) throw new BadRequestException('déjà payée');

    // Idempotence : double-clic sur "Payer" → stock déjà réservé → on renvoie OK sans re-décrémenter
    if (order.stockReservedAt || order.paymentStatus === PaymentStatus.PROCESSING) return { success: true };

    // Vérifie la disponibilité ; si insuffisant → renvoie les problèmes SANS décrémenter
    const issues = [];
    for (const item of order.items) { /* out-of-stock / insufficient */ }
    if (issues.length > 0) return { success: false, issues };

    // Stock OK → décrémente atomiquement + marque la réservation
    for (const item of order.items) {
      await tx.treeStock.update({ where: { treeId: item.treeId }, data: { quantity: { decrement: item.quantity } } });
    }
    await tx.order.update({ where: { id: orderId }, data: { stockReservedAt: new Date(), paymentStatus: PaymentStatus.PROCESSING } });
    return { success: true };
  });
}
```

**Ce qu'il faut voir** : réservation de stock **transactionnelle** + **idempotente** (gère le
double-clic) + le passage à `PROCESSING` qui déclenchera, si besoin, la réconciliation (§6).

### Étape 4 — Paiement (côté navigateur)
Avec le `clientSecret`, le front affiche **Stripe Elements** (la carte ne transite **jamais** par
notre serveur → conformité PCI) et appelle `stripe.confirmPayment()`. Stripe débite.

**Deux clics distincts** côté UI :
1. **« Valider »** (formulaire d'adresse) → `POST /checkout/init` → on reçoit le `clientSecret` → le formulaire se verrouille et le composant Stripe **se monte** (`<Elements>` n'apparaît **que si** `clientSecret !== null`, `Checkout.tsx:220`).
2. **« Payer »** (dans Elements) → `StripePaymentSection.handleConfirmPayment` : **étape 1** `POST /checkout/confirm` (réserve le stock côté back) → **étape 2** `stripe.confirmPayment()` (la carte part **directement** vers Stripe).

**Trois « secrets » à ne pas confondre** :
| Clé | Où vit-elle | Rôle |
|---|---|---|
| `STRIPE_SECRET_KEY` | backend **uniquement** | créer/piloter le PaymentIntent — ne quitte jamais le serveur |
| `VITE_STRIPE_PUBLIC_KEY` (publishable) | navigateur | initialiser le SDK Stripe |
| **`client_secret`** du PaymentIntent | back → front | jeton propre à **ce** paiement, autorise le navigateur à le confirmer sans exposer la clé secrète |

**Deux canaux séparés** (le point clé) :
- **Canal front (synchrone)** : `confirmPayment` ↔ Stripe en direct → réponse immédiate au navigateur (succeeded / 3D Secure / erreur) → feedback UI + redirection. **Ce résultat n'est PAS la source de vérité** (l'onglet peut se fermer, un client peut le falsifier).
- **Canal webhook (asynchrone, serveur-à-serveur)** : Stripe notifie **notre back** (étape 5 ci-dessous). **C'est LUI qui fait foi** et finalise la commande. Entre les deux, la base reste en `PROCESSING`.

> Sens à énoncer clairement : l'utilisateur soumet sa carte → **Stripe encaisse** → **Stripe nous notifie** via le webhook. Stripe n'est pas « prévenu qu'on a payé » : il *est* celui qui encaisse.

### Étape 5 — Webhook (`POST /payment/webhook`) — la source de vérité

```ts
// payment/payment.service.ts:22-70 (extrait)
async handleWebhook(signature, rawBody) {
  // Vérifie que l'appel vient BIEN de Stripe (signature + corps brut)
  const event = this.stripe.webhooks.constructEvent(rawBody, signature, process.env.STRIPE_WEBHOOK_SECRET);
  switch (event.type) {
    case 'payment_intent.succeeded':      await this.handlePaymentSuccess(event.data.object); break;
    case 'payment_intent.payment_failed': await this.handlePaymentFailed(event.data.object);  break;
    case 'payment_intent.canceled':       await this.handlePaymentCanceled(event.data.object); break;
    default: this.logger.log(`Événement ignoré: ${event.type}`);
  }
  return { received: true };
}
```

**Comment `constructEvent` sécurise l'endpoint** : `/payment/webhook` est une URL **publique** (Stripe doit l'atteindre) → sans protection, n'importe qui pourrait POSTer un faux `payment_intent.succeeded`. Stripe et le back partagent un secret (`STRIPE_WEBHOOK_SECRET`, `whsec_...`). À chaque envoi, Stripe calcule un **HMAC-SHA256** de `(timestamp . corps_brut)` avec ce secret et le met dans l'en-tête `Stripe-Signature`. `constructEvent` **recalcule** le HMAC de notre côté et compare : match → requête authentique et non altérée ; sinon → `throw` → 400. Deux détails : le **corps brut** (`rawBody: true` dans `main.ts`) est obligatoire car la signature porte sur les octets exacts (re-sérialiser casserait le HMAC), et le **timestamp** protège du rejeu. → même principe que la note « Signature HMAC » (secret partagé, symétrique).

```ts
// payment/payment.service.ts:78-129 (extrait) — succès du paiement
private async handlePaymentSuccess(paymentIntent) {
  const orderId = parseInt(paymentIntent.metadata.orderId, 10);
  const order = await this.prisma.order.findUnique({ where: { id: orderId } });

  if (order.paymentIntentId !== paymentIntent.id) return;       // SÉCURITÉ : correspondance PI ↔ commande
  if (order.paymentStatus === PaymentStatus.PAID) return;       // IDEMPOTENCE : déjà traité → on ne refait rien

  await this.prisma.$transaction(async (tx) => {
    const invoiceNumber = await generateInvoiceNumber(tx);      // ex: FA-2025-000001
    await tx.order.update({ where: { id: orderId }, data: {
      paymentStatus: PaymentStatus.PAID,
      orderStatus: OrderStatus.CONFIRMED,
      invoiceNumber, invoiceDate: new Date(),
    }});
  });
  // puis envoi de l'email de confirmation (non bloquant)
}
```

En cas d'échec, `handlePaymentFailed` restaure le stock et repasse la commande en `FAILED`
(réutilisable pour réessayer). En annulation, `handlePaymentCanceled` passe en `CANCELLED`.

> « Pourquoi un webhook et pas juste la réponse de Stripe au front ? » → parce que le front peut
> fermer l'onglet ou perdre le réseau. Le webhook serveur-à-serveur est fiable : c'est LUI qui
> valide définitivement le paiement. Mais un webhook peut se perdre → d'où le cron (§6).

### Étape 6 — Suivi & admin

Machine à états stricte des commandes :

```ts
// orders/orders.constants.ts (fichier complet, partie transitions)
export const ORDER_STATUS_TRANSITIONS: Record<OrderStatus, OrderStatus[]> = {
  PENDING:   [OrderStatus.CANCELLED],
  CONFIRMED: [OrderStatus.PLANTED, OrderStatus.CANCELLED],
  PLANTED:   [],   // terminal
  CANCELLED: [],   // terminal
};
```

Toute transition admin est validée contre cette table, et une annulation de commande **payée**
déclenche un **remboursement Stripe** + restaure le stock :

```ts
// orders/orders.service.ts:199-251 (extrait)
const allowedTransitions = ORDER_STATUS_TRANSITIONS[order.orderStatus] || [];
if (!allowedTransitions.includes(newStatus)) {
  throw new BadRequestException(`Transition non autorisée : ${...} → ${...}`);  // ← garde-fou métier
}
if (newStatus === OrderStatus.CANCELLED) {
  if (order.paymentStatus === PaymentStatus.PAID && order.paymentIntentId) {
    await this.stripe.refunds.create({ payment_intent: order.paymentIntentId });  // ← remboursement
  }
  return this.prisma.$transaction(async (tx) => {
    if (order.stockReservedAt) { /* restaure le stock */ }
    return tx.order.update({ where: { id }, data: {
      orderStatus: newStatus, stockReservedAt: null,
      paymentStatus: order.paymentStatus === PaymentStatus.PAID ? PaymentStatus.REFUNDED : order.paymentStatus,
    }});
  });
}
```

La facture PDF (`GET /orders/:id/invoice`) n'est servie que si la commande est **PAID** et a un
`invoiceNumber` (`orders.service.ts:307-317`), générée avec `@react-pdf/renderer`.

**Le numéro de facture** : il n'y a **pas de table `invoice`** — `invoiceNumber` (`@unique`) et `invoiceDate` sont deux **colonnes nullables du modèle `Order`**, remplies seulement au passage en `PAID`. `generateInvoiceNumber(tx)` (`common/utils/invoice.utils.ts`) construit le préfixe `FA-<année>-`, cherche la **dernière** facture de l'année (`orderBy desc`), parse le numéro, `+1`, pad à 6 chiffres → `FA-2026-000001`. Le préfixe contenant l'année, la numérotation **repart à `000001` chaque 1er janvier**. Appel **dans la transaction** pour l'atomicité ; garde-fou = la contrainte `@unique` (le motif « dernier + 1 » a un léger risque de concurrence — une **séquence Postgres** serait plus robuste : évolution à assumer).

---

## 6. Le cron de réconciliation (`orders/orders-cleanup.service.ts`)

**L'idée en une image.** Le webhook (§5) est la voie **normale** mais **best-effort** : un webhook peut
se perdre (réseau, serveur momentanément down). Le cron est le **filet de sécurité** qui passe
régulièrement « faire le ménage » sur les commandes restées coincées, et qui **réconcilie** l'état de
ma base avec l'état **réel** chez Stripe. Il ne remplace pas le webhook, il le **rattrape**.

**Le cycle de vie d'une commande** (pour situer où le cron intervient) :
```
   init            confirm             webhook succeeded            (admin)
PENDING/PENDING ─► PROCESSING/PENDING ─► PAID/CONFIRMED ─► PLANTED
                        │                       ▲
                        │ webhook perdu ?        │  le cron rejoue ce que
                        └───────────────────────┘  le webhook aurait fait
```
> Notation : `paymentStatus/orderStatus`. Le cron ne s'intéresse **qu'aux commandes restées en
> `orderStatus=PENDING`** depuis plus de 30 min - une commande déjà `CONFIRMED`/`CANCELLED` est
> réglée, il n'y touche pas.

**Le problème concret qu'il résout.** Que se passe-t-il si le client paie (argent débité) mais que le
**webhook n'arrive jamais** ? La commande reste bloquée en `PROCESSING`. Cas **critique** : il ne faut
**surtout pas** l'annuler sans vérifier, sinon on garde l'argent sans livrer.

**Le déclencheur** : une tâche planifiée **toutes les heures**.

```ts
// orders/orders-cleanup.service.ts:60-124 (extrait)
@Cron(CronExpression.EVERY_HOUR)
async cleanupAbandonedOrders() {
  const abandonedThreshold = new Date(Date.now() - ABANDONED_ORDER_THRESHOLD_MS);  // 30 min

  // Paquet 1 : vraies abandonnées → à annuler
  const abandonedOrders = await this.prisma.order.findMany({ where: {
    orderStatus: OrderStatus.PENDING,
    updatedAt: { lt: abandonedThreshold },
    paymentStatus: { in: [PaymentStatus.PENDING, PaymentStatus.FAILED] },
  }, include: { items: true } });

  // Paquet 2 : "en limbes" (payé côté front, webhook perdu) → à RÉCONCILIER avec Stripe
  const stuckProcessingOrders = await this.prisma.order.findMany({ where: {
    orderStatus: OrderStatus.PENDING,
    updatedAt: { lt: abandonedThreshold },
    paymentStatus: PaymentStatus.PROCESSING,
  }, include: { items: true } });

  await this.handleAbandonedOrders(abandonedOrders);       // annule + restaure le stock
  await this.reconcileProcessingOrders(stuckProcessingOrders);   // interroge Stripe
}
```

**Pourquoi DEUX paquets et pas un seul ?** Parce que le `paymentStatus` dit **où en était le client**,
et ça change tout :

| Paquet | État | Ce que ça signifie | Danger ? | Action |
|---|---|---|---|---|
| 1 | `PENDING` / `FAILED` | le client **n'a pas** payé (parti avant, ou carte refusée) | Aucun | on **annule direct** + restaure le stock |
| 2 | `PROCESSING` | le client **a cliqué Payer** - l'argent est **peut-être débité** | ⚠️ Élevé | on **NE décide RIEN seul** → on demande à Stripe |

**La règle d'or** (la phrase à dire au jury) : *« On n'annule jamais une commande en `PROCESSING` sans
interroger Stripe. »* Le paquet 1 est sûr à annuler ; le paquet 2 touche à de l'argent réel → seul
Stripe connaît la vérité, donc on va la lui demander.

**La réconciliation proprement dite** (paquet 2) : on interroge Stripe pour connaître l'état **réel**.

```ts
// orders/orders-cleanup.service.ts:271-346 (extrait)
private async reconcileProcessingOrders(orders) {
  for (const order of orders) {
    const paymentIntent = await this.stripe.paymentIntents.retrieve(order.paymentIntentId);

    if (paymentIntent.status === 'succeeded') {            // ← le webhook s'est perdu mais l'argent est là
      await this.prisma.$transaction(async (tx) => {       //   → on finalise comme l'aurait fait le webhook
        const invoiceNumber = await generateInvoiceNumber(tx);
        await tx.order.update({ where: { id: order.id }, data: {
          paymentStatus: PaymentStatus.PAID, orderStatus: OrderStatus.CONFIRMED,
          invoiceNumber, invoiceDate: new Date(),
        }});
      });
    } else if (['canceled', 'failed'].includes(paymentIntent.status)) {
      await this.handleAbandonedOrders([order]);           // ← annule + restaure le stock
    } else {
      // requires_action / processing (ex: 3D Secure en attente) → on NE TOUCHE À RIEN, prochain CRON
    }
  }
}
```

| État réel chez Stripe | Décision du cron |
|---|---|
| `succeeded` | **Finalise** : `PAID` + `CONFIRMED` + facture. Le client a payé, on honore. |
| `canceled` / `failed` | **Annule** et restaure le stock. |
| `requires_action` / `processing` | **Ne rien faire**, attendre le prochain passage. |

> Phrase clé : « Le webhook est la voie normale ; le cron est le **filet de sécurité**. Il réconcilie
> l'état de Stripe avec l'état de ma base pour ne jamais laisser une commande payée dans les limbes,
> et pour ne jamais annuler à tort une commande dont le paiement a réussi. »

**Les 4 points de robustesse à comprendre** (chacun répond à un « et si… ») :
- **Seuil 30 min ≥ timeout du checkout** → et si le cron passait pendant que le client paie encore ? Le
  seuil garantit qu'on ne nettoie jamais une commande « vivante ».
- **Idempotence** → et si le webhook ET le cron finalisaient la même commande ? Le cron ne cible que les
  commandes encore en `PENDING`/`PROCESSING` ; dès que l'une passe `PAID`/`CONFIRMED`, elle sort du
  périmètre. Le cron **rejoue exactement** ce que le webhook aurait fait (même passage `PAID` +
  `CONFIRMED` + `generateInvoiceNumber`) → peu importe qui gagne la course, le résultat est le même.
- **Stock restauré uniquement si `stockReservedAt ≠ null`** → et si on restaurait le stock d'une commande
  qui n'en avait jamais réservé (annulée en `PENDING`, avant le `confirm`) ? On créerait du stock
  fantôme. Le flag `stockReservedAt` évite cette double-restauration.
- **Appel Stripe HORS transaction DB** → et si Stripe timeout ? L'annulation en base ne doit pas être
  bloquée par un échec réseau côté Stripe (de toute façon un PaymentIntent non capturé expire seul en
  24 h). Idem, un **try/catch par commande** : l'échec sur une commande n'interrompt pas les autres.

> À l'oral : « Le webhook est la voie normale ; le cron est le filet de sécurité **idempotent**. Il ne
> peut ni finaliser deux fois, ni annuler à tort une commande payée, ni fabriquer du stock fantôme -
> parce qu'il rejoue les mêmes opérations que le webhook et s'appuie sur les statuts et le flag
> `stockReservedAt` comme garde-fous. »

---

## 7. Le frontend React

### 7.1 Stack

SPA **React 19 + Vite** qui consomme l'API. **TanStack Router** (routes typées), **TanStack Query**
(cache/requêtes serveur), **Zustand** (état local), Tailwind + shadcn/ui, react-hook-form + Zod,
Stripe Elements.

### 7.2 L'appel API et l'auth (le cookie fait tout)

```ts
// lib/api/client.ts (fichier complet)
export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL || "/api",
  headers: { "Content-Type": "application/json" },
  withCredentials: true,           // ← envoie AUTOMATIQUEMENT le cookie JWT à chaque requête
});

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error?.response?.status === 401) {          // ← 401 = session expirée/invalide
      useAuthStore.getState().logout();             //   → on vide le store
      if (window.location.pathname !== "/login") {
        window.location.href = "/login";            //   → et on redirige vers le login
      }
    }
    return Promise.reject(error);
  },
);
```

**Ce qu'il faut voir** : le front **n'attache aucun token** manuellement (`withCredentials` suffit,
le token est dans le cookie httpOnly). Pas de refresh token : un 401 = reconnexion.

### 7.3 L'état d'auth : le profil, pas le token

```ts
// features/auth/store/authStore.ts (extrait)
export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
      logout: () => set({ user: null }),
    }),
    { name: "greenroots-auth", partialize: (state) => ({ user: state.user }) },  // ← localStorage, user seulement
  ),
);
export const selectIsAuthenticated = (state) => Boolean(state.user);
```

**Ce qu'il faut voir** : le store ne contient **que le profil** (pour l'UI). Le JWT n'est jamais
en JavaScript — il vit dans le cookie. Les routes de garde (`_authenticated`, `_auth-guard`)
vérifient `selectIsAuthenticated` dans un `beforeLoad` et redirigent vers `/login?redirect=...`.

> **La sécurité réelle est côté back** (les guards). Le front ne protège que l'affichage / l'UX.

### 7.4 Le polling de confirmation (miroir du webhook)

Après le paiement, la page de confirmation **interroge en boucle** le statut, le temps que le
webhook finalise la commande :

```ts
// features/checkout/api/checkoutApi.ts (extrait du hook useOrderStatus)
refetchInterval: (query) => {
  const data = query.state.data;
  const updateCount = query.state.dataUpdateCount;

  if (data?.paymentStatus === "PAID")   return false;        // succès → stop
  if (data?.paymentStatus === "FAILED") return updateCount < 5 ? 2000 : false;  // grâce ~10s (webhook tardif)
  if (data?.paymentStatus === "PENDING") return false;       // pas finalisée → stop
  if (data?.paymentStatus === "PROCESSING" && updateCount < 30) return 2000;    // en cours → relance /2s (~1 min max)
  return false;
},
```

**Ce qu'il faut voir** : le front ne « devine » pas le résultat du paiement — il **redemande au
back l'état réel** jusqu'à ce que le webhook ait fait son travail. Cohérent avec « le webhook est
la source de vérité ».

---

## 8. CI/CD, tests et déploiement

Trois workflows **GitHub Actions**, calés sur la stratégie de branches (feature → develop → main) :

| Workflow | Déclencheur | Rôle |
|---|---|---|
| **pr-checks** | **push** sur les branches de travail (hors main/develop) | Feedback **rapide, sans BDD** : audit sécurité, lint, tests unitaires (**front Vitest + back Jest**), build |
| **develop-ci** | **push + PR d'intégration** sur `develop` | **Barrière qualité** : tests + couverture back, tests + build front, et validation **migrations + seed** sur une vraie BDD |
| **production-deploy** | push sur `main` | **Déploie seulement** (Coolify) + health-check. Pas de re-test : le code sur `main` a déjà été validé par develop-ci |

**Ce qu'il faut voir :**

- **Tests** : unitaires uniquement — front (**Vitest**) et back (**Jest**, où Prisma est **mocké**, donc pas besoin de BDD pour ces tests). Les tests **d'intégration et e2e ne sont pas encore là** : c'est une évolution assumée, pas un oubli.
- **Couverture** : un **plancher** `lines: 40` (`jest.coverageThreshold` dans `package.json`), appliqué par `test:cov` sur develop-ci. C'est un garde-fou **anti-régression** (la couverture back réelle est autour de 50 %), pas une cible.
- **Job « Migrations & Seed »** : il reproduit **exactement ce que la prod exécute au démarrage du conteneur**, à chaque déploiement :
  ```
  prisma migrate deploy  &&  (tsx seed.prod.ts || echo …)  &&  node main.js   # CMD Dockerfile.prod
  ```
  Le seed est **idempotent** : il fait un `user.count()` en tête et **sort si la base est déjà peuplée**. Le job le lance **deux fois** d'affilée pour prouver cette idempotence.
- **Déploiement** : sur `main`, le workflow déclenche un **webhook Coolify** (authentifié par token Bearer) puis fait un **health-check** de l'API (`/api/health`, avec retries) et du front. Hébergement : **VPS Hetzner + Coolify** (un PaaS auto-hébergé, façon Heroku maison), image **Docker multi-stage Alpine, utilisateur non-root**.
- **Filet de sécurité** : si une migration casse, le conteneur ne démarre pas → le health-check échoue → on le voit tout de suite.

> **Honnêteté à assumer** : le redéploiement Coolify n'est **pas** zero-downtime — il y a une coupure d'environ **30-45 s** (le conteneur s'arrête, rejoue migrations + seed, puis redémarre). C'est un axe d'amélioration identifié (rolling update / bascule sur health-check).

> Phrase clé : « **develop est ma barrière qualité** ; `main` ne fait que **déployer ce qui est déjà validé**, avec un health-check comme filet. J'ai volontairement retiré la double exécution des tests sur `main` : elle était redondante avec develop-ci. »

---

## 9. Questions probables du jury

- **« Comment sécurisez-vous l'API ? »** → JWT en cookie httpOnly + guards (Jwt/Roles), validation
  stricte (ValidationPipe whitelist), rate limiting global + `@Throttle` durci sur le login,
  anti-brute-force, blacklist de tokens, helmet, CORS restreint, tokens hachés, bcrypt.
- **« Paiement réussi mais webhook perdu ? »** → le cron interroge Stripe et finalise (§6).
  **LA réponse à préparer.**
- **« Pourquoi ne pas décrémenter le stock à la création de commande ? »** → pour ne pas bloquer du
  stock sur des paniers abandonnés ; on réserve seulement au `confirm`, juste avant le paiement.
- **« Pourquoi deux statuts (paiement/commande) ? »** → séparer l'état de l'argent de l'état
  logistique, et gérer proprement les cas limites.
- **« Éviter de traiter deux fois un webhook ? »** → idempotence (si déjà `PAID`, on ne refait rien)
  + vérification `paymentIntentId` ↔ commande.
- **« Pourquoi PostGIS ? »** → stockage géo des forêts (point, SRID 4326), extensible aux requêtes spatiales.
- **« Où est le panier ? »** → navigateur (localStorage/Zustand), pas en base ; devient commande au checkout.
- **« Rôle de Redis ? »** → cache (catégories, pays), lockout anti-brute-force, blacklist de tokens.
  Optionnel : si Redis tombe, l'app fonctionne (fail open).
- **« Comment déployez-vous ? »** → push sur `main` → GitHub Actions déclenche Coolify (webhook auth
  Bearer) + health-check API/front ; hébergement **VPS Hetzner + Coolify**. La qualité est validée
  **en amont sur `develop`** (tests + migrations + seed), `main` ne fait que déployer.
- **« Vos tests tournent-ils en CI ? »** → oui : unitaires **front (Vitest)** et **back (Jest)** sur
  chaque PR, **couverture back** avec un plancher anti-régression, et validation **migrations + seed
  idempotent** sur `develop`. Intégration/e2e = évolution assumée.

---

## 10. Mémo express (à relire juste avant l'oral)

- **NestJS** = modules (controller = routes + guards, service = logique, DTO = validation).
- **Auth** = JWT en cookie httpOnly ; Passport (Local à la connexion, JWT ensuite) ; bcrypt ;
  rôle relu en base à chaque requête.
- **Guards** = Throttler (global), JwtAuthGuard (connecté), RolesGuard (rôle). Ordre : Jwt **puis** Roles.
- **BDD** = PostgreSQL + Prisma, 11 modèles, argent en centimes, PostGIS pour les forêts,
  `OrderItem` = liaison Order↔Tree avec prix figé.
- **Deux statuts** : paiement (PENDING→PROCESSING→PAID/FAILED/REFUNDED) et commande
  (PENDING→CONFIRMED→PLANTED / CANCELLED, transitions contrôlées).
- **Tunnel** : cart → init (crée commande + PaymentIntent) → confirm (réserve stock) → Stripe →
  webhook (finalise, **source de vérité**) → suivi/facture.
- **Cron horaire** = filet de sécurité : annule les abandons, **réconcilie** avec Stripe les
  commandes payées dont le webhook s'est perdu.
- **Frontend** = React + TanStack Router/Query + Zustand ; cookie envoyé auto (`withCredentials`) ;
  polling de confirmation ; **sécurité réelle côté back**.
- **CI/CD** = 3 workflows GitHub Actions (feature → develop → main) : tests unitaires front+back,
  couverture back (plancher), validation migrations + seed idempotent sur `develop` ; `main`
  **déploie** via Coolify + health-check (pas de re-test). Hébergement VPS Hetzner + Coolify.
```
