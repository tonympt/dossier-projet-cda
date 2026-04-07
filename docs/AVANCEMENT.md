# État d'avancement - Construction du plan du dossier

## Où on en est

On est en train de définir le plan du dossier **section par section**, en discutant du contenu et des grandes lignes avant de rédiger. Les décisions sont validées ensemble au fil de la conversation.

## Sections validées

### ✅ Section 1 — Liste des compétences mises en œuvre
- Forme : tableau simple (CP / intitulé / couverte)
- Une demi-page max, sobre

### ✅ Section 2 — Cahier des charges / Expression des besoins
- Ce que le projet *demandait* (côté produit, pas technique)
- Contenu : présentation du projet, objectifs, cible utilisateurs, acteurs (3 rôles), périmètre MVP vs évolutions futures, user stories principales
- **Hors de cette section** : contraintes et livrables → section 5 / risques → section 4

### ✅ Section 3 — Contexte du projet (~0,5 page) — RÉDIGÉE
- Formation CDA chez O'clock, équipe de 5 apprenants, projet de fin de formation
- Thème imposé (e-commerce d'arbres), liberté sur la stack et les choix techniques
- Rôle initial Lead Dev Front au sprint 0, élargi au backend, conteneurisation et DevOps
- **Décisions rédaction** : O'clock nommé explicitement, pas de mention de l'aide aux camarades (gardé pour §4)

### ✅ Section 4 — Gestion de projet (~3-4 pages) — RÉDIGÉE
- **4.1 Méthodologie et organisation** : Scrum adapté, rôles Scrum (PO, SM) + rôles techniques (Lead Dev Front/Back, Full-stack), Tony a pris le leadership PO en pratique
- **4.2 Déroulement des sprints** : Sprint 0 (conception) → Sprint 1 (fondations) → Sprint 2 (MVP) → Sprint 3 (qualité/sécurité)
- **4.3 Outils et qualité** : GitFlow détaillé ici (main/dev/feature), conventions communes, code review, ESLint+Prettier
- **4.4 Bilan** : investissement inégal formulé positivement, sprints plus longs avec le recul
- **Décisions rédaction** : daily meetings en visio 9h ~15min, 1 capture Trello milieu/fin de sprint, pas de noms des membres, risques par référence au CDC

### ✅ Section 5.1 — Contraintes du projet et livrables attendus — RÉDIGÉE
- Contraintes temporelles (4 semaines, 5 personnes), techniques (Docker, TypeScript, responsive), réglementaires (RGPD sprint 3, RGAA via shadcn/ui), fonctionnelles (MVP + Stripe imposé par la logique e-commerce)
- Stripe : PaymentElement React communique directement avec Stripe, données bancaires ne transitent jamais par l'API
- Biblis "à l'étude" du CDC (TanStack, Zustand, Zod) toutes adoptées
- **Décisions rédaction** : Docker = choix d'équipe, TypeScript = standard non questionné, RGPD = initiative de Tony

### ✅ Section 5.2 — Architecture logicielle du projet (CP6, ~2-3 pages) — RÉDIGÉE
- Architecture 3 tiers : couche présentation (Nginx + SPA React) → couche métier (API NestJS) → couche données (PostgreSQL + Redis)
- Architecture modulaire NestJS (pas hexagonale) : Controller → Service → DTOs → Prisma (repository implicite)
- **Diagramme d'architecture créé** : `diagrammes/architecture-globale.md` — format paysage, 3 couches + services externes
- Justification choix techniques (React+Vite, NestJS, PostgreSQL+Prisma, Redis) — bref, détail en §6
- Conteneurisation Docker Compose (7 services dev, 3 prod) — bref, détail en §6
- Sécurité à chaque couche (bref, détail en §8)
- Éco-conception (Vite, Tailwind purge, shadcn sélectif, Redis cache) — cohérence mission reforestation
- **Décisions rédaction** : DPs retirés du texte (une phrase suffit), Spring Boot mentionné pour l'injection de dépendances

### ✅ Section 5.3 — Maquettes et enchaînement des écrans (CP5, ~2-3 pages) — RÉDIGÉE
- **Identité visuelle** : Montserrat hébergée local, palette verte, contrastes accessibilité validés Lighthouse
- **Wireframes** : desktop + mobile page catalogue (créés sur Balsamiq/Figma), pas de zoning séparé
- **Maquette Figma** : 1 capture page catalogue dans le corps, pas d'annexe Figma complète
- **Arborescence** : `diagrammes/arborescence-site.md` — arbre hiérarchique avec couleurs par niveau d'accès (public/auth/checkout/admin)
- **Parcours d'achat** : `diagrammes/parcours-achat.md` — flowchart horizontal simplifié (6 écrans, pas de détail technique)
- **Décisions rédaction** : pas de zoning, pas de maquettes tablette, pas d'annexe Figma, wireframe et maquette sur même page (catalogue)

### ✅ Section 5.4 — Modèle de données (CP7, ~2-3 pages) — RÉDIGÉE
- **ERD complet** : `diagrammes/erd-modele-donnees.md` — 13 entités, tous attributs, PK/FK/UK, format paysage
- 3 domaines : catalogue (Tree, Category, Location, Country, TreeStock), commandes (Order, OrderItem, OrderAddress), utilisateurs (User, Role, Cart, CartItem)
- Cart/CartItem en BDD mais non utilisés dans le MVP (panier côté client Zustand), mentionné dans le texte
- Choix notables : prix en INT (centimes, standard Stripe), enums (PaymentStatus, OrderStatus), PostGIS, séparation Tree/TreeStock
- Snapshot commandes : OrderAddress.countryCode = string sans FK (intégrité historique)
- Contraintes d'intégrité, sécurité données (bcrypt, tokens hashés, safeSelect), RGPD (anonymisation + export)
- Droits applicatifs (guards NestJS) vs BDD — justifié
- **Décisions rédaction** : enum ≠ state machine (précisé), ERD complet plutôt qu'allégé

### ✅ Section 5.5 — Diagramme de cas d'utilisation (CP5, ~1 page) — RÉDIGÉE
- **Diagramme** : `diagrammes/cas-utilisation.md` — Mermaid graph LR, 3 acteurs, 18 use cases, couleurs par niveau d'accès
- 3 acteurs avec héritage : Visiteur → Utilisateur → Admin
- Relations include (vérifier stock, payer Stripe) et extend (rembourser Stripe)
- Texte précise les use cases englobants (gérer profil = modif infos + modif mdp, gérer commandes = consultation + statut + annulation)
- Version agrandie en annexe pour lisibilité
- **Décisions rédaction** : Mermaid plutôt que PlantUML, pas de diagramme UML natif dans Mermaid

### ✅ Section 5.6 — Diagrammes de séquence et d'états (CP5/CP6, ~2-3 pages) — RÉDIGÉE
- **Diagramme 1 : Processus de commande** (`diagrammes/sequence-checkout.md`) — 5 acteurs, 5 phases (init sans stock → confirm transaction atomique → paiement direct Stripe → webhook HMAC → polling)
- **Diagramme 2 : Authentification** (`diagrammes/sequence-authentification.md`) — 5 acteurs, version compacte (login brute force Redis + cookie HttpOnly + logout blacklist)
- **Diagramme 3 : Cycle de vie commande** (`diagrammes/etats-commande.md`) — stateDiagram-v2, deux machines parallèles OrderStatus + PaymentStatus
- Diagrammes en version réduite dans le corps + version agrandie en annexe
- Texte d'intro développé : justification des deux flux choisis (métier + sécurité), renvoi §7.2/§8 pour les détails
- **Décisions rédaction** : diagrammes simplifiés (flux nominal uniquement, sans alt/erreurs) pour lisibilité Word, checkout en 2 étapes justifié dans le texte (pas de blocage stock inutile), webhook = source de vérité expliqué

---

### ✅ Section 5 complète — Spécifications fonctionnelles
Toutes les sous-sections (5.1 à 5.6) sont validées.

### ✅ Section 6 — Spécifications techniques (CP1, CP6, CP11, ~3-4 pages) — RÉDIGÉE
- **6.1 Stack technique** : tableau 15 technos (techno/version/rôle/justification), intro sur le critère de sélection (maîtrise équipe, coût d'apprentissage), paragraphes frontend (initiative Lead Dev Front), NestJS vs Express, PostgreSQL+Prisma, Stripe (PSP complet, PCI-DSS), Cloudinary (analyse solutions, upload signé)
- **6.2 Environnement de développement et outillage** (CP1) : tableau 7 services Docker Compose dev, différence dev/prod (7 vs 3 services, expose vs ports), healthchecks + depends_on, Makefile (~20 commandes), Dockerfiles multi-stage (build→run, non-root), variables d'environnement + validation Joi fail-fast
- **6.3 Stratégie de sécurité** (CP6) : organisée par couches (défense en profondeur), réseau (Helmet, CORS), application (ValidationPipe whitelist, ThrottlerGuard), données (Prisma paramétré), conformité OWASP Top 10:2025 (audit sprint 3), renvoi §8 pour le détail
- **6.4 Documentation et qualité** (CP11) : tableau 3 workflows GitHub Actions (pr-checks, develop-ci avec BDD, production-deploy), ESLint+Prettier en CI, Husky non implémenté → évolution prévue pre-commit, Swagger désactivé en prod (sécurité), conventions + code review obligatoire
- **Décisions rédaction** : pas de mention ANSSI explicite sur les mesures (seulement défense en profondeur), Husky honnêtement traité (pas installé, CI joue le rôle, évolution prévue), Coolify mentionné brièvement (pas géré par le candidat), Cloudinary sans argument CDN (pas pertinent pour le contexte)

---

## Section en cours

### ✅ Section 5.6 — Enrichie : ajout diagramme d'états (stateDiagram)
- Diagramme 3 ajouté : cycle de vie commande (OrderStatus + PaymentStatus en parallèle)
- Fait le pont entre la conception (§5) et l'implémentation de la state machine (§7.2)

### ✅ Section 7 — Réalisations / extraits de code (CP2, CP3, CP8, ~10-12 pages)
- **Fil rouge** : tunnel de vente (panier → checkout → confirmation) + back-office admin en complément
- **7.1 Interfaces utilisateur (CP2)** :
  - Panier : Zustand persist + syncWithStock + TanStack Query polling + dialog conflits stock
  - Checkout : machine à états PaymentState, formulaire Zod/RHF verrouillé, Stripe Elements lazy + double confirmation
  - OrderConfirmation : polling adaptatif 6 états, grace period webhook
  - Admin : OrdersTable responsive adaptatif (table→cards via useMediaQuery), TreeForm + ImageUpload Cloudinary, guards frontend beforeLoad
  - Patterns transversaux : Atomic Design, skeleton loading, accessibilité ARIA, responsive
- **7.2 Composants métier (CP3)** :
  - confirmCheckout : vérification propriétaire, idempotence, validation stock, transaction atomique, retour structuré issues
  - initCheckout : réutilisation commande PENDING, gestion PaymentIntent (update/create)
  - handleWebhook : signature HMAC, anti-spoofing PI, idempotence, 3 événements, @SkipThrottle
  - OrdersCleanupService : cron horaire, 2 cas (abandonnées + webhooks perdus), réconciliation Stripe API
  - updateOrderStatus : state machine + cas CANCELLED (refund + restauration stock)
- **7.3 Accès aux données (CP8)** :
  - SQL : transactions Prisma (confirmCheckout, cancelOrder), requêtes batch avec include imbriqué, pagination parallèle
  - NoSQL : Redis cache-aside (countries TTL 24h, categories TTL 1h + invalidation sur écriture), brute force login, JWT blacklist
- **7.4 Autres composants** :
  - Stripe module (factory global singleton), Cloudinary upload signé (backend signature → frontend upload direct), Guards+Decorators RBAC

### ✅ Section 8 — Éléments de sécurité (~3-4 pages) — RÉDIGÉE
- **8.1 Authentification** : migration JWT HttpOnly comme fil conducteur (cycle audit→correction), bcrypt, tokens hashés SHA-256, blacklist Redis TTL, vérification email
- **8.2 Attaques courantes** : organisé par menace OWASP (injection, XSS, CSRF, brute force, broken access control) — 100% textuel, renvois vers §7
- **8.3 Intégrations tierces** : Stripe (HMAC + anti-spoofing + données carte jamais sur serveur) + Cloudinary (upload signé, backend ne touche pas l'image) — même principe de signature
- **8.4 Infrastructure** : non-root, expose vs ports, Joi fail-fast, Swagger désactivé prod, multi-stage Alpine — renvoi §6.2
- **8.5 RGPD** : base MVP (pages légales, safeSelect, pas de bannière cookies) + sprint 3 (anonymisation Art.17 + obligation comptable 6 ans, export JSON) + facturation conforme CGI art.289
- **Décisions rédaction** : zéro code dans §8 (renvois §7 et §6), OWASP comme cadre d'organisation, pas de bannière cookies (cookies fonctionnels exemptés), détail technique anonymisation réservé à l'oral (SUJETS-ORAL.md)

### ✅ Section 9 — Plan de tests (CP9, ~3 pages) — RÉDIGÉE
- **9.1 Stratégie** : pyramide de tests, priorité logique métier critique, Jest (back) + Vitest/Testing Library (front), CI pr-checks
- **9.2 Tests significatifs** : tableau récapitulatif des fichiers (35 back / 8 front), 3 approches expliquées textuellement (mocking services, store Zustand, composants Testing Library) — pas de code
- **9.3 Jeu d'essai** : tableau tunnel de vente (8 scénarios : nominal + erreurs stock + webhook + auth + expiration)
- **9.4 Bilan honnête** : points forts (métier testé, CI) + faiblesses (controllers superficiels, pas de composants React initialement, pas de couverture CI)
- **Décision** : pas de E2E, ajouter 2-3 tests composants React (~2-3h) pour compléter la couverture CP2+CP9
- **Inventaire tests existants** : backend bien couvert sur services métier (orders, cart, payment, trees), front couvert sur store + utilitaires + hooks

### ✅ Section 10 — Veille technologique et sécurité (~2-2.5 pages) — RÉDIGÉE
- **10.1 Démarche de veille** : veille continue personnelle (Korben, DailyDev, podcasts) + veille projet (docs, articles comparatifs). Choix stack front valorisés : Zustand vs Redux/Context (benchmarks), shadcn/ui vs MUI/Chakra (légèreté, éco-conception), TanStack (maturité confirmée par veille)
- **10.2 Audit OWASP** : récit authentique (podcast → Top 10:2025 → audit structuré front/back avec IA → priorisation coût/risque → corrections), renvois §8
- **10.3 Résultats concrets** : cycles veille→action (HttpOnly, tokens hashés), npm audit en CI (critical bloquant, vulnérabilités transitives documentées), Mozilla Observatory identifié (pas scanné, reformulé honnêtement), posture quasi prod-ready
- **Décisions rédaction** : 3 sous-sections (fusionné vs 4 initiales), pas de tableau catalogue d'outils, récit authentique plutôt que liste, §8 = implémentations / §10 = démarche

## Liste des évolutions implémentées avant le passage
Voir `docs/CONSIGNES-REDACTION.md` pour le détail complet avec priorités.
- ~~P1 : Email confirmation commande~~ ✅ FAIT (branche `feature/order-confirmation-email`, mergée dans develop)
- ~~P1 : npm audit + CI~~ ✅ FAIT (branche `feature/npm-audit-ci`, audit fix + `npm audit --audit-level=critical` dans pr-checks)
- ~~P2 : RGPD suppression+anonymisation~~ ✅ FAIT (branche `feature/rgpd-account-deletion`, DELETE /auth/me + transaction anonymisation + frontend DeleteAccountSection)
- ~~P2 : RGPD export~~ ✅ FAIT (branche `feature/rgpd-data-export`, GET /auth/me/export + frontend ExportDataSection)
- ~~P3 : JWT HttpOnly~~ ✅ FAIT (commit `45b8cbb`, migration localStorage → cookies HttpOnly+Secure+SameSite=Lax, cookie-parser backend, withCredentials frontend, suppression accessToken du store)
- ~~P3 : tests composants React~~ ✅ FAIT (PR #3, LoadingSkeleton 4 tests + QuantityInput 7 tests, fix vitest jsdom config, 8/8 suites 60/60 tests)
- ~~P4 : Redis cache API~~ ✅ FAIT (PR #2, cache-aside pattern : countries billing/planting TTL 24h, categories list/detail TTL 1h avec invalidation sur write, graceful degradation si Redis down)

### ✅ Annexes — Structure validée (~27-43 pages, max 40)
- **A. Maquettes Figma** (~5-8p) : ensemble complet, par parcours
- **B. Schéma Prisma + Dictionnaire** (~3-5p) : schema.prisma complet + dictionnaire de données (Order, User, Tree)
- **C. Extraits de code complémentaires** (~8-12p) : services complets (checkout, payment, invoice, cleanup, cartStore)
- **D. CI/CD et déploiement** (~3-5p) : 3 workflows, compose.prod, Dockerfiles, Makefile, ⚠️ procédure Coolify à compléter
- **E. Captures d'écran** (~5-8p) : parcours utilisateur, admin, facture PDF, responsive, Lighthouse
- **F. Résultats tests et audits** (~3-5p) : output Jest/Vitest, npm audit, Mozilla Observatory, Lighthouse

---

## ✅ PLAN COMPLET VALIDÉ

Toutes les sections (1-10) et les annexes (A-F) ont leur contenu détaillé défini.
Prochaine étape : **rédaction** du dossier section par section + **implémentation** des évolutions listées dans `docs/CONSIGNES-REDACTION.md`.
- Section 8 — Éléments de sécurité
- Section 9 — Plan de tests (CP9)
- Section 10 — Veille sécurité
- Annexes

## Décisions structurantes prises
- On suit le **plan "projet en entreprise"** du référentiel (adapté au contexte formation)
- **Section 2 ne chevauche pas la section 5** : le CDC = les besoins exprimés, les spécifications = la réponse technique
- Les **risques** sont mentionnés en section 4 par référence au CDC, pas développés
- Les **contraintes** sont en section 5, pas en section 2
- **Pas de DoD formel** — le code review obligatoire avant merge joue ce rôle en pratique
- **Éco-conception** : fil conducteur à évoquer dans plusieurs sections (5.2, 7.1, et potentiellement §2 pour la cohérence mission/éco)
- **JWT HttpOnly cookies** : migration réalisée — cookies HttpOnly+Secure+SameSite=Lax, documenté en §8 et new-features/05
- **Redis cache API** : implémenté — cache-aside countries/categories, documenté en §7.3 et new-features/06
- **Tests composants React** : implémentés — LoadingSkeleton + QuantityInput (11 tests), documenté en §9 et new-features/07

## Ressources disponibles
- Code source complet dans `projet-greenroots/`
- Schéma BDD : `projet-greenroots/app/backend/prisma/schema.prisma`
- Guidelines frontend : `guidelines-frontend.md`
- Conventions communes : `conventions-communes.txt`
- CDC initial : `Cahier des charges GreenRoots.txt`
- Référentiel officiel : `Référentiel Activités Compétences.pdf`
- Captures Trello : disponibles (à intégrer en section 4)
- Audits sécurité : `SECURITY_AUDIT_BACKEND.md`, `SECURITY_AUDIT_FRONTEND.md`
- Audit éco-conception : `ECO-CONCEPTION-AUDIT.md`
- Performance : `PERFORMANCE_CODE_SPLITTING.md`
