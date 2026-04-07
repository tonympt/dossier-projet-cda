# Diagramme de séquence — Processus de commande

```mermaid
sequenceDiagram
    actor U as Utilisateur
    participant F as Frontend
    participant B as Backend
    participant DB as PostgreSQL
    participant S as Stripe

    note over F,S: Phase 1 — Initialisation (stock non réservé)
    U->>F: Clic "Commander"
    F->>B: POST /checkout/init (panier, adresse)
    B->>DB: Créer Order + OrderItems
    B->>S: Créer PaymentIntent (montant)
    S-->>B: clientSecret
    B-->>F: orderId + clientSecret

    note over F,S: Phase 2 — Réservation stock (transaction atomique)
    F->>B: POST /checkout/confirm (orderId)
    B->>DB: Vérifier stock + réserver (transaction)
    B-->>F: { success: true }

    note over F,S: Phase 3 — Paiement (données carte directes)
    F->>S: confirmPayment(clientSecret)

    note over F,S: Phase 4 — Confirmation (webhook serveur-à-serveur)
    S->>B: webhook payment_intent.succeeded (signature HMAC)
    B->>DB: Order CONFIRMED + Payment PAID
    B->>U: Email confirmation

    note over F,S: Phase 5 — Affichage
    F->>B: GET /orders/:id (polling)
    B-->>F: Commande confirmée
    F-->>U: Page de confirmation
```

> **Points clés** : Le stock n'est réservé qu'au moment du confirm (phase 2), pas à l'initialisation — cela minimise le blocage de stock. Le webhook Stripe (phase 4) est la source de vérité pour confirmer le paiement. Les données bancaires ne transitent jamais par le backend.
