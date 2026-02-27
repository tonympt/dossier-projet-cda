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
| CP10 | Préparer et documenter le déploiement | Oui | §10 (annexe) |
| CP11 | Contribuer à la mise en production (DevOps) | Oui | §11 (annexe) |

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

#### 5.1 Contraintes du projet et livrables attendus
- Contraintes techniques, délais, périmètre MVP

#### 5.2 Architecture logicielle du projet (~2-3 pages)
> **CP6** - Définir l'architecture logicielle
- Architecture multicouche : SPA React / API REST NestJS / PostgreSQL + Redis
- Rôle de chaque couche et stratégie de sécurité
- Diagramme d'architecture
- Éco-conception : choix de Vite (léger), Tailwind (purge CSS), shadcn/ui (import sélectif)

#### 5.3 Maquettes et enchaînement des maquettes (~2-3 pages)
> **CP5** - Maquetter une application
- Maquettes principales (landing, catalogue, fiche arbre, panier, checkout, admin)
- Schéma d'enchaînement des écrans
- Respect RGAA / accessibilité

#### 5.4 Modèle de données (~2-3 pages)
> **CP7** - Concevoir et mettre en place une BDD relationnelle
- MCD (Modèle Conceptuel de Données) / Modèle entités-associations
- MPD (Modèle Physique de Données) / Schéma Prisma
- Script de création de la BDD (extrait)
- Gestion des droits d'accès, intégrité, sécurité des données

#### 5.5 Diagramme de cas d'utilisation (~1 page)
> **CP5**
- Diagramme UML des cas d'utilisation (Visiteur, Utilisateur, Admin)

#### 5.6 Diagrammes de séquence (~1-2 pages)
> **CP5, CP6**
- Cas les plus significatifs : processus de commande, authentification

---

### 6. Spécifications techniques (~3-4 pages)
> **CP1, CP6** - Environnement de travail et architecture

- Stack technique détaillée et justification des choix
- Environnement de développement : Docker Compose, services configurés
- Conteneurisation : description des services (front, back, DB, Redis, Nginx, Stripe CLI)
- Sécurité : stratégie globale (JWT, guards, validation, HTTPS, helmet, rate limiting)

---

### 7. Réalisations - Extraits de code significatifs (~10-12 pages)
> **CP2, CP3, CP8**

#### 7.1 Interfaces utilisateur (~3-4 pages)
> **CP2** - Développer des interfaces utilisateur
- Captures d'écran + extraits de code correspondants
- Exemples : page catalogue, fiche arbre, panier, checkout, admin
- Responsive design, Atomic Design (atoms/molecules/organisms)
- Validation côté client (Zod + React Hook Form)

#### 7.2 Composants métier (~3-4 pages)
> **CP3** - Développer des composants métier
- Extraits de code backend NestJS
- Exemples : service de commande (transactions), service d'authentification, guards
- Architecture modulaire, injection de dépendances
- Style défensif, gestion des exceptions

#### 7.3 Composants d'accès aux données (~2-3 pages)
> **CP8** - Développer des composants d'accès aux données SQL et NoSQL
- Accès SQL via Prisma ORM (requêtes, transactions, relations)
- Accès NoSQL via Redis (cache, token blacklist, login attempts)
- Validation des entrées, gestion de l'intégrité

#### 7.4 Autres composants (~1-2 pages)
- Contrôleurs, middlewares, intercepteurs
- Intégration Stripe (paiement)
- Intégration Cloudinary (images)

---

### 8. Éléments de sécurité de l'application (~3-4 pages)
> Transversal sécurité (CP2, CP3, CP6, CP8)

- Authentification JWT (access + refresh tokens, expiration)
- Autorisation par rôles (guards)
- Protection contre les attaques courantes (XSS, CSRF, injection SQL)
- Helmet (headers HTTP), rate limiting (throttler)
- Validation des entrées (DTOs, Zod)
- Hachage des mots de passe (bcrypt)
- Gestion des tokens (blacklist, login attempts)
- RGPD : mentions légales, suppression de compte, données minimales

---

### 9. Plan de tests (~3-4 pages)
> **CP9** - Préparer et exécuter les plans de tests

- Stratégie de tests (unitaires, intégration, E2E)
- Outils : Jest (back), Vitest + Testing Library (front)
- Couverture de tests
- Jeu d'essai de la fonctionnalité la plus représentative :
  - Données en entrée
  - Données attendues
  - Données obtenues
  - Analyse des écarts éventuels

---

### 10. Veille sécurité (~2-3 pages)
> Transversal (CP9, CP10, CP11)

- Description de la veille effectuée durant le projet
- Sources de veille (OWASP, ANSSI, CVE, etc.)
- Vulnérabilités trouvées dans les dépendances
- Failles potentiellement identifiées et corrigées
- Audit de sécurité réalisé en sprint 3

---

### ANNEXES (40 pages max, hors comptage)

#### A. Maquettes des interfaces utilisateur
- Ensemble des maquettes détaillées

#### B. Captures d'écrans et code correspondant
- Interfaces utilisateur complètes

#### C. Code de composants métier significatifs
- Services backend complets

#### D. Code de composants d'accès aux données
- Requêtes Prisma, cache Redis

#### E. Code d'autres composants
- Contrôleurs, configurations, CI/CD

#### F. Documentation de déploiement
> **CP10** - Préparer et documenter le déploiement
- Procédure de déploiement (Hetzner/Coolify)
- Docker Compose prod
- Scripts de déploiement

#### G. CI/CD et DevOps
> **CP11** - Contribuer à la mise en production
- Workflows GitHub Actions
- Rapports de qualité de code
- Rapports de tests automatisés

---

## Estimation de pages

| Section | Pages estimées |
|---------|---------------|
| 1. Liste des compétences | ~1 |
| 2. Cahier des charges | ~3-4 |
| 3. Entreprise et service | ~1-2 |
| 4. Gestion de projet | ~4-5 |
| 5. Spécifications fonctionnelles | ~8-10 |
| 6. Spécifications techniques | ~3-4 |
| 7. Réalisations (code) | ~10-12 |
| 8. Sécurité | ~3-4 |
| 9. Plan de tests | ~3-4 |
| 10. Veille sécurité | ~2-3 |
| **TOTAL** | **~40-50 pages** |
