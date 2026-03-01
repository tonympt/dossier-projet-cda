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

### ✅ Section 3 — Contexte du projet (~0,5 page)
- Adaptation de "présentation de l'entreprise" pour un projet de formation
- Contenu : organisme de formation, GreenRoots projet fictif, rôle du candidat (Lead Dev Front + contribution étendue)
- **Hors de cette section** : organisation de l'équipe et pilotage → section 4
- Note : Tony a touché à tout malgré le rôle Lead Dev Front — à valoriser dans la rédaction

### ✅ Section 4 — Gestion de projet (~3-4 pages)
- Méthodologie Agile/Scrum, 4 sprints d'1 semaine (Sprint 0 à Sprint 3)
- Planning/suivi : Trello Kanban avec captures disponibles
- Cérémonies : daily meetings, rétrospectives, définition des objectifs
- Qualité : conventions communes documentées, guidelines frontend (Atomic Design), PRs + code review obligatoire, ESLint + Prettier
- Git flow : stratégie de branches + PRs focalisées, pas de merge sans review
- Risques : définis dans le CDC (section 2), simple référence ici — pas de suivi formel pendant le projet

### ✅ Section 5.1 — Contraintes du projet et livrables attendus
- **Contraintes temporelles/humaines** : 4 semaines, équipe de 5, rôles définis
- **Contraintes techniques** : Docker obligatoire, TypeScript front+back, responsive, compatibilité navigateurs (2 dernières versions majeures Chrome/Firefox/Edge/Safari)
- **Contraintes réglementaires** : RGPD (mentions légales, confidentialité, suppression compte), RGAA (accessibilité)
- **Contraintes fonctionnelles** : MVP borné au CDC (tunnel simulé initialement, Stripe intégré au final), distinction claire MVP vs évolutions futures
- **Livrables** : application MVP déployée, documentation technique (README, Swagger, conventions, guidelines), code source versionné sur GitHub
- Note : le dossier de projet CDA n'est pas un livrable du projet en tant que tel

### ✅ Section 5.2 — Architecture logicielle du projet (CP6, ~2-3 pages)
- **Architecture multicouche 3 tiers** : SPA React → API REST NestJS → PostgreSQL + Redis, Nginx en reverse proxy
- **Justification des choix techniques** : pourquoi React/Vite, NestJS, PostgreSQL/Prisma, Redis (bref, détail en §6)
- **Architecture modulaire backend** : Controller → Service → Repository → Prisma, injection de dépendances
- **Design patterns** : DI, Repository, Module, Guards/Decorators, State machine (backend) ; Atomic Design, Feature-based, Store pattern (frontend)
- **Diagramme d'architecture** : à créer en Mermaid
- **Sécurité dans l'architecture** : sécurité à chaque couche (validation client → validation API → ORM paramétré), référence OWASP
- **Éco-conception** : choix architecturaux (Vite, Tailwind purge, shadcn sélectif, Cloudinary, code splitting, Redis cache prévu), cohérence avec la mission GreenRoots, référentiels (WSG, RGESN, RWEB)
- **Conteneurisation** : Docker Compose (8 services) comme partie de l'architecture — détail en §6

### ✅ Section 5.3 — Maquettes et enchaînement des maquettes (CP5, ~2-3 pages)
- **Wireframes / Zoning** : zoning des pages clés (landing, catalogue, fiche arbre, checkout) — à créer
- **Maquettes Figma** : 3-4 captures représentatives dans le corps, ensemble complet en Annexe A
- **Arborescence du site** : diagramme Mermaid (pages publiques / authentifiées / admin)
- **Enchaînement des écrans** : 1-2 flowcharts Mermaid (parcours d'achat, authentification)
- **Pas de charte graphique ici** — traitée dans les sections techniques
- **Pas de section RGAA dédiée ici** — l'accessibilité est assurée par le choix de shadcn/ui (Radix UI, accessible par défaut), argument documenté en §5.2 (justification des choix techniques). Détail implémentation en §7.1, scores Lighthouse en §7.1
- **Atomic Design** : c'est une architecture frontend, pas du design visuel — traité en §5.2 (design patterns) et §7.1 (extraits de code)

### ✅ Section 5.4 — Modèle de données (CP7, ~2-3 pages)
- **MCD** : diagramme entités-relations des 13 entités avec cardinalités — à créer
- **MPD** : extrait commenté du schéma Prisma (Order/OrderItem, User), conventions `@map`/`@@map`, complet en annexe
- **Types remarquables** : prix en INT (centimes, standard Stripe), enums pour machines à états, PostGIS pour géolocalisation
- **Contraintes d'intégrité** : FK, unicité, enums, cascade vs protection
- **Droits d'accès** : choix applicatif (guards NestJS) vs BDD (rôles PostgreSQL) — justifié (Prisma connexion unique, pattern moderne API REST)
- **Sécurité données** : bcrypt, tokens hashés, select sécurisé, RGPD
- **Jeu d'essai** : seed Prisma + migrations versionnées (25) + procédure backup/restore (pg_dump à documenter)
- **Annexes** : dictionnaire de données (2-3 entités), index Prisma à implémenter
- **Points forts identifiés** : prix en INT, PostGIS, transactions atomiques, pessimistic locking, 25 migrations
- **Points à combler** : MCD à créer (obligatoire), procédure pg_dump à documenter, index à ajouter

### ✅ Section 5.5 — Diagramme de cas d'utilisation (CP5, ~1 page)
- Diagramme UML en Mermaid, 3 acteurs avec héritage d'accès (Visiteur → Utilisateur → Admin)
- Relations `<<include>>`/`<<extend>>` sur les cas pertinents (vérifier stock, rembourser Stripe)
- Héritage = hiérarchie d'accès visuelle, dans le code c'est les guards NestJS (pas d'héritage POO)
- Brève description textuelle sous le diagramme

### ✅ Section 5.6 — Diagrammes de séquence (CP5/CP6, ~1-2 pages)
- **Diagramme 1 : Processus de commande** — 5 acteurs (User→Frontend→Backend→PostgreSQL→Stripe), flux complet checkout avec webhook, transaction atomique
- **Diagramme 2 : Authentification** — 5 acteurs (User→Frontend→Backend→PostgreSQL→Redis), login (brute force) + logout (blacklist)
- Les deux en Mermaid sequenceDiagram avec légende
- Le diagramme auth sera adapté selon l'implémentation HttpOnly cookies
- Stratégie : checkout et auth en 5.6 (haut niveau), le code détaillé viendra en §7.2/7.3/8 (angles différents)

---

### ✅ Section 5 complète — Spécifications fonctionnelles
Toutes les sous-sections (5.1 à 5.6) sont validées.

### ✅ Section 6 — Spécifications techniques (CP1, CP6, ~3-4 pages)
- **6.1 Stack technique** : tableau récapitulatif techno/version/rôle/justification, toute la stack détaillée (React 19, NestJS 11, PostgreSQL 16/PostGIS, Prisma 7, Redis, Nginx, Stripe, Cloudinary, Resend)
- **6.2 Environnement de développement et outillage** (CP1) :
  - Docker Compose dev : 7 services (frontend, backend, database, redis, nginx, stripe-cli, prisma-studio), healthchecks partout
  - Docker Compose prod : 3 services épurés (expose, réseau Coolify, pas de ports exposés)
  - Makefile : ~20 commandes documentées, standardise le workflow équipe
  - Dockerfiles multi-stage : build → run avec utilisateur non-root en prod, gzip + cache 1 an en frontend
  - Variables d'environnement : .env.example + .env.production.example + validation Joi au démarrage
  - Indépendant de l'IDE
- **6.3 Stratégie de sécurité** :
  - Réseau : Helmet, CORS restrictif, expose (pas ports) en prod, réseau Docker isolé
  - Application : ValidationPipe (whitelist), ThrottlerGuard, Guards rôles, JWT (→ HttpOnly), bcrypt, brute force Redis, JWT blacklist
  - Données : Prisma paramétré, transactions pessimiste, Joi fail-fast, non-root containers
  - Audit OWASP Top 10:2025 (phases 1-2 backend OK), Stripe webhook secret
- **6.4 Documentation et qualité** : 3 workflows GitHub Actions (pr-checks, develop-ci, production-deploy + health check), ESLint, Swagger (dev only), conventions partagées, Prisma Studio
- **Observations notables** : Makefile = atout DX, différence dev/prod bien marquée, Joi fail-fast, vite.config.ts avec manual chunks + Critters
- **⚠️ Déploiement prod (Coolify/Hetzner)** : pas géré par le candidat — mis entre parenthèses, à approfondir avant rédaction §10/Annexes F-G

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
  - NoSQL : Redis brute force login + JWT blacklist
- **7.4 Autres composants** :
  - Stripe module (factory global singleton), Cloudinary upload signé (backend signature → frontend upload direct), Guards+Decorators RBAC

### ✅ Section 8 — Éléments de sécurité (~3-4 pages)
- **8.1 Authentification** : JWT, bcrypt, tokens hashés, blacklist Redis, évolution HttpOnly
- **8.2 Attaques courantes** : organisé par menace (injection, XSS, CSRF, brute force, broken access control) avec extraits de code
- **8.3 Intégrations tierces** : Stripe (webhook signature, anti-spoofing, pas de données carte), Cloudinary (upload signé)
- **8.4 Infrastructure** : non-root, expose vs ports, Joi fail-fast, Swagger désactivé prod
- **8.5 RGPD** : ce qui existe (pages légales, safeSelect, bcrypt) + évolutions à implémenter (suppression compte avec anonymisation, export données JSON)
- **Décision** : pas de bannière cookies (pas de tracking/analytics, cookies fonctionnels exemptés)
- **Décision** : OWASP comme cadre d'organisation en §8 (implémentations) vs §10 (démarche de veille)

### ✅ Section 9 — Plan de tests (CP9, ~3-4 pages)
- **9.1 Stratégie** : pyramide de tests, 41 fichiers (35 back + 6 front), Jest + Vitest/Testing Library, CI sur PR
- **9.2 Extraits significatifs** : payment.service.spec (webhook), orders.service.spec (accès/forbidden), cartStore.test (sync stock, expiration) + tests composants React à ajouter (CartItem, CheckoutForm)
- **9.3 Jeu d'essai** : tableau tunnel de vente (8 scénarios : nominal + erreurs stock + webhook + auth + expiration)
- **9.4 Bilan honnête** : points forts (métier testé, CI) + faiblesses (controllers superficiels, pas de composants React initialement, pas de couverture CI)
- **Décision** : pas de E2E, ajouter 2-3 tests composants React (~2-3h) pour compléter la couverture CP2+CP9
- **Inventaire tests existants** : backend bien couvert sur services métier (orders, cart, payment, trees), front couvert sur store + utilitaires + hooks

### ✅ Section 10 — Veille sécurité (~2-3 pages)
- **10.1 Sources de veille** : tableau (OWASP, npm audit, Dependabot, Stripe docs, Mozilla Observatory, Snyk)
- **10.2 Audit sécurité démarche** : processus OWASP Top 10:2025 en sprint 3, phases 1-2 réalisées, amélioration continue
- **10.3 Veille dépendances** : npm audit (résultats + ajout CI), lockfile, stratégie de mise à jour
- **10.4 Résultats concrets** : scan Mozilla Observatory, npm audit, cycle veille→action (tokens hashés)
- **Décision** : §8 = implémentations, §10 = démarche/processus — pas de redite
- **Posture dossier** : "quasi prod-ready" honnête avec liste des éléments manquants identifiés et priorisés

## Liste des évolutions à implémenter avant le passage (~2.5-3 jours)
Voir `docs/CONSIGNES-REDACTION.md` pour le détail complet avec priorités.
- ~~P1 : Email confirmation commande~~ ✅ FAIT (branche `feature/order-confirmation-email`, mergée dans develop)
- ~~P1 : npm audit + CI~~ ✅ FAIT (branche `feature/npm-audit-ci`, audit fix + `npm audit --audit-level=critical` dans pr-checks)
- ~~P2 : RGPD suppression+anonymisation~~ ✅ FAIT (branche `feature/rgpd-account-deletion`, DELETE /auth/me + transaction anonymisation + frontend DeleteAccountSection)
- ~~P2 : RGPD export~~ ✅ FAIT (branche `feature/rgpd-data-export`, GET /auth/me/export + frontend ExportDataSection)
- P3 : JWT HttpOnly (~4-6h), tests composants React (~2-3h)
- P4 : Redis cache API (~3-4h)

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
- **JWT HttpOnly cookies** : évolution prévue pour renforcer la sécurité (à implémenter puis documenter en §8)
- **Redis cache API** : évolution prévue au-delà du brute force/blacklist actuels (routes pays, catégories, catalogue)

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
