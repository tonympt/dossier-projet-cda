# Plan du Dossier de Projet - GreenRoots (Projet en entreprise/formation)

> Plan imposé par le Référentiel d'Évaluation pour les projets réalisés pendant une période en entreprise.

## Structure du dossier

### Page de garde (hors comptage)
- Titre du projet : GreenRoots
- Titre professionnel : Concepteur Développeur d'Applications
- Nom du candidat
- Date

### Sommaire (hors comptage)

---

### 1. Liste des compétences mises en oeuvre (~1 page)
> Tableau récapitulatif des 11 CPs couvertes par le projet

| CP | Intitulé | Couverte | Section(s) du dossier |
|----|----------|----------|----------------------|
| CP1 | Installer et configurer son environnement de travail | Oui | §6 |
| CP2 | Développer des interfaces utilisateur | Oui | §7.1 |
| CP3 | Développer des composants métier | Oui | §7.2 |
| CP4 | Contribuer à la gestion d'un projet informatique | Oui | §4 |
| CP5 | Analyser les besoins et maquetter une application | Oui | §2, §5 |
| CP6 | Définir l'architecture logicielle d'une application | Oui | §5.2 |
| CP7 | Concevoir et mettre en place une BDD relationnelle | Oui | §5.4 |
| CP8 | Développer des composants d'accès aux données SQL et NoSQL | Oui | §7.3 |
| CP9 | Préparer et exécuter les plans de tests | Oui | §9 |
| CP10 | Préparer et documenter le déploiement | Oui | §6.2, Annexe D |
| CP11 | Contribuer à la mise en production (DevOps) | Oui | §6.2, §6.4, Annexe D |

**Réf. :** `docs/COMPETENCES.md`

---

### 2. Cahier des charges / Expression des besoins (~2-3 pages)
> **CP5** - Analyser les besoins
> Ce que le projet *demandait* — le besoin exprimé, côté produit. Ne pas empiéter sur les spécifications fonctionnelles (section 5).

- **Présentation du projet** : contexte GreenRoots, mission de reforestation, genèse
- **Objectifs** : ce que la plateforme doit accomplir (simplifier l'achat d'un arbre, assurer la transparence)
- **Cible utilisateurs** : particuliers sensibles à l'écologie, entreprises RSE
- **Acteurs du système** : définition des 3 rôles (Visiteur, Utilisateur authentifié, Admin)
- **Périmètre du MVP** : tableau des fonctionnalités retenues vs évolutions futures (suivi détaillé, carte interactive, paiement réel, multilingue…)
- **User stories principales** : sélection représentative des 3 rôles

> Contraintes, livrables et compatibilité navigateurs → section 5 (spécifications fonctionnelles)
> Analyse des risques → section 4 (gestion de projet)

---

### 3. Contexte du projet (~0,5 page)
> Adaptation de "Présentation de l'entreprise et du service" pour un projet de formation.
> Contexte statique uniquement — organisation de l'équipe et pilotage → section 4.

- Organisme de formation + cadre (projet de fin de formation, titre CDA)
- GreenRoots : projet fictif, mission, concept
- Rôle du candidat (Lead Dev Front, contribution étendue à l'ensemble du projet)

---

### 4. Gestion de projet (~3-4 pages)
> **CP4** - Contribuer à la gestion d'un projet informatique

- **Méthodologie** : Agile/Scrum, cycles courts de 1 semaine
  - Sprint 0 : conceptualisation (CDC, maquettes, architecture, BDD)
  - Sprint 1 : fondations du projet, premières fonctionnalités MVP
  - Sprint 2 : finalisation MVP + fonctionnalités additionnelles
  - Sprint 3 : audit sécurité (OWASP Top 10 + correctifs), SEO, accessibilité, score Lighthouse, éco-conception, mise en place CI
- **Planning et suivi** : Trello Kanban (captures à l'appui), backlog, tickets par sprint
- **Cérémonies** : daily meetings quotidiens, rétrospectives de fin de sprint, définition des objectifs du sprint suivant, évaluation des difficultés
- **Environnement humain** : rôles Scrum définis (PO, SM, Lead Dev Front, Lead Dev Back, Dev Full-stack) → voir section 3 pour le détail
- **Outils collaboratifs** : Git/GitHub (stratégie de branches documentée), Trello, communication quotidienne (évitement des conflits de code)
- **Objectifs de qualité** : conventions communes documentées, guidelines frontend (Atomic Design), PRs avec description précise et focalisée, code review obligatoire avant merge, ESLint + Prettier
- **Analyse des risques** : définie en sprint 0 dans le CDC (risques techniques, organisationnels, humains, sécurité) — référence à la section 2

---

### 5. Spécifications fonctionnelles (~8-10 pages)
> **CP5, CP6, CP7**

#### 5.1 Contraintes du projet et livrables attendus (~1 page)
- **Contraintes temporelles/humaines** : 4 semaines de développement (sprint 0 à 3), équipe de 5 personnes avec rôles définis
- **Contraintes techniques** : conteneurisation Docker obligatoire, TypeScript imposé front+back, responsive design obligatoire, compatibilité navigateurs (2 dernières versions majeures Chrome/Firefox/Edge/Safari)
- **Contraintes réglementaires** : RGPD (mentions légales, politique de confidentialité, suppression de compte), RGAA (accessibilité)
- **Contraintes fonctionnelles / Périmètre** : MVP borné aux fonctionnalités du CDC (tunnel d'achat simulé initialement, Stripe intégré au final), distinction claire MVP vs évolutions futures (carte interactive, multilingue, parrainage… → hors périmètre)
- **Livrables** : application MVP fonctionnelle et déployée, documentation technique (README front/back, API Swagger, conventions communes, guidelines frontend), code source versionné sur GitHub
- Note : le dossier de projet CDA n'est pas un livrable du projet en tant que tel

#### 5.2 Architecture logicielle du projet (~2-3 pages)
> **CP6** - Définir l'architecture logicielle
- **Architecture multicouche 3 tiers** : SPA React (Vite) → API REST NestJS → PostgreSQL + Redis, Nginx en reverse proxy
- **Justification des choix techniques** : pourquoi React/Vite, NestJS, PostgreSQL/Prisma, Redis, shadcn/ui (triple argument : import sélectif → éco-conception, basé sur Radix UI → accessibilité native RGAA, composants personnalisables → flexibilité) — bref ici, détail complet en §6
- **Architecture modulaire backend** : Controller → Service → Repository → Prisma, injection de dépendances
- **Design patterns** :
  - Backend : Injection de dépendances, Repository, Module, Guards/Decorators, State machine (statuts commandes)
  - Frontend : Atomic Design, Feature-based architecture, Store pattern (Zustand)
- **Diagramme d'architecture** : schéma des couches et flux (à créer en Mermaid)
- **Sécurité dans l'architecture** : sécurité à chaque couche (validation client Zod → validation API DTOs → ORM paramétré Prisma), référence OWASP
- **Éco-conception** : choix architecturaux (Vite build léger, Tailwind purge CSS, shadcn/ui import sélectif, Cloudinary optimisation images, code splitting lazy loading 25 routes, Redis cache API prévu), cohérence avec la mission GreenRoots (plateforme de reforestation), référentiels utilisés (WSG W3C, RGESN 2024, RWEB GreenIT v5)
- **Conteneurisation** : Docker Compose (8 services) comme partie intégrante de l'architecture — détail en §6

#### 5.3 Maquettes et enchaînement des maquettes (~2-3 pages)
> **CP5** - Maquetter une application
- **Wireframes / Zoning** (~0,5 page) : zoning des pages clés (landing, catalogue, fiche arbre, checkout) — à créer, montre la démarche de conception progressive
- **Maquettes Figma** (~0,5-1 page) : sélection des maquettes les plus représentatives (3-4 captures), conformité avec le CDC. Ensemble complet → Annexe A
- **Arborescence du site** (~0,5 page) : diagramme Mermaid de la structure des pages (publiques / authentifiées / admin)
- **Enchaînement des écrans** (~0,5 page) : 1-2 parcours utilisateur clés (parcours d'achat, authentification) en flowchart Mermaid

#### 5.4 Modèle de données (~2-3 pages)
> **CP7** - Concevoir et mettre en place une BDD relationnelle

- **MCD / Modèle entités-associations** (~1 page) :
  - Diagramme des 13 entités et leurs relations avec cardinalités (1-N, N-N)
  - Relations clés : User → Cart → CartItem → Tree, User → Order → OrderItem → Tree, Tree → Category, Tree → Location → Country
  - À créer (Mermaid erDiagram ou outil dédié)

- **MPD / Schéma Prisma** (~0,5-1 page) :
  - Extrait commenté des modèles les plus significatifs (Order/OrderItem pour le métier e-commerce, User pour l'auth)
  - Conventions de nommage : camelCase Prisma → snake_case PostgreSQL via `@map`/`@@map`
  - Types remarquables : enums (OrderStatus, PaymentStatus, AddressType), Int pour les prix (centimes — standard Stripe, évite les erreurs de floating point), PostGIS pour les coordonnées
  - Facturation : champs `totalExclTax`, `totalVatAmount`, `currency`, `invoiceNumber`, `invoiceDate` sur Order — conformité comptable e-commerce
  - Schéma Prisma complet → Annexe D

- **Contraintes d'intégrité** (~0,25 page) :
  - Clés étrangères (relations Prisma)
  - Contraintes d'unicité (email utilisateur, etc.)
  - Enums pour les machines à états (OrderStatus, PaymentStatus)
  - Valeurs par défaut, champs obligatoires vs optionnels
  - Stratégie de suppression (cascade vs protection)

- **Gestion des droits d'accès** (~0,25 page) :
  - Choix des droits au niveau applicatif (guards NestJS : USER/ADMIN/SUPERADMIN) plutôt qu'au niveau BDD (rôles PostgreSQL)
  - Justification : Prisma utilise une connexion unique, la gestion par guards est plus fine et testable, pattern moderne pour les API REST
  - Select sécurisé : exclusion systématique du password dans les requêtes

- **Sécurité et confidentialité des données** (~0,25 page) :
  - Mots de passe hashés (bcrypt, 10 rounds) — jamais en clair en BDD
  - Tokens de reset/vérification email hashés en BDD (post-audit sécurité)
  - RGPD : données minimales collectées, suppression de compte possible, politique de confidentialité

- **Jeu d'essai et restauration** (~0,25 page) :
  - Seed Prisma : données de test reproductibles et cohérentes
  - 25 migrations versionnées (historique complet de l'évolution du schéma)
  - Procédure de restauration dev : `prisma migrate reset` + `prisma db seed`
  - Procédure de sauvegarde/restauration prod : `pg_dump`/`pg_restore` (à documenter)

- **Éléments complémentaires (annexes)** :
  - Dictionnaire de données pour 2-3 entités clés (Order, User, Tree) → Annexe D
  - Index Prisma (`@@index`) sur les colonnes fréquemment requêtées (userId, categoryId, paymentStatus) — optimisation performance à implémenter

#### 5.5 Diagramme de cas d'utilisation (~1 page)
> **CP5**
- **Diagramme UML** (Mermaid) avec 3 acteurs et héritage d'accès :
  - **Visiteur** : consulter catalogue, voir fiche arbre, s'inscrire, se connecter, réinitialiser mdp, ajouter au panier
  - **Utilisateur authentifié** : (hérite Visiteur) + gérer panier, passer commande (checkout), consulter profil, modifier infos, historique commandes, supprimer compte, se déconnecter
  - **Admin** : (hérite Utilisateur) + CRUD arbres, gérer localisations/catégories, gérer commandes (annulation/statuts)
- **Relations UML** : 1-2 `<<include>>`/`<<extend>>` sur les cas pertinents (ex : "Passer commande" include "Vérifier stock" ; "Annuler commande" extend "Rembourser via Stripe")
- Héritage des acteurs = hiérarchie d'accès (dans le code : guards NestJS, pas d'héritage POO)
- Brève description textuelle sous le diagramme

#### 5.6 Diagrammes de séquence et d'états (~2-3 pages)
> **CP5, CP6**

- **Diagramme 1 : Processus de commande** (~1 page, Mermaid sequenceDiagram) :
  - 5 acteurs/systèmes : Utilisateur → Frontend → Backend → PostgreSQL → Stripe
  - Flux : validation panier → checkout/init (Order + PaymentIntent) → checkout/confirm (réservation stock atomique) → stripe.confirmPayment → webhook payment_intent.succeeded → polling confirmation
  - Points clés à légender : architecture multicouche, communication asynchrone (webhook = source de vérité), transaction atomique, binding PaymentIntent↔Order

- **Diagramme 2 : Authentification** (~0,5-1 page, Mermaid sequenceDiagram) :
  - 5 acteurs/systèmes : Utilisateur → Frontend → Backend → PostgreSQL → Redis
  - Sous-flux login : credentials → vérification brute force Redis → bcrypt compare → génération JWT → réponse (HttpOnly cookie ou token selon implémentation finale)
  - Sous-flux logout + accès protégé : requête avec JWT → vérification blacklist Redis → accès autorisé / ou logout → ajout token à la blacklist → requêtes suivantes rejetées
  - Points clés à légender : rôle de Redis (brute force + blacklist), sécurité multicouche
  - Note : le flux exact dépendra de l'implémentation HttpOnly cookies (à finaliser)

- **Diagramme 3 : Diagramme d'états — Cycle de vie d'une commande** (~0,5 page, Mermaid stateDiagram) :
  - Deux machines parallèles : **OrderStatus** et **PaymentStatus**
  - OrderStatus : PENDING → CONFIRMED (webhook succeeded) → PLANTED (admin, état final) ; PENDING/CONFIRMED → CANCELLED (abandon/admin, état final)
  - PaymentStatus : PENDING → PROCESSING (confirm stock) → PAID (webhook) → REFUNDED (annulation admin) ; PROCESSING → FAILED (webhook failed, stock restauré, retry possible)
  - Lien entre les deux : le webhook Stripe pilote la transition PaymentStatus qui déclenche la transition OrderStatus
  - Points clés à légender : transitions autorisées (constante `ORDER_STATUS_TRANSITIONS` dans le code), cas spéciaux (CANCELLED + remboursement Stripe + restauration stock), récupération des webhooks perdus (cron reconciliation)
  - Fait le pont entre conception (§5) et implémentation (§7.2)

---

### 6. Spécifications techniques (~3-4 pages)
> **CP1, CP6** - Environnement de travail et architecture

#### 6.1 Stack technique et justification des choix (~1 page)
- **Tableau récapitulatif** : technologie / version / rôle / justification
- Frontend : React 19 + Vite (HMR, build rapide, ES modules natifs), TanStack Router/Query (type-safe, cache intelligent), Zustand (léger vs Redux), Tailwind + shadcn/ui (éco-conception, accessibilité Radix UI, flexibilité)
- Backend : NestJS 11 (architecture modulaire, DI native, décorateurs, 14 modules dans `app.module.ts`)
- BDD : PostgreSQL 16 + PostGIS (données géospatiales localisation arbres)
- ORM : Prisma 7 (type-safety, migrations versionnées, 25 migrations)
- Cache/Protection : Redis (brute force + JWT blacklist, évolution vers cache API)
- Reverse proxy : Nginx (routing frontend/API, gzip, cache assets 1 an)
- Paiement : Stripe (webhooks source de vérité)
- Images : Cloudinary (CDN, transformations à la volée)
- Email : Resend (API moderne, bonne DX)
- Référence au §5.2 pour l'architecture globale ; ici on détaille le **comment** et les versions

#### 6.2 Environnement de développement et outillage (~1 page)
> **CP1** - Installer et configurer son environnement de travail
- **Docker Compose dev** (`compose.dev.yml`) : 7 services orchestrés
  - frontend (Vite dev server, hot reload via volume monté)
  - backend (NestJS dev, volume monté, commande auto : migrate → generate → seed → start:dev)
  - database (PostGIS 16-3.4, healthcheck pg_isready + SELECT 1)
  - redis (Alpine, healthcheck redis-cli ping)
  - nginx (reverse proxy : frontend :5173 + backend /api → :3000, gestion CORS)
  - stripe-cli (écoute webhooks, forward vers backend)
  - prisma-studio (exploration BDD visuelle, port 5555)
- **Docker Compose prod** (`compose.prod.yml`) : 3 services épurés (frontend, backend, database), `expose` au lieu de `ports`, healthchecks
  - ⚠️ **Déploiement prod (Coolify/Hetzner)** : mis entre parenthèses pour l'instant — pas géré par le candidat. À approfondir avant rédaction (§10/Annexes F-G)
- **Makefile** (~20 commandes documentées) : standardisation du workflow équipe
  - Cycle de vie : `make dev`, `make up`, `make down`, `make clean`
  - Reset complet : `make reset-backend` (down → rm vol → build → dev → generate → migrate → seed)
  - BDD : `make migrate-dev`, `make seed`, `make shell-db`
  - Qualité : `make lint-frontend`, `make lint-backend` (+ variantes fix)
  - Debug : `make logs`, `make logs-backend`, `make logs-frontend`, `make ps`
  - Stripe : `make stripe-tunnel-local`
- **Dockerfiles multi-stage** :
  - Backend prod : étape build (npm ci + build) → étape run (`node:22-alpine`, utilisateur non-root `node`)
  - Frontend prod : étape build (Vite) → Nginx avec gzip activé + cache 1 an sur assets statiques
  - Dev : images simples avec volumes montés pour hot reload
- **Variables d'environnement** : `.env.example` pour dev, `.env.production.example` avec instructions Coolify, validation Joi au démarrage backend (`ConfigModule` + `validationSchema`)
- Pas de dépendance à un IDE spécifique — le Makefile + Docker standardisent l'environnement

#### 6.3 Stratégie de sécurité (~1 page)
- **Couche réseau/serveur** :
  - Helmet (headers HTTP sécurisés, configuré dans `main.ts`)
  - CORS restrictif (origin configurable via `FRONTEND_URL`)
  - `expose` au lieu de `ports` en prod (pas d'accès direct aux containers)
- **Couche application** :
  - ValidationPipe global (`whitelist: true` + `forbidNonWhitelisted: true`) : sanitization automatique des entrées
  - ThrottlerGuard global (rate limiting via `@nestjs/throttler`)
  - Guards par rôle (USER, ADMIN, SUPERADMIN) via Passport.js
  - JWT (évolution prévue → HttpOnly cookies pour éliminer le risque XSS sur le token)
  - Bcrypt (10 rounds) pour hash mots de passe
  - Protection brute force login via Redis (compteur tentatives)
  - JWT blacklist Redis (invalidation au logout)
- **Couche données** :
  - Prisma ORM (requêtes paramétrées → pas d'injection SQL possible)
  - Transactions avec verrouillage pessimiste (réservation stock checkout)
  - Validation Joi des variables d'environnement au démarrage (fail-fast si config manquante)
  - Utilisateur non-root dans les containers de production
- **Audit OWASP Top 10:2025** : phases 1-2 réalisées sur le backend en sprint 3 (correctifs appliqués), audit frontend planifié — détail en §8
- **Stripe** : webhook secret pour vérification signatures, aucune donnée bancaire côté serveur (Stripe Elements)

#### 6.4 Documentation et qualité de code (~0,5-1 page)
- **CI/CD — 3 workflows GitHub Actions** :
  - `pr-checks` : lint + tests automatiques sur chaque Pull Request
  - `develop-ci` : intégration continue sur branche develop
  - `production-deploy` : déploiement via webhook Coolify + health check HTTP post-déploiement (vérification que l'app répond)
- **Linting/Formatting** : ESLint configuré front et back, commandes Makefile dédiées, exécuté aussi en CI
- **Tests** : Vitest + Testing Library (front), Jest (back) — détail en §9
- **Swagger** : documentation API auto-générée, activée en dev uniquement (`NODE_ENV` check dans `main.ts`)
- **Conventions** : document partagé (naming, Git flow, PR rules, code review obligatoire)
- **Husky + lint-staged** : pre-commit hook qui exécute ESLint + Prettier automatiquement avant chaque commit (qualité garantie dès le développeur)
- **Prisma Studio** : outil d'exploration BDD intégré en dev (service Docker dédié)

**Observations issues de l'exploration du code :**
- Le Makefile est un vrai atout DX (Developer Experience) : onboarding simplifié, workflow reproductible
- La différence dev/prod est clairement marquée (7 vs 3 services, volumes montés vs multi-stage, ports vs expose)
- La validation Joi au démarrage garantit le fail-fast si une variable d'environnement est manquante
- `vite.config.ts` : manual chunks configurés + plugin Critters (CSS critique inline) + target ES2020

**⚠️ Note** : le déploiement production (Coolify sur Hetzner) n'a pas été géré par le candidat. Le compose.prod.yml et les workflows CI/CD existent mais la mise en place Coolify a été faite par un autre membre de l'équipe. À clarifier et approfondir avant rédaction des sections 10/Annexes F-G — possibilité de reprendre la main sur cette partie pour la maîtriser au passage du titre.

---

### 7. Réalisations - Extraits de code significatifs (~10-12 pages)
> **CP2, CP3, CP8**

#### 7.1 Interfaces utilisateur (~3-4 pages)
> **CP2** - Développer des interfaces utilisateur
> Fil rouge : le tunnel de vente (panier → checkout → confirmation), complété par le back-office admin

- **Panier — Gestion d'état et synchronisation stock** :
  - `cartStore.ts` : Zustand avec persist (localStorage) + TTL 24h + action `syncWithStock()` qui réconcilie le panier local avec le stock serveur
  - `useCartProducts.tsx` : hook TanStack Query avec polling 5min, refetch au focus, enrichissement du cache individuel par arbre
  - `Cart.tsx` : gestion des 3 états de stock (OK, quantité réduite, épuisé) + dialog de notification `CartAlertDialogChangeStock`
  - Pattern : séparation état local (Zustand) / état serveur (TanStack Query) / état formulaire (React Hook Form)

- **Checkout — Formulaire et paiement 2 phases** :
  - `Checkout.tsx` : machine à états `PaymentState` (initializing → clientSecret → payment), formulaire verrouillé une fois le paiement initié (`isFormLocked`)
  - `CheckoutForm.tsx` : React Hook Form + schéma Zod (messages FR) + shadcn/ui FormField/FormControl, chargement dynamique des pays (TanStack Query, staleTime 1h)
  - `StripePaymentSection.tsx` : double confirmation (1. backend confirm stock → 2. stripe.confirmPayment), gestion des conflits stock tardifs via `CheckoutStockDialog`
  - `CheckoutSummary.tsx` : injection du formulaire Stripe en children (Stripe Elements monté uniquement après obtention du clientSecret)
  - Stripe : lazy loading du SDK, singleton, personnalisation apparence

- **Confirmation commande — Polling intelligent** :
  - `OrderConfirmation.tsx` : 6 états terminaux (PENDING, PROCESSING, PAID, FAILED, grace period webhook, not found)
  - Polling adaptatif : 2s si PROCESSING (max 30 tentatives), 5 tentatives de grâce si FAILED (attente webhook retardé), arrêt si PAID
  - Side effect : clearCart sur PAID

- **Back-office admin — Responsive adaptatif** :
  - `OrdersTable.tsx` : TanStack React Table (desktop) → card list (mobile) via `useMediaQuery` — pas juste du CSS, changement complet de composant
  - `TreeForm.tsx` : formulaire CRUD avec `ImageUpload` (Cloudinary drag-drop), sélecteurs dynamiques catégorie/localisation avec dialog "ajouter nouveau"
  - Guards frontend : `beforeLoad` dans la route admin vérifie le rôle, hydrate depuis `/auth/me` si stale

- **Patterns transversaux** :
  - Atomic Design : `QuantityInput` (molécule, aria-label, role spinbutton), composants shadcn/ui (atomes)
  - Responsive : `flex-col lg:flex-row`, sticky sidebar desktop, card layout mobile
  - Accessibilité : aria-labels sur boutons icônes, rôles ARIA, sémantique HTML (FormLabel, FormMessage)
  - Skeleton loading : `CartItemSkeleton`, `CheckoutFormSkeleton`, `LoadingSkeleton`

#### 7.2 Composants métier (~3-4 pages)
> **CP3** - Développer des composants métier
> Fil rouge : le tunnel de vente côté backend (checkout → paiement → nettoyage), complété par la gestion admin des commandes

- **`CheckoutService.confirmCheckout()`** — Pièce maîtresse (~1 page) :
  - Vérification propriétaire (`order.userId === userId`)
  - Idempotence : détection double-confirm via flag `stockReservedAt` (évite double-décrémentation stock)
  - Validation stock pour TOUS les items avant décrémentation (collecte `issues[]` : type `out-of-stock` | `insufficient`)
  - Transaction atomique Prisma : décrémentation stock (`decrement`) + marquage `stockReservedAt` + `paymentStatus=PROCESSING`
  - Retour structuré : `{ success: true }` ou `{ success: false, issues: [...] }`

- **`CheckoutService.initCheckout()`** — Réutilisation intelligente :
  - Recherche de commande PENDING réutilisable (< 30min, stock non réservé) → mise à jour au lieu de création
  - Gestion PaymentIntent Stripe : update montant si PI existant, create si nouveau ou expiré
  - Constante `CHECKOUT_TIMEOUT_MS = 30 * 60 * 1000`

- **`PaymentService.handleWebhook()`** — Sécurité webhook (~0,5 page) :
  - Vérification signature HMAC-SHA256 (`stripe.webhooks.constructEvent`)
  - Vérification correspondance `paymentIntentId` (anti-spoofing)
  - Idempotence : détection double-traitement (`paymentStatus === PAID` → return early)
  - 3 événements gérés : `payment_intent.succeeded` (PAID + CONFIRMED), `payment_intent.payment_failed` (restauration stock en transaction), `payment_intent.canceled`
  - `@SkipThrottle()` sur le endpoint (webhooks ne doivent jamais être rate-limités)

- **`OrdersCleanupService`** — Résilience et réconciliation (~0,5 page) :
  - Cron horaire (`@Cron(CronExpression.EVERY_HOUR)`)
  - Cas 1 — Commandes abandonnées : PENDING/FAILED > 30min → transaction (restauration stock + CANCELLED) + annulation PaymentIntent Stripe
  - Cas 2 — Webhooks perdus : PROCESSING > 30min → interrogation API Stripe pour vérité terrain → `succeeded` = finaliser, `failed` = annuler, `processing` = attendre prochain cron
  - Pattern production-ready rarement vu en formation

- **`OrdersService.updateOrderStatus()`** — Machine à états :
  - Validation transition via constante `ORDER_STATUS_TRANSITIONS` (lien avec le diagramme d'états §5.6)
  - Cas CANCELLED : remboursement Stripe (`stripe.refunds.create()`) + restauration stock en transaction + `paymentStatus=REFUNDED`

- **`InvoiceService`** — Facturation PDF :
  - Génération de factures PDF via `@react-pdf/renderer` (491 lignes)
  - Calculs TVA : constantes (`DEFAULT_VAT_RATE = 2000`), fonctions utilitaires (`calculateVatAmount`, `calculateAmountInclTax`), stockage HT + TVA + TTC en BDD
  - Mentions légales obligatoires vendeur (CGI art. 289 : SIRET, TVA intracommunautaire, capital)
  - Numéro de facture unique, date de facturation
  - Endpoint téléchargement PDF + boutons frontend (OrdersTable, OrderDetailPage)
  - Montre un composant métier complet : calcul → stockage → génération document → exposition API → UI

#### 7.3 Composants d'accès aux données (~2-3 pages)
> **CP8** - Développer des composants d'accès aux données SQL et NoSQL

- **Accès SQL — Transactions Prisma** :
  - Extrait de `confirmCheckout` : `$transaction` avec lecture → validation → écriture atomique, `decrement` sur chaque item, rollback automatique
  - Extrait de `updateOrderStatus` (CANCELLED) : restauration stock (`increment`) + mise à jour commande dans une même transaction
  - Pattern : toutes les opérations critiques (stock, paiement) sont transactionnelles

- **Accès SQL — Requêtes avec relations** :
  - `CartService.getBatchDetails()` : `findMany` avec `where: { id: { in: treeIds } }` + include imbriqué (stock, location → country, category), détection items introuvables (`notFoundIds`)
  - `OrdersService.findAll()` : requêtes parallèles `findMany()` + `count()` pour pagination, include user + items + trees
  - Pattern : `TREE_INCLUDES` réutilisable (constante d'include typée)

- **Accès NoSQL — Redis** :
  - Protection brute force login : compteur de tentatives par IP/email, TTL d'expiration
  - JWT blacklist : ajout du token au logout, vérification à chaque requête authentifiée
  - Couvre l'exigence CP8 "SQL et NoSQL" avec deux cas d'usage concrets

#### 7.4 Autres composants (~1-2 pages)

- **Stripe module** — Factory provider global :
  - `stripe.module.ts` : `@Global()` module, `useFactory` avec injection `ConfigService`, version API pinnée
  - Singleton injecté via DI NestJS dans checkout, payment, orders

- **Cloudinary — Upload signé** :
  - Backend (`cloudinary.service.ts`) : génère signature HMAC-SHA1 + timestamp, valide URL, extrait public_id
  - Frontend (`ImageUpload.tsx`) : drag-drop + validation fichier (5MB max, JPEG/PNG/WebP) → appel signature backend → upload direct vers API Cloudinary
  - Sécurité : le backend ne touche jamais le fichier image, la clé secrète reste côté serveur

- **`@react-pdf/renderer` — Génération PDF** :
  - Bibliothèque React pour générer des PDF côté serveur (dans NestJS)
  - Utilisée pour les factures : `renderToBuffer()` → Buffer PDF → endpoint téléchargement
  - Choix justifié : syntaxe React familière (composants, styles), pas besoin de Puppeteer/Chrome headless

- **Serialize Interceptor + `@PriceToDecimal()`** :
  - Interceptor NestJS qui transforme les réponses API via les DTOs (pattern class-transformer)
  - Decorator `@PriceToDecimal()` : convertit automatiquement les prix INT (centimes en BDD) → décimal (euros en API)
  - Justification : prix stockés en INT pour la précision (standard Stripe, pas de floating point), mais exposés en décimal pour le frontend
  - Montre le pattern interceptor/transformer NestJS

- **Guards + Decorators RBAC** :
  - `@Roles('ADMIN')` decorator + `RolesGuard` : lit les métadonnées de route, vérifie le rôle JWT, SUPER_ADMIN bypass
  - `@CurrentUser()` decorator : extrait userId/email/role du JWT dans le request
  - Composition : `@UseGuards(JwtAuthGuard, RolesGuard)` — ordre d'exécution garanti

---

### 8. Éléments de sécurité de l'application (~3-4 pages)
> Transversal sécurité (CP2, CP3, CP6, CP8)
> Organisé par menace/risque, pas par technologie — montre une démarche de sécurité réfléchie

#### 8.1 Authentification et gestion des sessions (~1 page)
- **JWT** : génération, expiration configurable (`JWT_EXPIRATION`), stratégie Passport.js
- **Bcrypt** (10 rounds) : hash mots de passe, jamais en clair en BDD
- **Tokens hashés** : reset password et vérification email hashés en BDD (post-audit sécurité sprint 3)
- **JWT blacklist Redis** : invalidation au logout, vérification à chaque requête authentifiée
- **Vérification email** : flow complet à l'inscription (token hashé, expiry 24h, guard login bloque si non vérifié)
- **Évolution HttpOnly cookies** : migration prévue localStorage → cookies HttpOnly+Secure+SameSite (élimine le risque XSS sur le token) — à implémenter

#### 8.2 Protection contre les attaques courantes (~1-1.5 page)
- **Injection SQL/NoSQL** : Prisma ORM (requêtes paramétrées par défaut), ValidationPipe global (`whitelist: true`, `forbidNonWhitelisted: true`), DTOs class-validator avec contraintes de type/longueur
- **XSS** : Helmet (Content-Security-Policy, X-XSS-Protection et autres headers), React échappe par défaut, validation côté client Zod
- **CSRF** : CORS restrictif (origin unique configurable via `FRONTEND_URL`), SameSite cookies (quand HttpOnly implémenté)
- **Brute force** : ThrottlerGuard global (`@nestjs/throttler`) + compteur Redis par IP/email spécifique au login
- **Broken Access Control** : Guards par rôle (USER/ADMIN/SUPERADMIN), vérification propriétaire systématique (`order.userId === userId`), SUPER_ADMIN bypass, `@CurrentUser()` decorator
- Extraits de code concrets pour chaque menace

#### 8.3 Sécurité des intégrations tierces (~0.5-1 page)
- **Stripe** :
  - Vérification signature webhook HMAC-SHA256 (`stripe.webhooks.constructEvent`)
  - Correspondance `paymentIntentId` ↔ `Order` (anti-spoofing, loggé en ERROR si mismatch)
  - Aucune donnée bancaire côté serveur (Stripe Elements côté client)
  - `@SkipThrottle()` sur le endpoint webhook
- **Cloudinary** :
  - Upload signé (HMAC-SHA1 + timestamp, clé secrète côté serveur uniquement)
  - Le backend ne touche jamais le fichier image

#### 8.4 Sécurité de l'infrastructure (~0.5 page)
- Conteneurs non-root en production (utilisateur `node`)
- `expose` au lieu de `ports` (pas d'accès direct aux containers)
- Validation Joi des variables d'env au démarrage (fail-fast si config manquante)
- Swagger désactivé en production (`NODE_ENV` check)
- Dockerfiles multi-stage : image finale minimale (Alpine)

#### 8.5 RGPD et données personnelles (~0.5 page)
- **Ce qui existe** :
  - Pages légales (mentions légales, politique de confidentialité, CGV)
  - `safeSelect` : exclusion systématique du password et tokens dans toutes les réponses API
  - Mots de passe hashés bcrypt, tokens de reset hashés
  - Pas de tracking ni analytics → pas de bannière cookies nécessaire (exemption directive ePrivacy)
  - Facturation conforme : factures PDF avec mentions légales obligatoires (CGI art. 289), TVA stockée en BDD (HT + taux + montant), numéro de facture unique
- **Ce qui est implémenté/à implémenter** :
  - Suppression de compte avec anonymisation des commandes (`DELETE /users/me`) : transaction Prisma qui anonymise les données personnelles sur les commandes (obligation comptable 6 ans) puis supprime le User
  - Export de données (`GET /users/me/export`) : JSON structuré (profil, adresses, historique commandes)
  - Boutons correspondants dans le profil utilisateur
- **Justification du pattern anonymisation vs cascade delete** : les commandes doivent être conservées pour la comptabilité, mais les données personnelles sont effacées → conforme à l'Article 17 (droit à l'effacement) tout en respectant l'obligation légale de conservation

---

### 9. Plan de tests (~3-4 pages)
> **CP9** - Préparer et exécuter les plans de tests

#### 9.1 Stratégie de tests (~0.5 page)
- **Pyramide de tests** : unitaires (base large, 41 fichiers) → pas d'intégration lourde → E2E minimal (health check)
- **Justification** : priorité aux tests de logique métier (services) et de gestion d'état (stores), couverture pragmatique adaptée au planning (4 sprints)
- **Outils** :
  - Backend : Jest 29 + ts-jest, mocking via `jest.fn()`, `Test.createTestingModule()` NestJS
  - Frontend : Vitest 4 + React Testing Library + `@testing-library/jest-dom`, `renderHook` pour les hooks
- **Organisation** : fichiers `.spec.ts` colocalisés avec le code source (back), `__tests__/` directories (front)
- **CI** : tests exécutés automatiquement sur chaque PR via workflow `pr-checks`

#### 9.2 Extraits de tests significatifs (~2-3 pages)

- **Backend — `payment.service.spec.ts`** (~0.5 page) :
  - Test de la vérification signature webhook Stripe (signature manquante → BadRequest, signature invalide → BadRequest)
  - Test du traitement `payment_intent.succeeded` : mock Stripe event → vérification que l'order passe en PAID + CONFIRMED
  - Test de la variable d'env manquante (`STRIPE_WEBHOOK_SECRET` → Error)
  - Pattern : mocking Stripe SDK + PrismaService, simulation d'événements webhook

- **Backend — `orders.service.spec.ts`** (~0.5 page) :
  - Test `findAll` : pagination (skip/take calculés), résultat vide
  - Test `findOne` : accès autorisé (userId match), NotFoundException, ForbiddenException (userId mismatch)
  - Pattern : mock des relations Prisma imbriquées (order → user → items → trees), `expect.objectContaining`

- **Frontend — `cartStore.test.ts`** (~0.5 page) :
  - Test `addItem` : ajout, incrémentation quantité, initialisation `createdAt`
  - Test `syncWithStock` : stock suffisant → pas de changement, stock insuffisant → quantité réduite, produit épuisé → supprimé
  - Test `isExpired` : panier récent → false, panier > 24h → true
  - Pattern : accès direct `useCartStore.getState()`, reset `beforeEach`

- **Frontend — Tests de composants** (à implémenter, ~0.5 page) :
  - 2-3 tests composants React avec Testing Library : rendu, interaction utilisateur, validation formulaire
  - Exemples ciblés : `CartItem` (affichage, modification quantité, suppression), `CheckoutForm` (validation Zod, soumission)
  - Pattern : `render()`, `screen.getByRole()`, `userEvent.click()`, `waitFor()`

#### 9.3 Jeu d'essai — Tunnel de vente (~1 page)
- **Fonctionnalité testée** : processus de commande complet (fil rouge du dossier)
- **Tableau de tests** :

  | Scénario | Entrée | Résultat attendu | Résultat obtenu | Statut |
  |---|---|---|---|---|
  | Ajout au panier | Arbre id=1, qty=2 | Panier contient 1 item, qty=2 | ✅ Conforme | OK |
  | Stock insuffisant au confirm | qty=10, stock=3 | `{ success: false, issues: [insufficient] }` | ✅ Conforme | OK |
  | Produit épuisé | stock=0 | `{ success: false, issues: [out-of-stock] }` | ✅ Conforme | OK |
  | Paiement réussi (webhook) | `payment_intent.succeeded` | Order → PAID + CONFIRMED | ✅ Conforme | OK |
  | Paiement échoué (webhook) | `payment_intent.payment_failed` | Stock restauré, order → FAILED | ✅ Conforme | OK |
  | Signature webhook invalide | signature incorrecte | BadRequestException | ✅ Conforme | OK |
  | Accès commande d'un autre user | userId mismatch | ForbiddenException | ✅ Conforme | OK |
  | Panier expiré (>24h) | createdAt > 24h | `isExpired()` → true | ✅ Conforme | OK |

- **Analyse** : pas d'écarts constatés sur les cas testés, couverture des cas nominaux et d'erreur

#### 9.4 Bilan et axes d'amélioration (~0.5 page)
- **Points forts** : logique métier critique testée (checkout, paiement, panier), CI intégrée, deux frameworks configurés
- **Points faibles** (honnêteté) :
  - Tests de contrôleurs backend superficiels (instanciation uniquement)
  - Pas de tests de composants React initialement (corrigé avec l'ajout de 2-3 tests)
  - Pas de tests d'intégration du flux checkout complet (mock uniquement)
  - Pas de couverture de code imposée en CI
- **Axes d'amélioration identifiés** :
  - Tests d'intégration backend avec BDD de test (fixtures, setup/teardown)
  - Seuil de couverture minimum en CI (ex: 70%)
  - Tests E2E sur les parcours critiques (Playwright)

---

### 10. Veille sécurité (~2-3 pages)
> Transversal (CP9, CP10, CP11)
> Focus sur la **démarche** (sources, processus, cycle) — les implémentations concrètes sont en §8

#### 10.1 Sources de veille (~0.5 page)
- **Tableau** : source / type / fréquence / ce qu'on en tire

  | Source | Type | Usage concret |
  |---|---|---|
  | OWASP Top 10:2025 | Référentiel menaces web | Grille d'audit sprint 3, organisation des protections |
  | npm audit | Scan dépendances | Détection vulnérabilités transitives, intégré au CI |
  | GitHub Dependabot | Alertes automatiques | Notifications sur les dépendances vulnérables du repo |
  | Stripe Security Docs | Best practices paiement | Webhooks sécurisés, PCI-DSS compliance via Elements |
  | Mozilla Observatory | Scan headers HTTP | Score sécurité des headers (Helmet), avant/après |
  | Node.js Security WG / Snyk | Advisories écosystème | Veille spécifique aux vulnérabilités Node.js/npm |

#### 10.2 Audit de sécurité — démarche (~1-1.5 page)
- **Contexte** : audit réalisé en sprint 3 à partir du référentiel OWASP Top 10:2025
- **Processus** : évaluation systématique de chaque catégorie OWASP sur le backend
  - Phase 1 : identification des vulnérabilités potentielles
  - Phase 2 : correctifs implémentés (tokens hashés, rate limiting renforcé, etc.)
  - Phase 3 : restant à faire (identifié, priorisé)
- **Démarche d'amélioration continue** : audit → corrections → re-vérification
- Référence au document `SECURITY_AUDIT_BACKEND.md`
- **Audit frontend** : identifié mais non encore réalisé, priorité = migration HttpOnly cookies

#### 10.3 Veille sur les dépendances (~0.5 page)
- **npm audit** : résultats sur backend et frontend, vulnérabilités trouvées et corrigées
  - Captures avant/après correction
  - Ajout dans le workflow CI (`npm audit --audit-level=high`)
- **Gestion du lockfile** : `package-lock.json` versionné, garantit la reproductibilité
- **Stratégie de mise à jour** : mise à jour régulière des dépendances, priorisation par niveau de gravité (critical > high > moderate)

#### 10.4 Résultats concrets (~0.5 page)
- **Scan Mozilla Observatory** : score des headers HTTP avant/après configuration Helmet (captures)
  - URL : `https://observatory.mozilla.org`
- **npm audit** : nombre de vulnérabilités trouvées, corrections appliquées
- **Exemple concret de cycle veille → action** : tokens de reset/vérification email stockés en clair → audit OWASP → hashage des tokens en BDD (correction sprint 3)
- **Posture** : l'application est quasi prod-ready, les éléments manquants sont identifiés et priorisés (cf. liste des évolutions)

---

### ANNEXES (40 pages max, hors comptage)

#### Annexe A — Maquettes Figma (~5-8 pages)
- Ensemble complet des maquettes (le corps §5.3 n'en montre que 3-4 représentatives)
- Organisées par parcours : visiteur, utilisateur authentifié, admin
- Le jury peut vérifier la cohérence maquettes ↔ produit final

#### Annexe B — Schéma Prisma complet + Dictionnaire de données (~3-5 pages)
> **CP7**
- `schema.prisma` complet (13 modèles, enums, relations) — référencé depuis §5.4
- Dictionnaire de données pour les entités clés : Order (+ OrderItem, OrderAddress), User, Tree (+ TreeStock)
- Colonnes : nom du champ, type, contraintes, description

#### Annexe C — Extraits de code complémentaires (~8-12 pages)
> **CP2, CP3, CP8**
- Services complets dont le corps (§7) ne montre que des extraits ciblés :
  - `checkout.service.ts` : flux complet init + confirm
  - `payment.service.ts` : gestion complète des 3 événements webhook
  - `invoice.service.ts` : génération PDF complète
  - `orders-cleanup.service.ts` : cron de réconciliation
  - `cartStore.ts` + `useCartProducts.tsx` : gestion d'état frontend complète
- Le jury peut parcourir le code intégral s'il le souhaite

#### Annexe D — CI/CD et déploiement (~3-5 pages)
> **CP10, CP11** - Préparer et documenter le déploiement, contribuer à la mise en production
- Les 3 workflows GitHub Actions complets (`pr-checks`, `develop-ci`, `production-deploy`)
- `compose.prod.yml` + Dockerfiles prod (backend multi-stage + frontend Nginx)
- `Makefile` : liste des commandes documentées
- ⚠️ Procédure Coolify/Hetzner : à compléter selon la maîtrise du candidat avant le passage

#### Annexe E — Captures d'écran de l'application (~5-8 pages)
- **Parcours utilisateur** : catalogue → fiche arbre → panier → checkout → confirmation de commande
- **Back-office admin** : CRUD arbres, gestion commandes, changement de statut
- **Facture PDF** : exemple de rendu généré
- **Responsive** : comparaison mobile vs desktop sur 2-3 pages clés
- **Scores Lighthouse** : performance 95+, accessibilité 100, SEO 100

#### Annexe F — Résultats de tests et audits (~3-5 pages)
> **CP9**
- Résultats d'exécution des tests : output Jest (backend) + Vitest (frontend)
- Rapports npm audit (avant/après corrections)
- Scan Mozilla Observatory (captures avec score headers HTTP)
- Score Lighthouse détaillé (captures)

---

## Estimation de pages

### Corps du dossier (40-60 pages)

| Section | Pages estimées |
|---------|---------------|
| 1. Liste des compétences | ~1 |
| 2. Cahier des charges | ~3-4 |
| 3. Contexte du projet | ~1 |
| 4. Gestion de projet | ~3-4 |
| 5. Spécifications fonctionnelles | ~8-10 |
| 6. Spécifications techniques | ~3-4 |
| 7. Réalisations (code) | ~10-12 |
| 8. Sécurité | ~3-4 |
| 9. Plan de tests | ~3-4 |
| 10. Veille sécurité | ~2-3 |
| **TOTAL corps** | **~38-47 pages** |

### Annexes (40 pages max)

| Annexe | Pages estimées |
|--------|---------------|
| A. Maquettes Figma | ~5-8 |
| B. Schéma Prisma + Dictionnaire | ~3-5 |
| C. Extraits de code complémentaires | ~8-12 |
| D. CI/CD et déploiement | ~3-5 |
| E. Captures d'écran | ~5-8 |
| F. Résultats tests et audits | ~3-5 |
| **TOTAL annexes** | **~27-43 pages** |
