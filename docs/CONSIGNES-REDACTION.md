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

## Évolutions techniques implémentées (documentées dans les sections adéquates + `new-features/`)

### Priorité 1 — Corrections rapides (< 1h)
- **Email confirmation de commande** (~15 min) :
  - Le service `sendOrderConfirmationEmail()` existe, le template HTML est prêt
  - Il manque juste l'appel dans `payment.service.ts` → `handlePaymentSuccess()` (TODO existant)
  - Un quasi one-liner à ajouter
  - Sections concernées : §7.2, §7.4
- **~~npm audit + ajout CI~~** ✅ FAIT (branche `feature/npm-audit-ci`) :
  - `npm audit fix` exécuté sur backend (28→25 vulnérabilités) et frontend (6→3)
  - Restantes = dépendances transitives upstream (NestJS, Prisma, ESLint) non corrigeables sans breaking changes
  - Ajout de `npm audit --audit-level=critical` dans le workflow `pr-checks` (backend + frontend)
  - Niveaux : low → moderate → high → **critical** (seul niveau bloquant en CI)
  - Section concernée : §10 (veille sécurité)

### Priorité 2 — RGPD et conformité légale (~5-7h)
- **~~RGPD — Suppression de compte + anonymisation~~** ✅ FAIT (branche `feature/rgpd-account-deletion`) :
  - Endpoint `DELETE /auth/me` avec vérification mot de passe (bcrypt)
  - Transaction Prisma atomique : anonymisation OrderAddress + Order.contactEmail → suppression Cart → suppression User
  - Pattern anonymisation vs cascade delete : respect RGPD Art. 17 + obligation comptable 6 ans (L123-22)
  - Frontend : `DeleteAccountSection` avec AlertDialog, confirmation par mot de passe, clear cache + logout + redirect
  - Corrections annexes : `.gitattributes` (LF), normalisation Prettier (199 fichiers), `Dockerfile.prisma` (Prisma 7), imports inutilisés `invoice.service.ts`
  - Sections concernées : §7.2, §7.3, §8.5
- **~~RGPD — Export de données~~** ✅ FAIT (branche `feature/rgpd-data-export`) :
  - Endpoint `GET /auth/me/export` avec téléchargement JSON (Content-Disposition: attachment)
  - Données exportées : profil, commandes (items + adresses), panier — sans champs sensibles (password, tokens)
  - Frontend : `ExportDataSection` avec bouton de téléchargement Blob
  - Corrections annexes : suppression directives `eslint-disable prettier/prettier` obsolètes
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
- **~~JWT HttpOnly cookies~~** ✅ FAIT (PR develop, branche `feature/jwt-httponly-cookies`) :
  - Migration localStorage → cookies HttpOnly+Secure+SameSite=Lax
  - Backend : cookie-parser, helper `getCookieOptions()`, extracteur JWT custom depuis cookie, CORS credentials
  - Frontend : withCredentials axios, suppression accessToken du store Zustand, suppression interceptor Authorization
  - Fix boucle infinie React (JSON.stringify comparison dans useEffect)
  - Sections concernées : §8.1, §8.2, §5.6 (diagramme auth), §6.3
- **~~Tests composants React~~** ✅ FAIT (PR #3, branche `feature/react-component-tests`) :
  - LoadingSkeleton (4 tests : rendu par défaut, rows custom, rows=0, wrapper class)
  - QuantityInput (7 tests : valeur initiale, increment, decrement, min/max disable, disabled state, ARIA)
  - Config vitest extraite dans `vitest.config.ts` séparé (compatibilité tsc -b)
  - Résultat CI : 8/8 suites, 60/60 tests
  - Section concernée : §9, CP2

### Priorité 4 — Améliorations techniques (~3-4h)
- **~~Redis cache API~~** ✅ FAIT (PR #2, branche `feature/redis-cache-api`) :
  - Pattern cache-aside : countries billing/planting TTL 24h, categories list/detail TTL 1h
  - Invalidation cache catégories sur create/update/delete
  - Dégradation gracieuse si Redis indisponible
  - Sections concernées : §5.2, §7.3, §6.2

### Récapitulatif

| Tâche | Estimation | Priorité |
|---|---|---|
| ~~Email confirmation commande~~ | ✅ FAIT | — |
| ~~npm audit + ajout CI~~ | ✅ FAIT | — |
| ~~RGPD suppression + anonymisation~~ | ✅ FAIT | — |
| ~~RGPD export données~~ | ✅ FAIT | — |
| ~~TVA stockée en BDD~~ | ✅ FAIT | — |
| ~~Facturation PDF (invoice.service)~~ | ✅ FAIT (branche billing) | — |
| ~~Merger branche feature/billing~~ | ✅ FAIT | — |
| ~~JWT HttpOnly cookies~~ | ✅ FAIT | — |
| ~~Tests composants React~~ | ✅ FAIT (11 tests) | — |
| ~~Redis cache API~~ | ✅ FAIT | — |
| **TOTAL restant** | **0 — toutes les évolutions sont implémentées** | |
