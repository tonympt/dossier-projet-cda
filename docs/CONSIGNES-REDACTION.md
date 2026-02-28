# Consignes de rédaction - Dossier de Projet CDA

## Contraintes de format (Référentiel d'Évaluation)

| Élément | Exigence |
|---|---|
| Corps du dossier | **40 à 60 pages** (schémas et illustrations compris) |
| Annexes | **40 pages maximum** |
| Exclus du comptage | Page de garde, sommaire |
| Format | Dossier imprimé unique réunissant l'ensemble des projets |
| Support complémentaire | Diaporama de présentation orale (à préparer séparément) |

## Productions à fournir
1. **Un seul dossier de projet imprimé** réunissant l'ensemble des projets
2. **Un seul support de présentation** de type diaporama

## Évaluation
Le dossier est évalué lors de :
- **Présentation orale du projet** (avec diaporama)
- **Entretien technique** (questions du jury à partir du dossier)
- **Questionnaire professionnel** (écrit, sans internet)

## Points d'attention pour la rédaction
- **Sécurité** : fil rouge omniprésent, justifier les choix de sécurité dans chaque section
- **RGPD** : mentions légales, protection des données personnelles
- **RGAA** : accessibilité, prise en compte des personnes en situation de handicap
- **Éco-conception** : fil conducteur à évoquer dans plusieurs sections (§5.2 architecture, §7.1 interfaces/performance, et potentiellement §2 pour la cohérence mission/éco). Argument fort : cohérence entre la mission du projet (reforestation) et la démarche technique (réduire l'empreinte numérique). Référentiels : WSG (W3C), RGESN 2024, RWEB GreenIT v5
- **Anglais** : la documentation technique doit pouvoir être comprise en anglais (B1)
- **Veille sécurité** : section dédiée obligatoire sur les vulnérabilités trouvées/corrigées
- **Posture prod** : ne pas survendre "prod-ready", dire honnêtement "quasi prod-ready" avec liste des éléments manquants — montre de la maturité au jury

## Évolutions techniques prévues (à implémenter avant le passage + documenter dans les sections adéquates)

### Priorité 1 — Corrections rapides (< 1h)
- **Email confirmation de commande** (~15 min) :
  - Le service `sendOrderConfirmationEmail()` existe, le template HTML est prêt
  - Il manque juste l'appel dans `payment.service.ts` → `handlePaymentSuccess()` (TODO existant)
  - Un quasi one-liner à ajouter
  - Sections concernées : §7.2, §7.4
- **npm audit + ajout CI** (~30 min) :
  - Lancer `npm audit` sur backend et frontend, corriger les vulnérabilités
  - Ajouter `npm audit --audit-level=high` dans le workflow `pr-checks`
  - Section concernée : §10 (veille sécurité)

### Priorité 2 — RGPD et conformité légale (~5-7h)
- **RGPD — Suppression de compte + anonymisation** (~3-4h) :
  - Endpoint `DELETE /users/me` avec confirmation
  - Anonymisation des commandes existantes (obligation comptable 6 ans) : remplacer nom/email/adresses par "Utilisateur supprimé", supprimer le User
  - Transaction Prisma : anonymiser OrderAddress + Order.contactEmail → supprimer User
  - Bouton dans le profil frontend avec dialog de confirmation
  - Sections concernées : §7.2, §7.3, §8.5
- **RGPD — Export de données** (~2-3h) :
  - Endpoint `GET /users/me/export`
  - Retourne JSON structuré : profil, adresses, historique commandes
  - Bouton dans le profil frontend
  - Sections concernées : §7.2, §8.5
- **~~TVA stockée en BDD~~** ✅ FAIT (branche `feature/billing`) :
  - Champs ajoutés sur Order : `totalExclTax`, `totalVatAmount`, `currency`
  - Constantes TVA : `DEFAULT_VAT_RATE = 2000` (20%), fonctions `calculateVatAmount()`, `calculateAmountInclTax()`
  - Facturation PDF : `invoice.service.ts` (491 lignes) avec `@react-pdf/renderer`, mentions légales vendeur (CGI art. 289), numéro de facture
  - Endpoint téléchargement PDF + boutons frontend (OrdersTable, OrderDetailPage)
  - Migration Prisma appliquée
  - ⚠️ Branche pas encore mergée dans develop/main — à merger
  - Sections concernées : §5.4, §7.2, §7.3, §7.4
- **Bannière cookies** : NON nécessaire — pas de tracking ni d'analytics, cookies fonctionnels uniquement (exemptés directive ePrivacy)

### Priorité 3 — Sécurité et qualité (~7-10h)
- **JWT HttpOnly cookies** (~4-6h) :
  - Migration du stockage localStorage vers cookies HttpOnly+Secure+SameSite
  - Modification auth backend (set-cookie) + frontend (suppression localStorage, credentials: include)
  - Sections concernées : §8.1, §5.6 (diagramme auth)
- **Tests composants React** (~2-3h) :
  - Ajouter 2-3 tests composants avec Testing Library : `CartItem` (affichage, modification qty, suppression), `CheckoutForm` (validation Zod, soumission)
  - Pattern : render, screen.getByRole, userEvent, waitFor
  - Objectif : compléter la couverture frontend (CP2+CP9)
  - Section concernée : §9

### Priorité 4 — Améliorations techniques (~3-4h)
- **Redis cache API** (~3-4h) :
  - Étendre l'usage de Redis au-delà du brute force/blacklist vers le cache de routes (pays, catégories, catalogue)
  - Sections concernées : §5.2, §7.3

### Récapitulatif

| Tâche | Estimation | Priorité |
|---|---|---|
| Email confirmation commande | ~15 min | P1 |
| npm audit + ajout CI | ~30 min | P1 |
| RGPD suppression + anonymisation | ~3-4h | P2 |
| RGPD export données | ~2-3h | P2 |
| ~~TVA stockée en BDD~~ | ✅ FAIT | — |
| Facturation PDF (invoice.service) | ✅ FAIT (branche billing) | — |
| Merger branche feature/billing | ~15 min | P1 |
| JWT HttpOnly cookies | ~4-6h | P3 |
| Tests composants React (2-3) | ~2-3h | P3 |
| Redis cache API | ~3-4h | P4 |
| **TOTAL restant** | **~2 jours** | |
