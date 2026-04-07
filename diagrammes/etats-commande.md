# Diagramme d'états — Cycle de vie d'une commande

```mermaid
stateDiagram-v2
    direction LR

    state "OrderStatus" as OS {
        direction LR
        [*] --> PENDING: init checkout
        PENDING --> CONFIRMED: webhook succeeded
        CONFIRMED --> PLANTED: admin
        PENDING --> CANCELLED: abandon / admin
        CONFIRMED --> CANCELLED: admin (+ remboursement)
    }

    state "PaymentStatus" as PS {
        direction LR
        [*] --> PENDING_P: init checkout
        PENDING_P --> PROCESSING: confirm stock
        PROCESSING --> PAID: webhook succeeded
        PROCESSING --> FAILED: webhook failed
        PAID --> REFUNDED: annulation admin
        FAILED --> PROCESSING: retry paiement

        state "PENDING" as PENDING_P
    }
```

> **Points clés** : Les deux machines évoluent en parallèle — le webhook Stripe pilote PaymentStatus, qui déclenche la transition OrderStatus. Les transitions autorisées sont définies dans la constante `ORDER_STATUS_TRANSITIONS` du code. L'annulation d'une commande payée déclenche un remboursement Stripe + restauration du stock.
