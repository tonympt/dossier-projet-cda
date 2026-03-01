# Référentiel des Compétences Professionnelles - CDA

## Vue synoptique

### AT1 - Développer une application sécurisée

#### CP1 - Installer et configurer son environnement de travail en fonction du projet

**Ce qu'il faut démontrer :**
- Avoir installé et configuré les outils de développement nécessaires au projet (IDE, SDK, extensions…)
- Avoir mis en place un outil de gestion de versions (Git, GitHub/GitLab…)
- Avoir utilisé des conteneurs (Docker) pour reproduire un environnement de développement cohérent avec la production
- Avoir su lire de la documentation technique en français ou en anglais

**Critères d'évaluation (jury) :**
- Les outils de développement nécessaires sont installés
- Les outils de gestion des versions et de collaboration sont installés
- Les conteneurs implémentent les services requis
- La documentation technique est comprise (FR/EN niveau B1)

**Mots-clés :** IDE, Git, Docker, docker-compose, variables d'environnement, fichiers de config

**Dans GreenRoots :** Docker Compose (8 services : frontend, backend, PostgreSQL, Redis, Nginx, Stripe CLI, Prisma Studio), Git/GitHub, VS Code, Node.js 22, variables d'environnement par environnement (dev/prod)

---

#### CP2 - Développer des interfaces utilisateur

**Ce qu'il faut démontrer :**
- Avoir développé des interfaces conformes à une maquette/charte graphique
- Avoir rendu les interfaces responsives (adaptées à différents supports)
- Avoir utilisé des composants graphiques (bibliothèques UI, frameworks front)
- Avoir géré les événements, utilisé des appels asynchrones (AJAX, fetch, async/await)
- Avoir documenté le code et réalisé des tests unitaires sur les composants UI
- Avoir pris en compte la sécurité côté client (validation des entrées, XSS…)

**Critères d'évaluation (jury) :**
- L'interface est conforme au dossier de conception
- L'interface s'adapte au type d'utilisation (responsive)
- La charte graphique est respectée
- La réglementation en vigueur est respectée (RGAA)
- Le code est documenté
- Les tests unitaires sont réalisés
- Le jeu d'essai fonctionnel est complet
- Les tests de sécurité sont réalisés
- Veille technologique sur les interfaces utilisateur

**Mots-clés :** HTML/CSS, JavaScript/TypeScript, framework front (React…), responsive design, RGPD, accessibilité, tests unitaires (Vitest…)

**Dans GreenRoots :** React 19 + Vite + TanStack Router, Tailwind CSS + shadcn/ui (Atomic Design : atoms/molecules/organisms), responsive, TanStack Query (appels asynchrones), Vitest + Testing Library (tests composants : LoadingSkeleton, QuantityInput — 11 tests couvrant rendu, interactions, accessibilité ARIA), validation Zod côté client, mentions légales RGPD

---

#### CP3 - Développer des composants métier

**Ce qu'il faut démontrer :**
- Avoir développé la logique métier côté serveur en orienté objet, avec un style défensif
- Avoir géré la sécurité applicative (authentification, autorisation, validation des entrées)
- Avoir utilisé des composants d'accès aux données (ORM, repositories…)
- Avoir réalisé des tests unitaires et des tests de sécurité sur les composants
- Avoir documenté le code

**Critères d'évaluation (jury) :**
- Les bonnes pratiques POO sont respectées
- Les composants serveurs sont sécurisés
- Les règles de nommage sont conformes
- Le code source est documenté
- Les tests unitaires sont réalisés
- Les tests de sécurité sont réalisés

**Mots-clés :** POO, API REST, authentification (JWT, sessions), ORM, tests unitaires, validation, gestion des exceptions, refactoring

**Dans GreenRoots :** NestJS (architecture modulaire, injection de dépendances), services métier (orders, checkout, cart, trees), guards JWT/Roles, JWT stocké en cookie HttpOnly (migration sécurité XSS), helper centralisé cookie options, validation class-validator, Jest, style défensif, gestion des exceptions (filters), Prisma ORM

---

#### CP4 - Contribuer à la gestion d'un projet informatique

**Ce qu'il faut démontrer :**
- Avoir planifié et suivi ses tâches de développement (backlog, sprints, tickets…)
- Avoir utilisé un outil de gestion de projet (Trello, Jira, GitHub Projects…)
- Avoir participé ou organisé des réunions de suivi, rédigé des comptes rendus
- Avoir travaillé en adéquation avec une méthode de gestion (Agile/Scrum ou séquentielle)

**Critères d'évaluation (jury) :**
- Les tâches sont planifiées en fonction du délai
- Le suivi des tâches est en rapprochement avec la planification
- Les procédures qualité sont mises en oeuvre
- L'environnement de développement est en adéquation avec l'architecture
- Les outils collaboratifs sont choisis en fonction de la méthode
- Les comptes rendus de réunion sont structurés

**Mots-clés :** Agile/Scrum, kanban, backlog, sprints, outils de gestion (Trello…), comptes rendus, collaboration

**Dans GreenRoots :** Méthode Agile/Scrum, 4 sprints d'1 semaine, Trello (Kanban), daily meetings, sprint reviews, Git flow (branches, PRs), convention de commits, rôles définis (PO, SM, Lead Dev Front, Lead Dev Back, Dev Full-stack)

---

### AT2 - Concevoir et développer une application sécurisée organisée en couches

#### CP5 - Analyser les besoins et maquetter une application

**Ce qu'il faut démontrer :**
- Avoir analysé un cahier des charges pour identifier les besoins utilisateurs
- Avoir formalisé les besoins (user stories, cas d'usage…)
- Avoir réalisé des maquettes (wireframes, prototypes) avec un outil dédié
- Avoir modélisé l'enchaînement des écrans
- Avoir constitué un dossier de conception structuré

**Critères d'évaluation (jury) :**
- Les besoins couvrent l'ensemble des exigences utilisateur
- Les maquettes sont conformes au CDC
- L'enchaînement des maquettes est formalisé par un schéma
- Le dossier de conception est structuré

**Mots-clés :** cahier des charges, user stories, use cases, wireframes, Figma/Balsamiq, zoning, dossier de conception, RGPD, RGAA

**Dans GreenRoots :** CDC rédigé en sprint 0, user stories (26 user stories, 3 rôles), maquettes Figma, arborescence des pages, routes API documentées (Swagger/OpenAPI), diagrammes de cas d'utilisation

---

#### CP6 - Définir l'architecture logicielle d'une application

**Ce qu'il faut démontrer :**
- Avoir défini une architecture multicouche (ex : MVC, client/serveur, API + front séparé…)
- Avoir justifié les choix techniques (frameworks, ORM, patterns…)
- Avoir intégré des considérations de sécurité dans l'architecture (ANSSI, OWASP)
- Avoir identifié les besoins d'éco-conception

**Critères d'évaluation (jury) :**
- L'architecture est conforme aux bonnes pratiques multicouche répartie sécurisée
- Le rôle de chaque couche est bien défini (stratégie de sécurité)
- Les besoins d'éco-conception sont identifiés

**Mots-clés :** architecture multicouche, MVC, API REST, design patterns, sécurité by design, DICP, éco-conception

**Dans GreenRoots :** Architecture 3 couches (SPA React / API REST NestJS / PostgreSQL+Redis), architecture modulaire NestJS (séparation controllers/services/DTOs), guards de sécurité, cookie HttpOnly pour JWT (sécurité by design, OWASP), cache Redis dans l'architecture (layer cache entre service et BDD), conteneurisation Docker, Nginx reverse proxy ; éco-conception : Vite (build léger), Tailwind purge CSS, shadcn/ui (import sélectif), cache Redis (réduction requêtes SQL redondantes)

---

#### CP7 - Concevoir et mettre en place une base de données relationnelle

**Ce qu'il faut démontrer :**
- Avoir conçu un schéma conceptuel (MCD/ERD), logique (MLD) et physique (MPD)
- Avoir créé la base de données via scripts SQL
- Avoir défini les droits utilisateurs et les contraintes d'intégrité
- Avoir créé un jeu d'essai et mis en place une procédure de sauvegarde/restauration

**Critères d'évaluation (jury) :**
- Le schéma conceptuel respecte les règles du relationnel
- Le schéma physique est conforme au CDC
- Les règles de nommage sont respectées
- L'intégrité, sécurité et confidentialité des données sont assurées
- La BDD de test est créée avec jeu d'essai complet et restaurable

**Mots-clés :** MCD, MLD, MPD, SQL, contraintes (clés étrangères, unicité…), gestion des droits, jeu d'essai, backup/restore, RGPD

**Dans GreenRoots :** PostgreSQL + PostGIS, Prisma (13 modèles, 25 migrations), contraintes d'intégrité (FK, unicité), seed de données de test, enums (PaymentStatus, OrderStatus, AddressType), droits via rôles applicatifs

---

#### CP8 - Développer des composants d'accès aux données SQL et NoSQL

**Ce qu'il faut démontrer :**
- Avoir codé des opérations CRUD sécurisées (requêtes paramétrées, ORM, procédures stockées)
- Avoir géré l'intégrité des données et les transactions
- Avoir validé les entrées avant toute interaction avec la base
- Avoir réalisé des tests unitaires et de sécurité sur les composants d'accès aux données
- Avoir travaillé avec du SQL et/ou du NoSQL (MongoDB, Redis…)

**Critères d'évaluation (jury) :**
- Les traitements répondent aux fonctionnalités du dossier de conception
- Les cas d'exception sont pris en compte
- L'intégrité et la confidentialité des données sont maintenues
- Les conflits d'accès sont gérés
- Toutes les entrées sont contrôlées et validées
- Les tests unitaires et de sécurité sont réalisés

**Mots-clés :** SQL, NoSQL, ORM, CRUD, injection SQL, transactions, requêtes paramétrées, tests unitaires

**Dans GreenRoots :** Prisma ORM (accès SQL type-safe, requêtes paramétrées, transactions pour les commandes), Redis (cache NoSQL clé/valeur avec pattern cache-aside : pays TTL 24h, catégories TTL 1h + invalidation sur écriture ; token blacklist, login attempts), validation via DTOs (class-validator), gestion des conflits de stock

---

### AT3 - Préparer le déploiement d'une application sécurisée

#### CP9 - Préparer et exécuter les plans de tests d'une application

**Ce qu'il faut démontrer :**
- Avoir rédigé un plan de tests couvrant les tests d'intégration, système, sécurité et acceptation
- Avoir créé un environnement de tests dédié
- Avoir exécuté les tests (manuellement et/ou automatiquement) et analysé les résultats
- Avoir documenté les résultats dans un compte rendu de tests

**Critères d'évaluation (jury) :**
- Le plan de tests couvre l'ensemble des fonctionnalités
- Un environnement de tests est créé
- L'intégralité des tests sont conformes au plan
- Les résultats sont cohérents avec les résultats attendus
- Le plan tient compte des évolutions technologiques et de sécurité

**Mots-clés :** plan de tests, tests d'intégration, tests système, tests de sécurité (OWASP), tests d'acceptation (UAT), automatisation (Vitest, Jest, Postman…), rapport de tests

**Dans GreenRoots :** Tests unitaires (Jest back : 35 suites / 151 tests, Vitest + Testing Library front : 8 suites / 60 tests — incluant tests composants React avec interactions userEvent et vérification ARIA), coverage, audit de sécurité sprint 3, environnement de test Docker isolé, seed Prisma (jeux d'essai)

---

#### CP10 - Préparer et documenter le déploiement d'une application

**Ce qu'il faut démontrer :**
- Avoir rédigé une procédure de déploiement claire et reproductible
- Avoir écrit et documenté des scripts de déploiement
- Avoir pris en compte les dépendances et les versions des composants
- Avoir défini les environnements (dev, staging, production)

**Critères d'évaluation (jury) :**
- La procédure de déploiement est rédigée
- Les scripts de déploiement sont écrits et documentés
- Les environnements de tests sont définis
- Veille technologique sur le déploiement

**Mots-clés :** procédure de déploiement, scripts (bash, CI/CD), gestion des dépendances, versioning, environnements dev/prod, documentation technique

**Dans GreenRoots :** Docker Compose (compose.dev.yml + compose.prod.yml), Makefile documenté (commandes make), guide de déploiement Hetzner/Coolify (51KB), Nginx configuration, scripts de migration Prisma

---

#### CP11 - Contribuer à la mise en production dans une démarche DevOps

**Ce qu'il faut démontrer :**
- Avoir mis en place une pipeline CI/CD (GitHub Actions, GitLab CI…)
- Avoir automatisé les tests dans le processus d'intégration continue
- Avoir utilisé des outils de qualité de code (linter, SonarQube…)
- Avoir utilisé des conteneurs dans le pipeline (Docker)
- Avoir interprété les rapports générés par les outils CI

**Critères d'évaluation (jury) :**
- Les outils de qualité de code sont utilisés
- Les outils d'automatisation de tests sont utilisés
- Les scripts d'intégration continue s'exécutent sans erreur
- Le serveur d'automatisation est paramétré
- Les rapports de l'IC sont interprétés

**Mots-clés :** CI/CD, GitHub Actions, Docker, qualité de code (ESLint, Prettier…), intégration continue, livrable, automatisation des tests, DevOps

**Dans GreenRoots :** GitHub Actions (3 workflows : pr-checks → lint + tests, develop-ci → suite étendue, production-deploy → build + déploiement), ESLint + Prettier, tests automatisés dans la CI, Docker build automatisé, déploiement via Coolify

---

## Compétences transversales
Évaluées à travers les CPs ci-dessus :
1. **Communiquer en français et en anglais** - Documentation technique, réunions, CDC, commentaires de code
2. **Mettre en oeuvre une démarche de résolution de problème** - Debugging, analyse des résultats de tests, gestion des incidents
3. **Apprendre en continu** - Veille techno (sprint 3), nouvelles bibliothèques adoptées, sécurité
