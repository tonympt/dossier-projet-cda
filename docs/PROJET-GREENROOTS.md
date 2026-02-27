# Contexte Technique et Fonctionnel - GreenRoots

## Description du projet
GreenRoots est une plateforme e-commerce permettant à des particuliers et entreprises d'acheter des arbres pour contribuer à la reforestation mondiale.

## Équipe (5 personnes)
- **Marie** : Product Owner
- **Léo** : Scrum Master
- **Vincent** : Lead Dev Back
- **Tony** : Lead Dev Front (candidat)
- **Naomi** : Développeur Full-stack

## Organisation en sprints (1 semaine chacun)
- **Sprint 0** : Conception (CDC, maquettes, architecture, modèle BDD, user stories)
- **Sprint 1** : Mise en place de la base (frontend React, backend NestJS, Docker, BDD)
- **Sprint 2** : Finalisation des fonctionnalités MVP (e-commerce complet)
- **Sprint 3** : Refactor, CI/CD, audits (sécurité, éco-conception, SEO, accessibilité)

## Fonctionnalités MVP
1. **Landing page** : Présentation de GreenRoots et sélection d'arbres
2. **Authentification** : Inscription, connexion, mot de passe oublié, vérification email
3. **Catalogue** : Liste des arbres avec filtrage par catégories/localisation
4. **Fiche produit** : Détail d'un arbre (nom, nom scientifique, description, prix, image)
5. **Panier** : Ajout, modification quantité, suppression d'articles
6. **Checkout** : Formulaire d'adresse + paiement Stripe (simulé)
7. **Confirmation de commande** : Récapitulatif post-achat
8. **Espace utilisateur** : Profil, historique de commandes, modification mot de passe
9. **Back-office admin** : CRUD arbres, gestion des commandes, gestion des localisations

## Stack technique

### Frontend
| Techno | Rôle | Version |
|--------|------|---------|
| React | Framework UI | 19.2.0 |
| Vite | Build tool | 7.2.4 |
| TanStack Router | Routing type-safe | 1.139.0 |
| TanStack Query | Data fetching/cache | 5.90.10 |
| Zustand | State management (panier) | 5.0.9 |
| Tailwind CSS | Styling utilitaire | 4.1.17 |
| shadcn/ui + Radix UI | Composants UI | - |
| React Hook Form + Zod | Formulaires + validation | 7.71.1 / 4.3.5 |
| Stripe React | Paiement | 5.4.1 |
| Vitest + Testing Library | Tests | 4.0.18 |

### Backend
| Techno | Rôle | Version |
|--------|------|---------|
| NestJS | Framework API | 11.0.1 |
| Prisma | ORM | 7.3.0 |
| PostgreSQL + PostGIS | BDD relationnelle | 16 |
| Redis | Cache + sessions | alpine |
| Passport.js | Authentification (JWT + Local) | - |
| Stripe | Paiement | 20.0.0 |
| Resend | Emails transactionnels | 6.7.0 |
| Cloudinary | Stockage images | - |
| Helmet | Sécurité headers HTTP | 8.1.0 |
| Throttler | Rate limiting | 6.5.0 |
| Swagger | Documentation API | 11.2.3 |
| Jest | Tests | 29.7.0 |

### Infrastructure
| Techno | Rôle |
|--------|------|
| Docker Compose | Conteneurisation (8 services) |
| Nginx | Reverse proxy |
| GitHub Actions | CI/CD (3 workflows) |
| Hetzner + Coolify | Hébergement production |

## Architecture de la BDD (13 modèles Prisma)
- **User** : Profil utilisateur (auth, infos personnelles)
- **Role** : Rôles (USER, ADMIN)
- **Tree** : Catalogue produits (nom, description, prix, image)
- **TreeStock** : Gestion des stocks (quantité, seuil bas)
- **Category** : Catégories d'arbres
- **Location** : Lieux de plantation (coordonnées PostGIS)
- **Country** : Pays (facturation/plantation)
- **Cart / CartItem** : Panier par utilisateur
- **Order / OrderItem** : Commandes et détails
- **OrderAddress** : Adresses de facturation/livraison

## Routes principales

### Pages publiques
`/` `/trees` `/tree/:id` `/about` `/legal` `/terms` `/privacy` `/login` `/register`

### Espace achat (authentifié)
`/cart` `/checkout` `/order-confirmation`

### Espace utilisateur (authentifié)
`/profile` `/profile/me` `/profile/me/orders`

### Espace admin
`/admin` `/admin/trees` `/admin/trees/new` `/admin/trees/edit/:id` `/admin/orders`

## Sécurité implémentée
- JWT (access + refresh tokens) avec expiration
- Guards d'authentification et d'autorisation par rôles
- Helmet (protection headers HTTP)
- Rate limiting (60 req/min par IP)
- Bcrypt pour le hachage des mots de passe
- Token blacklist (déconnexion)
- Protection brute force (login attempts via Redis)
- Validation des entrées (class-validator backend, Zod frontend)
- HTTPS en production
- RGPD : mentions légales, suppression de compte

## CI/CD
1. **pr-checks.yml** : Lint + tests sur les PRs (timeout 5min)
2. **develop-ci.yml** : Suite de tests étendue sur develop
3. **production-deploy.yml** : Build Docker + déploiement sur main
