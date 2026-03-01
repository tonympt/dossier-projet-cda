# Feature 01 — Email de confirmation de commande

## Priorité : P1 | Estimation : ~15 min

## Contexte
Le service `sendOrderConfirmationEmail()` et le template HTML existaient déjà dans le projet, mais n'étaient jamais appelés. Un TODO dans `payment.service.ts` marquait l'emplacement prévu.

## Modifications

### `app/backend/src/payment/payment.service.ts`
- **Import** de `EmailService` depuis le module email
- **Injection** de `EmailService` dans le constructeur de `PaymentService`
- **Appel** de `sendOrderConfirmationEmail()` après la transaction de confirmation de paiement dans `handlePaymentSuccess()`
- **Re-fetch** de la commande avec les relations `user` (email, prénom) et `items.tree` (nom de l'arbre) nécessaires au template
- **Try/catch** autour de l'envoi : un échec d'email ne bloque pas la confirmation du webhook Stripe

## Flux
1. Stripe envoie `payment_intent.succeeded` au webhook
2. `handlePaymentSuccess()` vérifie la sécurité (paymentIntentId) et met à jour le statut en base (PAID + CONFIRMED + facture)
3. **Nouveau** : re-fetch de la commande avec user + items, appel à `sendOrderConfirmationEmail()`
4. L'utilisateur reçoit un email avec le récap de sa commande et un lien vers son profil

## Branche
`feature/order-confirmation-email`

## Fichiers modifiés
- `app/backend/src/payment/payment.service.ts` (1 fichier, +23 lignes, -2 lignes)

## Sections du dossier CDA concernées
- §7.2 (composants métier back — webhook Stripe)
- §7.4 (intégrations tierces — Resend)
