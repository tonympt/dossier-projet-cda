# Extraits de code — Dossier CDA GreenRoots

> Ce fichier centralise tous les extraits de code validés pour le dossier, organisés par section.
> Le code est **réel** (copié du projet), seule la troncature `// ...` est utilisée.

---

## §7.1 — Interfaces utilisateur (CP2)

### 1. cartStore — Zustand persist + syncWithStock

**Fichier** : `app/frontend/src/features/cart/store/cartStore.ts`
**CP démontrée** : CP2 — Développer des interfaces utilisateur
**Destination** : Corps §7.1

**Annotation** : Store Zustand avec middleware `persist` (localStorage, TTL 24h) et synchronisation du stock côté serveur. `syncWithStock` réconcilie le panier local avec le stock réel avant le checkout. Le `partialize` contrôle ce qui est sérialisé : seuls `items` et `createdAt` sont persistés, `itemsChangeStock` (données éphémères de notification) en est exclu.

```typescript
// app/frontend/src/features/cart/store/cartStore.ts
const CART_EXPIRATION_MS = 60 * 60 * 24 * 1000; // 24h

export const useCartStore = create<CartState & CartActions>()(
  persist(
    (set, get) => ({
      items: [],
      createdAt: null,
      itemsChangeStock: [],

      addItem: (productId, quantity) => {
        const state = get();
        if (state.isExpired()) {
          state.clearCart();
        }
        // Incrémente la quantité si le produit existe déjà, sinon l'ajoute
        // ...
      },

      syncWithStock: (stockData) => {
        // Pour chaque article du panier :
        //   1. Chercher le stock serveur correspondant
        //   2. Si stock suffisant → garder tel quel
        //   3. Si stock insuffisant → ajuster la quantité + mémoriser le changement
        //      dans itemsChangeStock (pour notifier l'utilisateur via un dialog)
        // ...
      },

      // removeItem, updateQuantity, clearCart, getTotalQuantity, isExpired...
    }),
    {
      name: "cart-storage",
      partialize: (state) => ({
        items: state.items,
        createdAt: state.createdAt,
      }),
    },
  ),
);
```

---

### 2. Checkout — PaymentState + initialisation paiement + montage Stripe

**Fichier** : `app/frontend/src/features/checkout/Checkout.tsx` + `types/checkout.type.ts`
**CP démontrée** : CP2 — Développer des interfaces utilisateur
**Destination** : Corps §7.1

**Annotation** : State machine implicite via `PaymentState` : IDLE → INITIALIZING → READY (clientSecret reçu) ou ERROR. Le formulaire se verrouille dès réception du `clientSecret`. Stripe Elements n'est monté qu'après obtention du secret — avant, le composant `StripePaymentSection` n'existe pas dans le DOM.

```typescript
// app/frontend/src/features/checkout/types/checkout.type.ts
export interface PaymentState {
  isInitializing: boolean;
  clientSecret: string | null;
  orderId: number | null;
  error: string | null;
}
```

```tsx
// app/frontend/src/features/checkout/Checkout.tsx
const Checkout = () => {
  const items = useCartStore((state) => state.items);
  const { cartProducts, isLoading } = useCartProducts();

  const [paymentState, setPaymentState] = useState<PaymentState>({
    isInitializing: false,
    clientSecret: null,
    orderId: null,
    error: null,
  });

  // Le formulaire est verrouillé une fois le paiement initialisé
  const isFormLocked = paymentState.clientSecret !== null;

  const handleInitializePayment = async () => {
    const isValid = await form.trigger();
    if (!isValid) {
      toast.error("Formulaire incomplet", {
        description: "Veuillez corriger les erreurs avant de continuer.",
      });
      return;
    }

    const formData = form.getValues();
    setPaymentState((prev) => ({ ...prev, isInitializing: true, error: null }));

    try {
      const payload = {
        items: items.map((item) => ({
          treeId: item.productId,
          quantity: item.quantity,
        })),
        contactEmail: formData.contactEmail,
        billingAddress: { /* firstName, lastName, street, postalCode, city... */ },
      };

      const { orderId, clientSecret } = await checkoutApi.initCheckout(payload);
      setPaymentState({ isInitializing: false, clientSecret, orderId, error: null });
    } catch (error) {
      const message = getErrorMessage(error);
      setPaymentState((prev) => ({ ...prev, isInitializing: false, error: message }));
      toast.error(message);
    }
  };

  // ...

  return (
    <div className="flex flex-col md:flex-row gap-6 mt-6 items-start">
      <div className="w-full flex-1 md:w-1/2">
        <CheckoutForm form={form} disabled={isFormLocked} />
      </div>

      <div className="w-full md:w-1/2 lg:sticky lg:top-30">
        {/* Stripe Elements monté UNIQUEMENT après obtention du clientSecret */}
        {paymentState.clientSecret && stripeOptions ? (
          <Elements stripe={getStripePromise()} options={stripeOptions}>
            <CheckoutSummary paymentState={paymentState} onInitializePayment={handleInitializePayment}>
              <StripePaymentSection orderId={paymentState.orderId!} totalTTC={totalTTC} onStockIssues={handleStockIssues} />
            </CheckoutSummary>
          </Elements>
        ) : (
          <CheckoutSummary paymentState={paymentState} onInitializePayment={handleInitializePayment} />
        )}
      </div>
    </div>
  );
};
```

---

### 3. StripePaymentSection — Double confirm (stock puis paiement)

**Fichier** : `app/frontend/src/features/checkout/StripePaymentSection.tsx`
**CP démontrée** : CP2 — Développer des interfaces utilisateur
**Destination** : Corps §7.1

**Annotation** : Pattern de double confirmation : étape 1 = réservation stock côté backend (`confirmCheckout`), étape 2 = paiement Stripe (`stripe.confirmPayment`). Si le stock est insuffisant, le paiement n'est jamais lancé. Le panier est vidé uniquement après succès Stripe.

```tsx
const handleConfirmPayment = async () => {
  if (!stripe || !elements) {
    toast.error("Stripe n'est pas prêt");
    return;
  }

  setIsConfirming(true);

  try {
    // 1. ÉTAPE 1 : Confirmer côté backend (réserve le stock)
    const confirmResult = await checkoutApi.confirmCheckout({ orderId });

    if (!confirmResult.success) {
      if (confirmResult.issues && confirmResult.issues.length > 0) {
        onStockIssues(confirmResult.issues);
      } else {
        toast.error("Stock insuffisant", {
          description: "Certains articles ne sont plus disponibles.",
        });
      }
      setIsConfirming(false);
      return;
    }

    // 2. ÉTAPE 2 : Stock réservé → lancer le paiement Stripe
    const { error, paymentIntent } = await stripe.confirmPayment({
      elements,
      confirmParams: {
        return_url: `${window.location.origin}/order-confirmation?orderId=${orderId}`,
      },
      redirect: "if_required",
    });

    if (error) {
      toast.error("Paiement refusé", {
        description: error.message,
      });
      setIsConfirming(false);
      return;
    }

    if (paymentIntent?.status === "succeeded") {
      clearCart();
      toast.success("Paiement réussi !");
      navigate({
        to: "/order-confirmation",
        search: { orderId },
      });
    }
  } catch (error) {
    toast.error(getErrorMessage(error));
    setIsConfirming(false);
  }
};
```

---

### 4. OrderConfirmation — Polling avec délai de grâce

**Fichier** : `app/frontend/src/features/checkout/OrderConfirmation.tsx` + `api/checkoutApi.ts`
**CP démontrée** : CP2 — Développer des interfaces utilisateur
**Destination** : Corps §7.1

**Annotation** : Polling intelligent via TanStack Query `refetchInterval`. Le composant gère 6 états d'affichage dont un timeout (30 polling = ~1min) et un délai de grâce pour FAILED (5 polling = ~10s au cas où le webhook PAID arrive juste après).

#### Extrait — Hook useOrderStatus (logique polling)

```typescript
// app/frontend/src/features/checkout/api/checkoutApi.ts
export const useOrderStatus = (orderId: number | undefined) => {
  const queryClient = useQueryClient();
  const queryKey = ["order", "status", orderId];

  const query = useQuery({
    queryKey,
    queryFn: () => checkoutApi.getOrderStatus(orderId!),
    enabled: !!orderId,
    refetchInterval: (query) => {
      const data = query.state.data;
      const updateCount = query.state.dataUpdateCount;

      // Arrêter le polling si PAID
      if (data?.paymentStatus === "PAID") {
        return false;
      }

      // FAILED : délai de grâce de 5 polling (~10 secondes)
      if (data?.paymentStatus === "FAILED") {
        if (updateCount < 5) {
          return 2000;
        }
        return false;
      }

      // Arrêter si PENDING (commande pas encore confirmée)
      if (data?.paymentStatus === "PENDING") {
        return false;
      }

      // PROCESSING : continuer le polling (max 30 essais = ~1 minute)
      if (data?.paymentStatus === "PROCESSING" && updateCount < 30) {
        return 2000;
      }

      return false;
    },
  });

  const queryState = queryClient.getQueryState(queryKey);

  return {
    ...query,
    updateCount: queryState?.dataUpdateCount ?? 0,
  };
};
```

#### Extrait — Composant OrderConfirmation (gestion des 6 états)

```tsx
// app/frontend/src/features/checkout/OrderConfirmation.tsx
const OrderConfirmation = () => {
  const { orderId } = useSearch({ strict: false }) as { orderId?: string };
  const clearCart = useCartStore((state) => state.clearCart);

  const {
    data: orderStatus,
    isLoading,
    updateCount,
  } = useOrderStatus(Number(orderId));

  // Timeout après 30 tentatives de polling (environ 1 minute)
  const isProcessingTimeout =
    orderStatus?.paymentStatus === "PROCESSING" && updateCount >= 30;

  // FAILED mais encore en délai de grâce (5 polling = ~10 secondes)
  const isFailedButWaiting =
    orderStatus?.paymentStatus === "FAILED" && updateCount < 5;

  // Vider le panier quand le paiement est confirmé
  useEffect(() => {
    if (orderStatus?.paymentStatus === "PAID") {
      clearCart();
    }
  }, [orderStatus?.paymentStatus, clearCart]);

  // ...

  const renderContent = () => {
    // Cas 1 : PENDING → Commande non finalisée
    if (orderStatus?.paymentStatus === "PENDING") { /* ... */ }

    // Cas 2 : PROCESSING avec timeout → Validation bancaire longue
    if (isProcessingTimeout) { /* ... */ }

    // Cas 3 : PROCESSING (sans timeout) → Polling en cours
    if (orderStatus?.paymentStatus === "PROCESSING") { /* ... */ }

    // Cas 4 : PAID → Succès
    if (orderStatus?.paymentStatus === "PAID") { /* ... */ }

    // Cas 5 : FAILED → Échec (après le délai de grâce)
    if (orderStatus?.paymentStatus === "FAILED" && !isFailedButWaiting) { /* ... */ }

    // Cas 6 : FAILED mais en délai de grâce → On attend le webhook PAID
    if (isFailedButWaiting) { /* ... */ }
  };
  // ...
};
```

---

### 5. OrdersTable — Responsive avec useMediaQuery

**Fichier** : `app/frontend/src/features/orders/components/OrdersTable.tsx`
**CP démontrée** : CP2 — Développer des interfaces utilisateur
**Destination** : Corps §7.1

**Annotation** : Rendu conditionnel JS via `useMediaQuery` plutôt que CSS `hidden`. Sur mobile, le composant `Table` (lourd : TanStack Table + colonnes triables) n'est pas monté du tout — seul `MobileOrdersList` (cartes légères) est rendu. Cela réduit le poids DOM et le coût de rendu côté navigateur.

```tsx
export function OrdersTable({
  orders,
  onViewOrder,
  config: userConfig,
}: OrdersTableProps) {
  const config = React.useMemo(
    () => ({ ...defaultConfig, ...userConfig }),
    [userConfig],
  );

  const isDesktop = useMediaQuery("(min-width: 768px)");
  const [sorting, setSorting] = React.useState<SortingState>([]);

  // ... définition des colonnes ...

  const table = useReactTable({
    data: orders,
    columns,
    state: { sorting },
    onSortingChange: setSorting,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: config.enableSorting ? getSortedRowModel() : undefined,
    getPaginationRowModel: config.enablePagination
      ? getPaginationRowModel()
      : undefined,
  });

  // Rendu conditionnel : mobile → cartes légères, desktop → table complète
  if (!isDesktop) {
    return (
      <div className="space-y-4">
        <MobileOrdersList
          orders={orders}
          onViewOrder={onViewOrder}
          config={config}
        />
      </div>
    );
  }

  return (
    <div className="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl shadow-sm overflow-hidden">
      <div className="overflow-x-auto">
        <Table>
          {/* ... rendu table desktop ... */}
        </Table>
      </div>
      {/* ... pagination ... */}
    </div>
  );
}
```

---

## §7.2 — Composants métier (CP3)

> Fil rouge : le cycle de vie d'une commande — de l'initialisation au nettoyage.
> Fil conducteur : chaque composant garantit l'intégrité des données (commandes, stock, paiements).

### 1. initCheckout — Réutilisation intelligente des commandes

**Fichier** : `app/backend/src/checkout/checkout.service.ts`
**CP démontrée** : CP3 — Développer des composants métier
**Destination** : Corps §7.2

**Annotation** : Stratégie "stock au paiement" — le stock n'est pas décrémenté à l'init pour éviter de bloquer du stock sur des paniers abandonnés. Le service cherche une commande PENDING réutilisable (< 30 min) avant d'en créer une nouvelle, ce qui évite de polluer la base avec des commandes fantômes. Le PaymentIntent Stripe est créé ou mis à jour selon le cas.

```typescript
// app/backend/src/checkout/checkout.service.ts
const CHECKOUT_TIMEOUT_MS = 30 * 60 * 1000; // 30 minutes

async initCheckout(
  userId: number,
  items: CheckoutItemDto[],
  contactEmail: string,
  billingAddress: BillingAddressDto,
) {
  return this.prisma.$transaction(async (tx) => {
    const { currency } = await this.validateCountry(tx, billingAddress.countryCode);

    // Chercher une commande PENDING existante (moins de 30 min)
    // Permet de réutiliser la même commande si l'utilisateur navigue
    const existingOrder = await this.findReusableOrder(tx, userId);

    let orderId: number;
    let totalAmount: number;

    if (existingOrder) {
      // L'utilisateur revient modifier son panier avant paiement
      orderId = existingOrder.id;
      const totals = await this.syncOrderItems(tx, orderId, items);
      totalAmount = totals.totalAmount;
      await tx.order.update({
        where: { id: orderId },
        data: { totalAmount: totals.totalAmount, totalExclTax: totals.totalExclTax,
                totalVatAmount: totals.totalVatAmount, currency, contactEmail,
                paymentStatus: PaymentStatus.PENDING },
      });
      await this.upsertBillingAddress(tx, orderId, billingAddress);
    } else {
      // Nouvelle commande, premier passage au checkout
      const newOrder = await this.createNewOrder(tx, userId, items, contactEmail,
                                                  billingAddress, currency);
      orderId = newOrder.id;
      totalAmount = newOrder.totalAmount;
    }

    // Crée ou met à jour le PaymentIntent avec le montant actuel
    const clientSecret = await this.handleStripePaymentIntent(tx, orderId, totalAmount);
    return { orderId, clientSecret };
  });
}
```

```typescript
// Même fichier — Critères de recherche d'une commande réutilisable
private async findReusableOrder(tx: Prisma.TransactionClient, userId: number) {
  const timeoutThreshold = new Date(Date.now() - CHECKOUT_TIMEOUT_MS);

  return tx.order.findFirst({
    where: {
      userId,
      orderStatus: OrderStatus.PENDING,
      updatedAt: { gt: timeoutThreshold },
      paymentStatus: { in: [PaymentStatus.PENDING, PaymentStatus.FAILED] },
      stockReservedAt: null, // Pas encore confirmé
    },
    include: { items: true },
  });
}
```

---

### 2. confirmCheckout — Pièce maîtresse : vérification et réservation de stock

**Fichier** : `app/backend/src/checkout/checkout.service.ts`
**CP démontrée** : CP3 — Développer des composants métier
**Destination** : Corps §7.2

**Annotation** : Méthode centrale du tunnel de vente. Vérifications de sécurité (propriétaire, statut), garde d'idempotence (`stockReservedAt` empêche la double-décrémentation en cas de double-clic), validation stock pour TOUS les items avant décrémentation, puis transaction atomique Prisma. Retour structuré avec `issues[]` typées (`out-of-stock` | `insufficient`) pour feedback précis au frontend.

```typescript
// app/backend/src/checkout/checkout.service.ts
async confirmCheckout(userId: number, orderId: number): Promise<CheckoutConfirmResponseDto> {
  return this.prisma.$transaction(async (tx) => {
    // 1. Récupérer la commande avec ses items et le stock de chaque arbre
    const order = await tx.order.findUnique({
      where: { id: orderId },
      include: { items: { include: { tree: { include: { stock: true } } } } },
    });

    // 2. Vérifications de sécurité
    if (!order) throw new NotFoundException('Commande introuvable');
    if (order.userId !== userId)
      throw new ForbiddenException("Vous n'êtes pas autorisé à accéder à cette commande");
    if (order.orderStatus !== OrderStatus.PENDING)
      throw new BadRequestException('Cette commande ne peut plus être modifiée');
    if (order.paymentStatus === PaymentStatus.PAID)
      throw new BadRequestException('Cette commande a déjà été payée');

    // 3. Garde d'idempotence : évite la double-décrémentation (double-clic)
    if (order.stockReservedAt || order.paymentStatus === PaymentStatus.PROCESSING) {
      return { success: true };
    }

    // 4. Vérifier la disponibilité du stock pour TOUS les items
    const issues: StockIssueDto[] = [];
    for (const item of order.items) {
      const available = item.tree.stock?.quantity ?? 0;
      if (available <= 0) {
        issues.push({ treeId: item.treeId, treeName: item.tree.name, type: 'out-of-stock' });
      } else if (available < item.quantity) {
        issues.push({ treeId: item.treeId, treeName: item.tree.name,
                      type: 'insufficient', requested: item.quantity, available });
      }
    }

    // 5. Si problèmes de stock, retourner sans décrémenter
    if (issues.length > 0) {
      return { success: false, issues };
    }

    // 6. Stock OK : décrémenter atomiquement
    for (const item of order.items) {
      await tx.treeStock.update({
        where: { treeId: item.treeId },
        data: { quantity: { decrement: item.quantity } },
      });
    }

    // 7. Marquer la commande : stock réservé + paiement en cours
    await tx.order.update({
      where: { id: orderId },
      data: { stockReservedAt: new Date(), paymentStatus: PaymentStatus.PROCESSING },
    });

    return { success: true };
  });
}
```

---

### 3. handleWebhook + handlePaymentSuccess — Source de vérité paiement

**Fichier** : `app/backend/src/payment/payment.service.ts`
**CP démontrée** : CP3 — Développer des composants métier
**Destination** : Corps §7.2

**Annotation** : Le webhook Stripe est la seule source de vérité pour les mises à jour de paiement (communication serveur-à-serveur, indépendante du client). Vérification de la signature HMAC, routing vers 3 handlers, puis dans `handlePaymentSuccess` : vérification anti-spoofing du `paymentIntentId`, idempotence (déjà PAID → return), génération de facture atomique, et envoi d'email non bloquant. Le endpoint est décoré `@SkipThrottle()` car les webhooks ne doivent jamais être rate-limités.

```typescript
// app/backend/src/payment/payment.service.ts
async handleWebhook(signature: string, rawBody: Buffer) {
  // Vérification de la signature HMAC-SHA256
  let event: Stripe.Event;
  try {
    const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET;
    event = this.stripe.webhooks.constructEvent(rawBody, signature, webhookSecret!);
  } catch (err) {
    throw new BadRequestException('Signature du webhook invalide');
  }

  // Routing selon le type d'événement
  switch (event.type) {
    case 'payment_intent.succeeded':
      await this.handlePaymentSuccess(event.data.object);
      break;
    case 'payment_intent.payment_failed':
      await this.handlePaymentFailed(event.data.object);
      break;
    case 'payment_intent.canceled':
      await this.handlePaymentCanceled(event.data.object);
      break;
  }
  return { received: true };
}
```

```typescript
// Même fichier — Confirmation paiement avec sécurité anti-spoofing
private async handlePaymentSuccess(paymentIntent: Stripe.PaymentIntent) {
  const orderId = parseInt(paymentIntent.metadata.orderId, 10);
  const order = await this.prisma.order.findUnique({ where: { id: orderId } });

  // Anti-spoofing : le paymentIntentId DOIT correspondre à celui stocké en base
  if (order?.paymentIntentId !== paymentIntent.id) {
    this.logger.error(`SÉCURITÉ : PaymentIntent ${paymentIntent.id} ne correspond pas`);
    return;
  }

  // Idempotence : éviter le double traitement
  if (order.paymentStatus === PaymentStatus.PAID) return;

  // Transaction : confirmation + génération numéro de facture atomique
  await this.prisma.$transaction(async (tx) => {
    const invoiceNumber = await generateInvoiceNumber(tx);
    await tx.order.update({
      where: { id: orderId },
      data: { paymentStatus: PaymentStatus.PAID, orderStatus: OrderStatus.CONFIRMED,
              invoiceNumber, invoiceDate: new Date() },
    });
  });

  // Email de confirmation (non bloquant — un échec n'annule pas la commande)
  try {
    // ... chargement commande avec relations → emailService.sendOrderConfirmationEmail()
  } catch (emailError) {
    this.logger.error(`Échec envoi email pour commande #${orderId}`);
  }
}
```

---

### 4. OrdersCleanupService — Réconciliation et nettoyage

**Fichier** : `app/backend/src/orders/orders-cleanup.service.ts`
**CP démontrée** : CP3 — Développer des composants métier
**Destination** : Corps §7.2

**Annotation** : Cron horaire qui gère deux cas distincts de commandes zombies : (1) les commandes vraiment abandonnées (PENDING/FAILED) → annulation + restauration stock + cancel PaymentIntent Stripe, et (2) les commandes bloquées en PROCESSING → réconciliation avec l'API Stripe pour récupérer l'état réel du paiement. Le cas 2 est critique : il évite de perdre des commandes payées dont le webhook s'est perdu entre les deux infrastructures.

```typescript
// app/backend/src/orders/orders-cleanup.service.ts
const ABANDONED_ORDER_THRESHOLD_MS = 30 * 60 * 1000; // 30 minutes

@Cron(CronExpression.EVERY_HOUR)
async cleanupAbandonedOrders() {
  const abandonedThreshold = new Date(Date.now() - ABANDONED_ORDER_THRESHOLD_MS);

  // Cas 1 : commandes vraiment abandonnées (client parti ou paiement refusé)
  const abandonedOrders = await this.prisma.order.findMany({
    where: {
      orderStatus: OrderStatus.PENDING,
      updatedAt: { lt: abandonedThreshold },
      paymentStatus: { in: [PaymentStatus.PENDING, PaymentStatus.FAILED] },
    },
    include: { items: true },
  });

  // Cas 2 : paiement accepté côté Stripe mais webhook jamais reçu
  const stuckProcessingOrders = await this.prisma.order.findMany({
    where: {
      orderStatus: OrderStatus.PENDING,
      updatedAt: { lt: abandonedThreshold },
      paymentStatus: PaymentStatus.PROCESSING,
    },
    include: { items: true },
  });

  await this.handleAbandonedOrders(abandonedOrders);
  await this.reconcileProcessingOrders(stuckProcessingOrders);
}
```

```typescript
// Même fichier — Réconciliation avec l'API Stripe (cas critique)
private async reconcileProcessingOrders(orders: /* ... */) {
  for (const order of orders) {
    // Interroger Stripe : quel est l'état RÉEL du paiement ?
    const paymentIntent = await this.stripe.paymentIntents.retrieve(order.paymentIntentId!);

    if (paymentIntent.status === 'succeeded') {
      // Le webhook s'est perdu mais l'argent est là → finaliser la commande
      await this.prisma.$transaction(async (tx) => {
        const invoiceNumber = await generateInvoiceNumber(tx);
        await tx.order.update({
          where: { id: order.id },
          data: { paymentStatus: PaymentStatus.PAID, orderStatus: OrderStatus.CONFIRMED,
                  invoiceNumber, invoiceDate: new Date() },
        });
      });
    } else if (['canceled', 'failed'].includes(paymentIntent.status)) {
      // Paiement échoué/annulé → annuler et restaurer le stock
      await this.handleAbandonedOrders([order]);
    }
    // Sinon (processing, requires_action) → attendre le prochain cron
  }
}
```

---

### 5. updateOrderStatus — State machine explicite + annulation

**Fichier** : `app/backend/src/orders/orders.service.ts` + `orders.constants.ts`
**CP démontrée** : CP3 — Développer des composants métier
**Destination** : Corps §7.2

**Annotation** : Machine à états explicite via la constante `ORDER_STATUS_TRANSITIONS` qui centralise les transitions autorisées (lisibilité, extensibilité, maintenabilité vs if/else). Le cas CANCELLED applique un pattern fail-early : le refund Stripe est tenté AVANT la transaction BDD — si Stripe refuse le remboursement, la commande reste inchangée. La restauration de stock se fait en transaction atomique avec la mise à jour du statut.

```typescript
// app/backend/src/orders/orders.constants.ts
export const ORDER_STATUS_TRANSITIONS: Record<OrderStatus, OrderStatus[]> = {
  PENDING: [OrderStatus.CANCELLED],
  CONFIRMED: [OrderStatus.PLANTED, OrderStatus.CANCELLED],
  PLANTED: [],      // État final
  CANCELLED: [],    // État final
};
```

```typescript
// app/backend/src/orders/orders.service.ts
async updateOrderStatus(id: number, updateOrderDto: UpdateOrderDto) {
  const order = await this.prisma.order.findUnique({
    where: { id }, include: { items: true },
  });
  if (!order) throw new NotFoundException('Commande introuvable');

  const newStatus = updateOrderDto.orderStatus;

  // Validation de la transition via la constante
  const allowedTransitions = ORDER_STATUS_TRANSITIONS[order.orderStatus] || [];
  if (!allowedTransitions.includes(newStatus!)) {
    throw new BadRequestException(
      `Transition non autorisée : ${ORDER_STATUS_LABELS[order.orderStatus]} → ${ORDER_STATUS_LABELS[newStatus!]}`,
    );
  }

  // Cas CANCELLED : refund Stripe (fail-early) puis restauration stock
  if (newStatus === OrderStatus.CANCELLED) {
    if (order.paymentStatus === PaymentStatus.PAID && order.paymentIntentId) {
      // Si le remboursement échoue → exception → la commande reste inchangée
      const refund = await this.stripe.refunds.create({ payment_intent: order.paymentIntentId });
      this.logger.log(`Commande #${id} : remboursement Stripe créé (${refund.id})`);
    }

    return this.prisma.$transaction(async (tx) => {
      // Restaurer le stock si il avait été réservé
      if (order.stockReservedAt) {
        for (const item of order.items) {
          await tx.treeStock.update({
            where: { treeId: item.treeId },
            data: { quantity: { increment: item.quantity } },
          });
        }
      }
      return tx.order.update({
        where: { id },
        data: {
          orderStatus: newStatus,
          stockReservedAt: null,
          paymentStatus: order.paymentStatus === PaymentStatus.PAID
            ? PaymentStatus.REFUNDED : order.paymentStatus,
        },
        // ... include relations
      });
    });
  }

  // Transition simple (ex: CONFIRMED → PLANTED)
  return this.prisma.order.update({
    where: { id }, data: { orderStatus: newStatus },
    // ... include relations
  });
}
```

## §7.3 — Composants d'accès aux données (CP8)

> Angle : accès SQL (Prisma ORM, requêtes avec relations, pagination) et NoSQL (Redis cache-aside, brute force).
> Les transactions atomiques ont été détaillées en §7.2 — ici on se concentre sur les patterns d'accès.

### 1. TREE_INCLUDES + pagination parallèle — Accès SQL avec relations

**Fichier** : `app/backend/src/trees/trees.service.ts`
**CP démontrée** : CP8 — Développer des composants d'accès aux données SQL et NoSQL
**Destination** : Corps §7.3

**Annotation** : La constante `TREE_INCLUDES` centralise la profondeur d'include (stock, category, location → country) en un seul endroit — réutilisée dans `findAll`, `findOne`, `create`, `update`. La pagination utilise `Promise.all` pour exécuter `findMany` et `count` en parallèle : on gagne le temps de la plus lente au lieu de la somme des deux. Prisma traduit les includes en requêtes séparées batchées (pas de JOIN SQL), ce qui est acceptable pour le volume de GreenRoots.

```typescript
// app/backend/src/trees/trees.service.ts

// Configuration des includes réutilisable
const TREE_INCLUDES = {
  stock: true,
  category: true,
  location: {
    include: { country: true },
  },
} as const;

async findAll(query: GetTreesDto) {
  const { categoryId, search, orderBy, exclude, take, page = 1, limit = 10 } = query;
  const skip = (page - 1) * limit;

  // Construction des filtres
  const where: Prisma.TreeWhereInput = {};
  if (categoryId) { where.category = { id: categoryId }; }
  if (search) {
    where.OR = [
      { name: { contains: search, mode: 'insensitive' } },
      { scientificName: { contains: search, mode: 'insensitive' } },
    ];
  }
  // ... filtres exclude, tri

  // Récupération des arbres et du total en parallèle
  const [trees, total] = await Promise.all([
    this.prisma.tree.findMany({
      where,
      skip,
      take: resolvedLimit,
      include: TREE_INCLUDES,
      // ... orderBy
    }),
    this.prisma.tree.count({ where }),
  ]);

  return { data: trees, total, page, limit: resolvedLimit };
}
```

---

### 2. Cache-aside Redis — Categories avec invalidation sur écriture

**Fichier** : `app/backend/src/categories/categories.service.ts`
**CP démontrée** : CP8 — Développer des composants d'accès aux données SQL et NoSQL
**Destination** : Corps §7.3

**Annotation** : Pattern cache-aside complet : lecture (cache hit → retour immédiat, cache miss → requête PostgreSQL + stockage Redis avec TTL 1h) et invalidation (chaque écriture supprime les clés concernées). La dégradation gracieuse est transparente : si Redis est indisponible, `this.redis.get()` retourne `null` (cache miss) et l'API continue via PostgreSQL.

```typescript
// app/backend/src/categories/categories.service.ts
const CACHE_TTL = 3600; // 1 hour
const CACHE_KEY_LIST = 'categories:list';
const CACHE_KEY_PREFIX = 'categories:';

async findAll() {
  // 1. Vérifier le cache Redis
  const cached = await this.redis.get(CACHE_KEY_LIST);
  if (cached) {
    return JSON.parse(cached) as unknown[];
  }

  // 2. Cache miss → requête PostgreSQL
  const categories = await this.prisma.category.findMany({
    select: CATEGORY_SELECT,
    orderBy: { name: 'asc' },
  });

  // 3. Stocker en cache avec TTL
  await this.redis.set(CACHE_KEY_LIST, JSON.stringify(categories), CACHE_TTL);
  return categories;
}

// Invalidation sur chaque écriture (create, update, delete)
private async invalidateCache(id?: number) {
  await this.redis.del(CACHE_KEY_LIST);
  if (id) {
    await this.redis.del(`${CACHE_KEY_PREFIX}${id}`);
  }
}
```

---

### 3. Brute force login — Redis INCR atomique + TTL

**Fichier** : `app/backend/src/auth/services/login-attempts.service.ts`
**CP démontrée** : CP8 — Développer des composants d'accès aux données SQL et NoSQL
**Destination** : Corps §7.3

**Annotation** : Usage de Redis distinct du cache : compteur de tentatives de login par email avec `INCR` atomique (pas de race condition) et TTL automatique (15 min). Après 5 échecs, le compte est verrouillé. Le TTL fait que le compteur se réinitialise tout seul sans cron de nettoyage. Pattern "fail open" si Redis est down : on ne bloque personne (`isAvailable()` → false → return 0/false).

```typescript
// app/backend/src/auth/services/login-attempts.service.ts
private readonly MAX_ATTEMPTS = 5;
private readonly LOCKOUT_DURATION_SECONDS = 15 * 60; // 15 minutes
private readonly KEY_PREFIX = 'login_attempts:';

async recordFailure(email: string): Promise<number> {
  if (!this.redisService.isAvailable()) return 0; // Fail open

  const key = this.getKey(email);
  // INCR atomique + TTL posé au premier échec uniquement
  const attempts = await this.redisService.incr(key, this.LOCKOUT_DURATION_SECONDS);

  if (attempts >= this.MAX_ATTEMPTS) {
    this.logger.warn(`Account locked for email: ${this.maskEmail(email)}`);
  }
  return attempts;
}

async isLocked(email: string): Promise<boolean> {
  if (!this.redisService.isAvailable()) return false; // Fail open

  const attemptsStr = await this.redisService.get(this.getKey(email));
  return attemptsStr ? parseInt(attemptsStr, 10) >= this.MAX_ATTEMPTS : false;
}

async clearAttempts(email: string): Promise<void> {
  if (!this.redisService.isAvailable()) return;
  await this.redisService.del(this.getKey(email));
}
```

## §7.4 — Autres composants significatifs

> Section courte (~0.75 page). Un seul extrait de code (Stripe module), le reste en texte.

### 1. Stripe module — Singleton global via factory

**Fichier** : `app/backend/src/stripe/stripe.module.ts`
**CP démontrée** : CP3/CP6 — Architecture modulaire NestJS
**Destination** : Corps §7.4

**Annotation** : Module `@Global()` (accessible partout sans import explicite) avec `useFactory` pour construire l'instance Stripe au démarrage. Injection du `ConfigService` pour lire la clé secrète, vérification fail-fast si absente, version API pinnée. Le résultat est un singleton injecté via DI dans checkout, payment et orders.

```typescript
// app/backend/src/stripe/stripe.module.ts
@Global()
@Module({
  providers: [
    {
      provide: Stripe,
      useFactory: (configService: ConfigService) => {
        const secretKey = configService.get<string>('STRIPE_SECRET_KEY');
        if (!secretKey) {
          throw new Error('STRIPE_SECRET_KEY is not defined');
        }
        return new Stripe(secretKey, {
          apiVersion: '2025-11-17.clover',
        });
      },
      inject: [ConfigService],
    },
  ],
  exports: [Stripe],
})
export class StripeModule {}
```
