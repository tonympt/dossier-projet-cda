# Diagramme d'activité — Réconciliation des commandes (CRON)

> Processus **automatisé** `OrdersCleanupService`, déclenché toutes les heures (`@Cron(EVERY_HOUR)`).
> Filet de sécurité pour les commandes « zombies » restées incohérentes au-delà de 30 min.
>
> **Format retenu : PlantUML** (diagramme d'activité UML, flux vertical compact).
> Source : `activite-reconciliation-cron.puml` · Image : `activite-reconciliation-cron.png`
> (rendu via [planttext.com](https://www.planttext.com) ou plantuml.com).
> Système responsable indiqué par préfixe **[CRON] / [Stripe] / [BDD]**.

![Diagramme d'activité — réconciliation CRON](activite-reconciliation-cron.png)

## Lecture

Le CRON récupère les commandes bloquées (> 30 min) puis branche selon le statut de paiement :
- **Abandonnées** (PENDING / FAILED) → stock restauré (s'il avait été réservé), commande CANCELLED/FAILED, PaymentIntent Stripe annulé (best-effort).
- **Bloquées en PROCESSING** (webhook Stripe perdu) → interrogation directe de l'API Stripe :
  - *succeeded* → finalisation (PAID/CONFIRMED + facture) : on ne perd pas une vente payée ;
  - *canceled / failed* → annulation + libération du stock ;
  - *requires_action / processing* (3D Secure) → report au prochain passage.
- Cas défensif : une commande PROCESSING **sans** PaymentIntent est ignorée (log d'avertissement).

Robustesse : chaque itération est isolée par un `try/catch` — une erreur Stripe (timeout, PaymentIntent
introuvable) est journalisée sans interrompre le traitement des autres commandes.

> Le diagramme d'états (§5.6) décrit *les statuts* d'une commande ; ce diagramme d'activité décrit
> *l'algorithme* qui les réconcilie. Implémentation détaillée en §7.2.
