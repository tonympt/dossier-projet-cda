## Référentiel des compétences (résumés)

### Activité 1 — Développer une application sécurisée

---

#### CP1 — Installer et configurer son environnement de travail

**Ce qu'il faut démontrer :**
- Avoir installé et configuré les outils de développement nécessaires au projet (IDE, SDK, extensions…)
- Avoir mis en place un outil de gestion de versions (Git, GitHub/GitLab…)
- Avoir utilisé des conteneurs (Docker) pour reproduire un environnement de développement cohérent avec la production
- Avoir su lire de la documentation technique en français ou en anglais

**Mots-clés :** IDE, Git, Docker, docker-compose, variables d'environnement, fichiers de config

---

#### CP2 — Développer des interfaces utilisateur

**Ce qu'il faut démontrer :**
- Avoir développé des interfaces conformes à une maquette/charte graphique
- Avoir rendu les interfaces responsives (adaptées à différents supports)
- Avoir utilisé des composants graphiques (bibliothèques UI, frameworks front)
- Avoir géré les événements, utilisé des appels asynchrones (AJAX, fetch, async/await)
- Avoir documenté le code et réalisé des tests unitaires sur les composants UI
- Avoir pris en compte la sécurité côté client (validation des entrées, XSS…)

**Mots-clés :** HTML/CSS, JavaScript/TypeScript, framework front (React, Vue, Angular…), responsive design, RGPD, accessibilité, tests unitaires (Jest…)

---

#### CP3 — Développer des composants métier

**Ce qu'il faut démontrer :**
- Avoir développé la logique métier côté serveur en orienté objet, avec un style défensif
- Avoir géré la sécurité applicative (authentification, autorisation, validation des entrées)
- Avoir utilisé des composants d'accès aux données (ORM, repositories…)
- Avoir réalisé des tests unitaires et des tests de sécurité sur les composants
- Avoir documenté le code

**Mots-clés :** POO, API REST, authentification (JWT, sessions), ORM, tests unitaires, validation, gestion des exceptions, refactoring

---

#### CP4 — Contribuer à la gestion d'un projet informatique

**Ce qu'il faut démontrer :**
- Avoir planifié et suivi ses tâches de développement (backlog, sprints, tickets…)
- Avoir utilisé un outil de gestion de projet (Trello, Jira, GitHub Projects…)
- Avoir participé ou organisé des réunions de suivi, rédigé des comptes rendus
- Avoir travaillé en adéquation avec une méthode de gestion (Agile/Scrum ou séquentielle)

**Mots-clés :** Agile/Scrum, kanban, backlog, sprints, outils de gestion (Jira, Trello…), comptes rendus, collaboration

---

### Activité 2 — Concevoir et développer une application sécurisée organisée en couches

---

#### CP5 — Analyser les besoins et maquetter une application

**Ce qu'il faut démontrer :**
- Avoir analysé un cahier des charges pour identifier les besoins utilisateurs
- Avoir formalisé les besoins (user stories, cas d'usage…)
- Avoir réalisé des maquettes (wireframes, prototypes) avec un outil dédié
- Avoir modélisé l'enchaînement des écrans
- Avoir constitué un dossier de conception structuré

**Mots-clés :** cahier des charges, user stories, use cases, wireframes, Figma/Balsamiq, zoning, dossier de conception, RGPD, RGAA

---

#### CP6 — Définir l'architecture logicielle d'une application

**Ce qu'il faut démontrer :**
- Avoir défini une architecture multicouche (ex : MVC, client/serveur, API + front séparé…)
- Avoir justifié les choix techniques (frameworks, ORM, patterns…)
- Avoir intégré des considérations de sécurité dans l'architecture (ANSSI, OWASP)
- Avoir identifié les besoins d'éco-conception

**Mots-clés :** architecture multicouche, MVC, microservices, API REST, design patterns, sécurité by design, DICP, éco-conception

---

#### CP7 — Concevoir et mettre en place une base de données relationnelle

**Ce qu'il faut démontrer :**
- Avoir conçu un schéma conceptuel (MCD/ERD), logique (MLD) et physique (MPD)
- Avoir créé la base de données via scripts SQL
- Avoir défini les droits utilisateurs et les contraintes d'intégrité
- Avoir créé un jeu d'essai et mis en place une procédure de sauvegarde/restauration

**Mots-clés :** MCD, MLD, MPD, SQL, contraintes (clés étrangères, unicité…), gestion des droits, jeu d'essai, backup/restore, RGPD

---

#### CP8 — Développer des composants d'accès aux données SQL et NoSQL

**Ce qu'il faut démontrer :**
- Avoir codé des opérations CRUD sécurisées (requêtes paramétrées, ORM, procédures stockées)
- Avoir géré l'intégrité des données et les transactions
- Avoir validé les entrées avant toute interaction avec la base
- Avoir réalisé des tests unitaires et de sécurité sur les composants d'accès aux données
- Avoir travaillé avec du SQL et/ou du NoSQL (MongoDB, Redis…)

**Mots-clés :** SQL, NoSQL, ORM, CRUD, injection SQL, transactions, requêtes paramétrées, triggers, tests unitaires

---

### Activité 3 — Préparer le déploiement d'une application sécurisée

---

#### CP9 — Préparer et exécuter les plans de tests

**Ce qu'il faut démontrer :**
- Avoir rédigé un plan de tests couvrant les tests d'intégration, système, sécurité et acceptation
- Avoir créé un environnement de tests dédié
- Avoir exécuté les tests (manuellement et/ou automatiquement) et analysé les résultats
- Avoir documenté les résultats dans un compte rendu de tests

**Mots-clés :** plan de tests, tests d'intégration, tests système, tests de sécurité (OWASP), tests d'acceptation (UAT), automatisation (Cypress, Playwright, Postman…), rapport de tests

---

#### CP10 — Préparer et documenter le déploiement

**Ce qu'il faut démontrer :**
- Avoir rédigé une procédure de déploiement claire et reproductible
- Avoir écrit et documenté des scripts de déploiement
- Avoir pris en compte les dépendances et les versions des composants
- Avoir défini les environnements (SIT, UAT, production)

**Mots-clés :** procédure de déploiement, scripts (bash, CI/CD), gestion des dépendances, versioning, environnements SIT/UAT/PROD, documentation technique

---

#### CP11 — Contribuer à la mise en production dans une démarche DevOps

**Ce qu'il faut démontrer :**
- Avoir mis en place une pipeline CI/CD (GitHub Actions, GitLab CI…)
- Avoir automatisé les tests dans le processus d'intégration continue
- Avoir utilisé des outils de qualité de code (linter, SonarQube…)
- Avoir utilisé des conteneurs dans le pipeline (Docker)
- Avoir interprété les rapports générés par les outils CI

**Mots-clés :** CI/CD, GitHub Actions / GitLab CI, Docker, qualité de code (ESLint, SonarQube…), intégration continue, livrable, automatisation des tests, DevOps

