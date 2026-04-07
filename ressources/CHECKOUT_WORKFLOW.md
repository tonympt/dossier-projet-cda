# Documentation Workflow Checkout & Paiements GreenRoots

## Table des matières

1. [Vue d'ensemble du tunnel de vente](#1-vue-densemble-du-tunnel-de-vente)
2. [Panier (Frontend - localStorage)](#2-panier-frontend---localstorage)
3. [Validation panier et vérification de stock](#3-validation-panier-et-vérification-de-stock)
4. [Checkout - Initialisation](#4-checkout---initialisation)
5. [Confirmation et réservation de stock](#5-confirmation-et-réservation-de-stock)
6. [Intégration Stripe](#6-intégration-stripe)
7. [Webhooks Stripe](#7-webhooks-stripe)
8. [Statuts de commande et paiement](#8-statuts-de-commande-et-paiement)
9. [Cron de nettoyage](#9-cron-de-nettoyage)
10. [Administration des commandes](#10-administration-des-commandes)
11. [Architecture et dépendances entre modules](#11-architecture-et-dépendances-entre-modules)
12. [Points de sécurité clés](#12-points-de-sécurité-clés)

---

## 1. Vue d'ensemble du tunnel de vente

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   PANIER    │───>│  CHECKOUT   │───>│  CONFIRM    │───>│  PAIEMENT   │───>│ CONFIRMATION│
│ localStorage│    │    /init    │    │   STOCK     │    │   STRIPE    │    │   WEBHOOK   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
      │                  │                  │                  │                  │
      v                  v                  v                  v                  v
   Zustand +         Création           Réservation       confirmPayment    payment_intent
   persist           Order +            atomique du       Stripe SDK        .succeeded
                     PaymentIntent      stock (TX)                          -> PAID/CONFIRMED
```

---

## 2. Panier (Frontend - localStorage)

### Fichiers concernés

| Fichier | Description |
|---------|-------------|
| `app/frontend/src/features/cart/store/cartStore.ts` | Store Zustand avec persist |
| `app/frontend/src/features/cart/types/cart.types.ts` | Types TypeScript |

### Fonctionnement

Le panier utilise **Zustand avec middleware `persist`**, ce qui stocke automatiquement l'état dans `localStorage` sous la clé `"cart-storage"`.

**Structure persistée** :

```typescript
{
  items: [{ productId: number, quantity: number }, ...],
  createdAt: number | null  // timestamp de création
}
```

### Actions disponibles

| Action | Description |
|--------|-------------|
| `addItem(productId, quantity)` | Ajoute ou cumule la quantité si l'item existe déjà |
| `removeItem(productId)` | Supprime l'item du panier |
| `updateQuantity(productId, quantity)` | Modifie directement la quantité |
| `clearCart()` | Vide le panier |
| `isExpired()` | Retourne `true` si le panier a plus de 24h |
| `checkAndClearIfExpired()` | Vérifie et vide le panier si expiré |
| `syncWithStock(stockData)` | Synchronise les quantités avec le stock serveur |

### TTL du panier (24 heures)

Le panier a une durée de vie de **24 heures**. Le composant `CartTimer.tsx` affiche un countdown :

- Alerte jaune si < 4 heures restantes
- Alerte rouge si < 1 heure restante
- Expiration = vidage automatique via `checkAndClearIfExpired()`

---

## 3. Validation panier et vérification de stock

### Fichiers concernés

| Fichier | Description |
|---------|-------------|
| `app/frontend/src/features/cart/components/Cart.tsx` | Composant page panier |
| `app/frontend/src/features/cart/hooks/useCartProducts.tsx` | Hook de récupération produits |
| `app/frontend/src/features/cart/api/cartApi.ts` | Appels API |
| `app/backend/src/cart/cart.controller.ts` | Controller backend |
| `app/backend/src/cart/cart.service.ts` | Service backend |

### Flux de vérification (Soft Check)

```
┌──────────────┐         POST /cart/batch-details         ┌──────────────┐
│   Frontend   │  ─────────────────────────────────────>  │   Backend    │
│  useCartProducts                                        │  CartService │
│              │  <─────────────────────────────────────  │              │
└──────────────┘    items[] avec stock actuel + notFoundIds└──────────────┘
       │
       v
  syncWithStock()  -> Si stock < quantité demandée,
                      ajuste silencieusement la quantité
                      et stocke le changement dans itemsChangeStock
```

### Endpoint `POST /cart/batch-details`

**Entrée** (aucune auth requise) :

```typescript
{ items: [{ treeId: number, quantity: number }] }
```

**Sortie** :

```typescript
{
  items: [{
    id, name, price, stock, lowStockThreshold,
    location: { name, country },
    category: { name }
  }],
  notFoundIds: number[]  // arbres supprimés
}
```

### Comportement au clic "Acheter"

1. `refetch()` pour avoir le stock le plus frais
2. Vérification des items avec `stock <= 0` → Toast d'erreur
3. Vérification des items dans `itemsChangeStock` → Dialog de confirmation
4. Si stock épuisé : bouton "Continuer" désactivé, l'utilisateur doit retirer manuellement
5. Si seulement réduction de quantité : confirmation possible puis redirection vers `/checkout`

---

## 4. Checkout - Initialisation

### Fichiers concernés

| Fichier | Description |
|---------|-------------|
| `app/frontend/src/features/checkout/Checkout.tsx` | Page checkout |
| `app/frontend/src/features/checkout/schemas/checkoutSchema.ts` | Schema Zod validation |
| `app/backend/src/checkout/checkout.controller.ts` | Controller backend |
| `app/backend/src/checkout/checkout.service.ts` | Service backend |

### Flux d'initialisation

```
┌──────────────────────────────────────────────────────────────────┐
│                         FRONTEND                                  │
│  1. Formulaire adresse (Zod validation)                          │
│  2. Clic "Valider l'achat"                                       │
│  3. POST /checkout/init                                          │
└──────────────────────────────────────────────────────────────────┘
                              │
                              v
┌──────────────────────────────────────────────────────────────────┐
│                    BACKEND (Transaction Prisma)                   │
│                                                                   │
│  1. Validation pays (isBillingAvailable = true)                  │
│  2. Recherche commande réutilisable*                             │
│  3. Création/MAJ Order + OrderItems + BillingAddress             │
│  4. Création/MAJ PaymentIntent Stripe                            │
│  5. Retourne { orderId, clientSecret }                           │
└──────────────────────────────────────────────────────────────────┘
```

**\*Commande réutilisable** : Une commande existante peut être réutilisée si :

- `orderStatus = PENDING`
- `paymentStatus = PENDING`
- `stockReservedAt = null`
- `updatedAt < 30 minutes`

Cela permet à l'utilisateur de modifier son panier et revenir au checkout sans créer une nouvelle commande à chaque fois.

### Endpoint `POST /checkout/init` (Auth requise)

**Entrée** :

```typescript
{
  items: [{ treeId: number, quantity: number }],
  contactEmail: string,
  billingAddress: {
    firstName, lastName, street, city, zipCode, countryCode
  }
}
```

**Sortie** :

```typescript
{ orderId: number, clientSecret: string }
```

### Point important : Stock NON décrémenté ici

> Le stock n'est PAS décrémenté à cette étape pour éviter le "phantom stock" des paniers abandonnés.

---

## 5. Confirmation et réservation de stock

### Fichiers concernés

| Fichier | Description |
|---------|-------------|
| `app/frontend/src/features/checkout/StripePaymentSection.tsx` | Composant paiement Stripe |
| `app/backend/src/checkout/checkout.service.ts` | Méthode `confirmCheckout` |

### Flux de confirmation (Hard Check - Atomique)

```
┌──────────────────────────────────────────────────────────────────┐
│  POST /checkout/confirm { orderId }  (AVANT confirmPayment)      │
└──────────────────────────────────────────────────────────────────┘
                              │
                              v
┌──────────────────────────────────────────────────────────────────┐
│                 TRANSACTION PRISMA ATOMIQUE                       │
│                                                                   │
│  1. Charge Order + OrderItems + TreeStock                        │
│  2. Vérifie : order existe, appartient à l'user, PENDING         │
│  3. Guard idempotence : si stockReservedAt déjà set -> OK        │
│  4. Pour chaque item :                                           │
│     - Si stock = 0 -> issue "out-of-stock"                       │
│     - Si stock < quantity -> issue "insufficient"                │
│  5. Si issues[] non vide -> return { success: false, issues }    │
│  6. Si OK :                                                      │
│     - Décremente TreeStock.quantity pour chaque item             │
│     - Set stockReservedAt = now()                                │
│     - Set paymentStatus = PROCESSING                             │
│     - return { success: true }                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Gestion des erreurs de stock

Si `success: false`, le frontend affiche `CheckoutStockDialog` avec les problèmes :

```typescript
interface StockIssue {
  treeId: number;
  treeName: string;
  type: "out-of-stock" | "insufficient";
  requested?: number;
  available?: number;
}
```

L'utilisateur doit retourner au panier pour ajuster. **Stripe n'est PAS appelé si le stock est insuffisant.**

---

## 6. Intégration Stripe

### Fichiers concernés

| Fichier | Description |
|---------|-------------|
| `app/backend/src/stripe/stripe.module.ts` | Module Stripe global |
| `app/frontend/src/lib/stripe.ts` | Initialisation Stripe frontend |
| `app/backend/src/checkout/checkout.service.ts` | Création PaymentIntent |

### Architecture Stripe Backend

Le `StripeModule` est **@Global()** et fournit une instance `Stripe` initialisée avec `STRIPE_SECRET_KEY` (API version `2025-11-17.clover`).

### Création du PaymentIntent

```typescript
// checkout.service.ts - handleStripePaymentIntent()
const paymentIntent = await this.stripe.paymentIntents.create({
  amount,                              // en centimes (integer)
  currency: 'eur',
  metadata: { orderId: orderId.toString() },  // CRITIQUE : lien Order <-> PI
  payment_method_types: ['card'],
});
```

### Frontend : Confirmation du paiement

```typescript
// StripePaymentSection.tsx
const { error, paymentIntent } = await stripe.confirmPayment({
  elements,
  confirmParams: {
    return_url: `${window.location.origin}/order-confirmation?orderId=${orderId}`,
  },
  redirect: "if_required",  // 3DS si nécessaire
});
```

### Flux complet côté Frontend

1. Formulaire rempli
2. `POST /checkout/init` → `{ orderId, clientSecret }`
3. Mount `<Elements>` avec clientSecret
4. User clique "Payer"
5. `POST /checkout/confirm` → vérifie stock, réserve
6. Si success: `stripe.confirmPayment()`
7. Si paiement OK : `clearCart()`, navigate → `/order-confirmation`
8. Si redirect (3DS) : Stripe gère, retour sur `return_url`

---

## 7. Webhooks Stripe

### Fichiers concernés

| Fichier | Description |
|---------|-------------|
| `app/backend/src/payment/payment.controller.ts` | Controller webhook |
| `app/backend/src/payment/payment.service.ts` | Service de traitement |

### Endpoint `POST /payment/webhook`

- Décoré avec `@SkipThrottle()` pour ne pas être rate-limited
- Reçoit le raw body + header `stripe-signature`
- Vérifie la signature avec `stripe.webhooks.constructEvent()`

### Events traités

| Event Stripe | Handler | Action |
|--------------|---------|--------|
| `payment_intent.succeeded` | `handlePaymentSuccess` | `paymentStatus = PAID`, `orderStatus = CONFIRMED` |
| `payment_intent.payment_failed` | `handlePaymentFailed` | `paymentStatus = FAILED` (stock NON restauré) |
| `payment_intent.canceled` | `handlePaymentCanceled` | `orderStatus = CANCELLED` si pas déjà annulé |

### Sécurité des webhooks

**Vérification critique** dans chaque handler :

```typescript
// Extrait orderId des metadata
const orderId = paymentIntent.metadata.orderId;
// Charge la commande
const order = await this.prisma.order.findUnique({ where: { id: orderId } });
// VÉRIFIE que le paymentIntentId correspond
if (order.paymentIntentId !== paymentIntent.id) {
  throw new BadRequestException('PaymentIntent mismatch');
}
```

Cela empêche un attaquant de rejouer un PaymentIntent succeeded sur une autre commande.

### Page de confirmation (Polling)

**Fichier** : `app/frontend/src/features/checkout/OrderConfirmation.tsx`

Polling `GET /orders/:id/status` toutes les 2 secondes :

| Status | Comportement |
|--------|--------------|
| `PAID` | Stop polling, affiche succès |
| `PENDING` | Stop polling, propose retour au checkout |
| `PROCESSING` | Continue polling (max 30 fois ≈ 1 min), puis message "vérifiez email" |
| `FAILED` | Continue 5 fois de plus (grace period), puis affiche erreur + lien retry |

> **Note importante** : "Le webhook Stripe est la seule source de vérité. On ne se fie jamais à ce que le front 'pense' savoir."

---

## 8. Statuts de commande et paiement

### Fichiers concernés

| Fichier | Description |
|---------|-------------|
| `app/backend/prisma/schema.prisma` | Définition des enums |
| `app/backend/src/orders/orders.constants.ts` | Constantes de transitions |

### Enums définis

```prisma
enum PaymentStatus {
  PENDING      // Commande créée, en attente de paiement
  PROCESSING   // Stock réservé, paiement en cours
  PAID         // Paiement confirmé (webhook)
  FAILED       // Paiement échoué
  REFUNDED     // Remboursé (annulation admin)
}

enum OrderStatus {
  PENDING      // Commande créée
  CONFIRMED    // Paiement reçu
  PLANTED      // Arbre planté (terminal)
  CANCELLED    // Annulée (terminal)
}
```

### Diagramme des transitions

```
                    ┌─────────────────────────────────────────────┐
                    │              PaymentStatus                   │
                    │                                              │
                    │  PENDING ──> PROCESSING ──> PAID            │
                    │                    │                         │
                    │                    ├──> FAILED               │
                    │                    │                         │
                    │                    └──> REFUNDED (via cancel)│
                    └─────────────────────────────────────────────┘

                    ┌─────────────────────────────────────────────┐
                    │               OrderStatus                    │
                    │                                              │
                    │  PENDING ──────────> CONFIRMED ──> PLANTED  │
                    │     │                    │           (fin)   │
                    │     │                    │                   │
                    │     └──> CANCELLED <────┘                   │
                    │              (fin)                           │
                    └─────────────────────────────────────────────┘
```

### Transitions autorisées (Admin)

```typescript
// orders.constants.ts
ORDER_STATUS_TRANSITIONS = {
  PENDING:   ['CANCELLED'],
  CONFIRMED: ['PLANTED', 'CANCELLED'],
  PLANTED:   [],           // Terminal
  CANCELLED: [],           // Terminal
};
```

**Note** : La transition `PENDING → CONFIRMED` est faite automatiquement par le webhook, pas par l'admin.

---

## 9. Cron de nettoyage

### Fichier concerné

- `app/backend/src/orders/orders-cleanup.service.ts`

### Configuration

- **Fréquence** : Toutes les heures (`@Cron(CronExpression.EVERY_HOUR)`)
- **Seuil d'abandon** : 30 minutes (`ABANDONED_ORDER_THRESHOLD_MS`)

### Catégorie 1 : Commandes abandonnées

**Critères** :

- `orderStatus = PENDING`
- `paymentStatus IN (PENDING, FAILED)`
- `updatedAt < 30 minutes`

**Actions** (dans une transaction) :

1. Si `stockReservedAt` est set → Restaure le stock (incrémente `TreeStock.quantity`)
2. Set `orderStatus = CANCELLED`, `stockReservedAt = null`
3. Hors transaction : Annule le PaymentIntent Stripe (best effort)

### Catégorie 2 : Commandes bloquées en PROCESSING

**Critères** :

- `orderStatus = PENDING`
- `paymentStatus = PROCESSING`
- `updatedAt < 30 minutes`

**Scénario** : Le paiement a réussi mais le webhook n'est jamais arrivé (crash serveur, problème réseau).

**Actions** :

```typescript
// Récupère le PaymentIntent directement chez Stripe
const paymentIntent = await stripe.paymentIntents.retrieve(order.paymentIntentId);

switch (paymentIntent.status) {
  case 'succeeded':
    // RÉCONCILIATION : Le client a payé, on honore la commande
    order.paymentStatus = 'PAID';
    order.orderStatus = 'CONFIRMED';
    break;

  case 'canceled':
  case 'failed':
    // Traite comme abandonné
    restoreStock();
    cancelOrder();
    break;

  default:
    // Autre status (requires_action, processing) -> Attendre prochain cron
    break;
}
```

---

## 10. Administration des commandes

### Fichiers concernés

| Fichier | Description |
|---------|-------------|
| `app/backend/src/orders/orders.controller.ts` | Controller backend |
| `app/backend/src/orders/orders.service.ts` | Service backend |
| `app/frontend/src/features/admin/orders/AdminOrdersPage.tsx` | Page liste admin |
| `app/frontend/src/features/admin/orders/AdminOrderDetailPage.tsx` | Page détail admin |
| `app/frontend/src/features/orders/api/ordersApi.ts` | API frontend |

### Endpoints Admin (Auth + Roles ADMIN/SUPERADMIN)

| Méthode | Route | Description |
|---------|-------|-------------|
| GET | `/orders` | Liste paginée de toutes les commandes |
| GET | `/orders/:id` | Détail d'une commande (avec infos user) |
| GET | `/orders/:id/allowed-transitions` | Statuts possibles pour cette commande |
| PATCH | `/orders/:id` | Modifier le statut |

### Logique de modification de statut

```typescript
// orders.service.ts - updateOrderStatus()

// 1. Valide la transition
const allowedTransitions = ORDER_STATUS_TRANSITIONS[order.orderStatus];
if (!allowedTransitions.includes(newStatus)) {
  throw new BadRequestException('Transition non autorisée');
}

// 2. Si CANCELLED et paiement était PAID
if (newStatus === 'CANCELLED' && order.paymentStatus === 'PAID') {
  // Rembourse via Stripe
  await this.stripe.refunds.create({ payment_intent: order.paymentIntentId });
  // Dans la transaction :
  // - Restaure le stock
  // - Set paymentStatus = REFUNDED
}

// 3. Update final
order.orderStatus = newStatus;
order.stockReservedAt = null;  // si annulation
```

### Interface Admin Frontend

1. **Liste des commandes** : Tableau avec recherche par ID, email ou nom
2. **Détail** : Infos complètes + Select des transitions autorisées
3. **Confirmation** : AlertDialog avant changement de statut
4. **Refetch** : Invalidation des queries React Query après modification

---

## 11. Architecture et dépendances entre modules

```
AppModule
├── StripeModule (@Global)
│   └── Fournit instance Stripe partout
│
├── PrismaModule (@Global)
│   └── Fournit PrismaService partout
│
├── ScheduleModule.forRoot()
│   └── Active les décorateurs @Cron
│
├── CartModule
│   ├── imports: PrismaModule
│   └── POST /cart/batch-details (public)
│
├── CheckoutModule
│   ├── imports: PrismaModule, StripeModule
│   ├── POST /checkout/init (auth)
│   └── POST /checkout/confirm (auth)
│
├── PaymentModule
│   ├── imports: PrismaModule
│   ├── Utilise Stripe pour webhook verification
│   └── POST /payment/webhook (public, skip throttle)
│
├── OrdersModule
│   ├── imports: PrismaModule
│   ├── providers: OrdersService, OrdersCleanupService
│   ├── Utilise Stripe pour refunds et réconciliation
│   ├── GET/PATCH /orders (admin)
│   └── Cron cleanup toutes les heures
│
└── CountriesModule
    ├── GET /countries/billing
    └── GET /countries/planting
```

### Gestion des prix

**Fichier** : `app/backend/src/common/decorators/price.decorator.ts`

- **Stockage DB** : Entiers en centimes (évite les problèmes de floating point)
- **@PriceToDecimal()** : Divise par 100 à la sérialisation (response)
- **@PriceToCents()** : Multiplie par 100 à la désérialisation (request)
- **Stripe** : Attend des montants en centimes → cohérent avec la DB

**Intégrité des prix** : Le frontend envoie uniquement `treeId + quantity`. Le backend relit `tree.price` depuis la DB à la création de l'OrderItem.

---

## 12. Points de sécurité clés

### 1. Intégrité des prix

Le client ne peut pas manipuler les prix. Le backend lit toujours le prix depuis la base de données lors de la création des OrderItems.

### 2. Liaison PaymentIntent <-> Order

Chaque webhook vérifie que `order.paymentIntentId === paymentIntent.id`. Empêche le replay d'un PaymentIntent sur une autre commande.

### 3. Idempotence de la réservation de stock

`confirmCheckout()` retourne `{ success: true }` immédiatement si `stockReservedAt` est déjà défini → pas de double décrémentation en cas de double-clic.

### 4. Webhook = source de vérité

La page de confirmation poll la DB, pas la réponse Stripe côté client. Le webhook est le seul à mettre à jour les statuts.

### 5. Throttle désactivé pour webhook

`@SkipThrottle()` sur l'endpoint webhook pour que les retries Stripe passent.

### 6. Auth sur checkout

`/checkout/init` et `/checkout/confirm` requièrent un JWT valide. Seul `/cart/batch-details` est public (info de stock publique).

### 7. Réconciliation automatique

Le cron vérifie directement chez Stripe le statut des commandes bloquées en `PROCESSING` et les confirme si le paiement a réussi → pas de perte de vente en cas de webhook manqué.

---

## Résumé du flux complet

```
1.  User ajoute des arbres au panier (localStorage, Zustand persist)
2.  Panier TTL 24h avec timer visuel
3.  Page panier : POST /cart/batch-details -> sync stock
4.  Clic "Acheter" -> refetch stock, validation, redirection /checkout
5.  Formulaire adresse + POST /checkout/init -> Order créée + PaymentIntent
6.  Mount Stripe Elements avec clientSecret
7.  Clic "Payer" -> POST /checkout/confirm (réserve stock atomiquement)
8.  Si stock OK -> stripe.confirmPayment()
9.  Si 3DS -> redirect Stripe, retour sur return_url
10. Webhook payment_intent.succeeded -> PAID + CONFIRMED
11. Page confirmation poll le status jusqu'à PAID
12. Admin peut : CONFIRMED -> PLANTED ou CANCELLED (avec refund auto)
13. Cron horaire nettoie les commandes abandonnées et réconcilie les PROCESSING
```
