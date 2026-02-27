# Dossier de Projet - Titre Professionnel CDA (Concepteur Développeur d'Applications)

## Contexte
Ce dépôt contient le travail de préparation et rédaction du **dossier de projet** pour le passage du titre professionnel CDA.

Le projet support est **GreenRoots**, une application e-commerce de vente d'arbres pour la reforestation.

## Structure du dépôt
```
dossier-projet/
├── CLAUDE.md                          # Ce fichier - contexte principal
├── docs/                              # Documentation de référence
│   ├── PLAN-DOSSIER.md               # Plan détaillé du dossier avec mapping CPs
│   ├── COMPETENCES.md                # Référentiel des 11 CPs et leur couverture
│   ├── PROJET-GREENROOTS.md          # Contexte technique et fonctionnel du projet
│   └── CONSIGNES-REDACTION.md        # Contraintes de format et rédaction
├── projet-greenroots/                 # Code source de l'application
│   ├── app/backend/                   # API NestJS
│   └── app/frontend/                  # SPA React/Vite
├── Référentiel Activités Compétences.pdf  # REAC + RE officiel
└── Cahier des charges GreenRoots.docx     # CDC initial (sprint 0)
```

## Règles de travail

### Rédaction du dossier
- Le dossier fait entre **40 et 60 pages** (hors page de garde, sommaire, annexes)
- Les annexes font **40 pages maximum**
- Toujours se référer à `docs/COMPETENCES.md` pour vérifier la couverture des CPs
- Toujours se référer à `docs/PLAN-DOSSIER.md` pour la structure imposée
- Le ton est professionnel, technique mais accessible au jury
- Justifier chaque choix technique en lien avec les compétences évaluées

### Stack technique
- **Frontend** : React 19 + Vite + TanStack Router/Query + Zustand + Tailwind + shadcn/ui
- **Backend** : NestJS 11 + Prisma 7 + PostgreSQL/PostGIS + Redis
- **Auth** : Passport.js (JWT + Local), guards, roles
- **Paiement** : Stripe
- **Images** : Cloudinary
- **Email** : Resend
- **Tests** : Vitest + Testing Library (front), Jest (back)
- **CI/CD** : GitHub Actions (3 workflows)
- **Conteneurisation** : Docker Compose (8 services)
- **Déploiement** : Hetzner + Coolify

### Commandes utiles
- Explorer le code : regarder dans `projet-greenroots/app/`
- Schéma BDD : `projet-greenroots/app/backend/prisma/schema.prisma`
- Routes API : contrôleurs dans `projet-greenroots/app/backend/src/*/`
- Routes frontend : `projet-greenroots/app/frontend/src/routes/`
- CI/CD : `.github/workflows/` dans projet-greenroots
