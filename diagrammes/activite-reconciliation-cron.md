# Diagramme d'activité — Réconciliation des commandes (CRON)

> UML activité (rendu Mermaid `flowchart` + `subgraph` = couloirs). Processus **automatisé**
> `OrdersCleanupService`, déclenché toutes les heures (`@Cron(EVERY_HOUR)`). Filet de sécurité
> pour les commandes « zombies » restées incohérentes après 30 min.
> Couloirs : **CRON** (orchestration/décisions) · **API Stripe** · **Base de données**.

```mermaid
flowchart TD
    subgraph CRON["CRON · OrdersCleanupService — @Cron(EVERY_HOUR)"]
        direction TB
        start([Déclenchement horaire]) --> scan[Récupérer les commandes<br/>bloquées depuis plus de 30 min]
        scan --> dtype{Statut de paiement ?}
        sdec{État réel renvoyé<br/>par Stripe ?}
        wait[Ne rien faire<br/>prochain CRON réessaiera]
        stop([Fin])
    end

    subgraph STRIPE["API Stripe"]
        cancelPI[Annuler le PaymentIntent]
        queryPI[Interroger l'état<br/>du PaymentIntent]
    end

    subgraph BDD["Base de données · transaction"]
        abandon[Restaurer le stock réservé<br/>commande CANCELLED / FAILED]
        finalize[Commande PAID / CONFIRMED<br/>génération de la facture]
        cancelFree[Commande CANCELLED<br/>restauration du stock]
    end

    dtype -->|"PENDING / FAILED (abandonnée)"| abandon
    abandon --> cancelPI --> stop

    dtype -->|"PROCESSING (webhook perdu)"| queryPI
    queryPI --> sdec
    sdec -->|succeeded| finalize --> stop
    sdec -->|"canceled / failed"| cancelFree --> stop
    sdec -->|"requires_action / processing (3D Secure)"| wait --> stop
```

**Lecture** : le CRON récupère les commandes bloquées, puis branche selon le statut de paiement.
Les commandes *abandonnées* (PENDING/FAILED) sont annulées et leur stock restauré. Les commandes
*bloquées en PROCESSING* (webhook perdu) déclenchent une **interrogation directe de Stripe** :
paiement réussi → on finalise (évite de perdre une vente payée) ; annulé/échoué → on annule et
libère le stock ; en attente (3D Secure) → on laisse le prochain passage réessayer.
