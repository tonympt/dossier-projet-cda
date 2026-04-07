# Sujets oral — Points non (ou peu) développés dans le dossier écrit

> Ce fichier liste les sujets que Tony peut évoquer/montrer lors de la présentation orale ou de l'entretien technique, mais qui ne sont pas détaillés dans le dossier écrit (par manque de place ou choix de priorisation).

---

## Accessibilité / Lighthouse

- **Scores Lighthouse** : performance 95+, accessibilité 100, SEO 100
- shadcn/ui (basé sur Radix UI) fournit l'accessibilité ARIA nativement
- Pas de travail spécifique RGAA majeur, mais les bases sont couvertes par le choix de la bibliothèque
- Montrer une capture Lighthouse si le jury demande
- **CP concernée** : CP2, CP5

---

## RGPD — Pattern anonymisation

- `DELETE /auth/me` : transaction Prisma atomique 4 étapes
  1. Anonymiser OrderAddress (noms, adresse → "Utilisateur supprimé", "00000")
  2. Anonymiser Order.contactEmail → "deleted@anonymized.local"
  3. Supprimer Cart/CartItem
  4. Supprimer User
- **Pourquoi pas cascade delete ?** Obligation comptable de conservation 6 ans (L123-22) — on efface l'identité mais on garde les données financières (montants, factures, articles)
- Conforme Article 17 RGPD (droit à l'effacement) + obligation légale de conservation
- Frontend : section danger rouge dans le profil, AlertDialog confirmation par mot de passe, clear cache TanStack Query + logout Zustand + redirect
- **CP concernée** : CP3, CP8

---

## RGPD — Export de données

- `GET /auth/me/export` : JSON structuré (profil, commandes avec items et adresses, panier)
- Header Content-Disposition pour téléchargement automatique
- Exclusion automatique des champs sensibles (password, tokens) via select Prisma
- Bouton dans le profil utilisateur
- **CP concernée** : CP3, CP8

---

## Eco-conception

- **Cohérence mission/technique** : site de reforestation → démarche éco-conception logique

### Code splitting — le détail technique
- **35+ routes lazy-loaded** via `lazyRouteComponent(() => import("@/features/..."))` de TanStack Router
- Chaque `import()` dynamique signale à **Vite** de créer un **chunk séparé** au build (fichier JS distinct)
- Le build produit tous les chunks, mais le navigateur ne télécharge **que celui de la route visitée**
- Au clic sur une nouvelle route → TanStack Router appelle l'import → le navigateur fetch le chunk → React rend le composant
- **Cache Nginx** : `Cache-Control: public, immutable` + `expires 1y` → chunk téléchargé une seule fois, mis en cache navigateur
- Manual chunks Vite/Rollup (vendor séparé) pour les dépendances lourdes (React, TanStack...)
- **Résultat Lighthouse** : Performance 95+, Accessibilité 100, SEO 100

### Autres mesures
- Tailwind CSS purge (seules les classes utilisées dans le bundle)
- shadcn/ui : import sélectif (pas de bibliothèque UI entière)
- Redis cache : réduction requêtes SQL redondantes (countries TTL 24h, categories TTL 1h)
- Images Cloudinary : transformations côté CDN (pas de traitement serveur)
- Rendu conditionnel JS (`useMediaQuery`) plutôt que CSS hidden → composants lourds non montés sur mobile
- Police Montserrat hébergée en local (pas de requête externe Google Fonts)
- Upload Cloudinary direct frontend → évite double transfert (frontend → backend → Cloudinary)
- **Référentiels** : RGESN 2024, RWEB GreenIT v5, WSG (W3C)
- **CP concernée** : CP6

---

## Tests composants React — Détail

- LoadingSkeleton (4 tests) : rendu par défaut, rows custom, cas limite rows=0, wrapper CSS
- QuantityInput (7 tests) : valeur initiale, increment/decrement via userEvent.click(), désactivation aux bornes min/max, état disabled, attributs ARIA (aria-valuemin, aria-valuemax, aria-valuenow)
- Config vitest extraite dans `vitest.config.ts` séparé (compatibilité tsc -b), environnement jsdom
- Ajout tardif (sprint 3) — à assumer honnêtement : initiative de compléter la couverture CP2+CP9
- **CP concernée** : CP2, CP9

---

## Facturation PDF

- `invoice.service.ts` (491 lignes) avec @react-pdf/renderer
- Mentions légales vendeur obligatoires (CGI art. 289)
- Numéro de facture unique (généré atomiquement en transaction)
- TVA stockée en BDD : HT + taux (20%) + montant TVA + TTC
- Boutons téléchargement dans OrdersTable et OrderDetailPage
- **CP concernée** : CP3

---

## Migration JWT HttpOnly — Le récit complet

- Sprint initial : JWT en localStorage (simple, rapide)
- Sprint 3 / audit OWASP : identification du risque XSS → décision de migration
- Impact full-stack : cookie-parser backend, helper getCookieOptions(), extracteur JWT custom, CORS credentials, withCredentials frontend, suppression interceptor Authorization, suppression accessToken du store
- Fix inattendu : boucle infinie React (JSON.stringify comparison dans useEffect)
- Montrer le diff avant/après si le jury demande le détail
- **CP concernée** : CP3, CP6

---

## Docker / Infrastructure

- 7 services dev vs 3 services prod (pourquoi la différence)
- Dockerfiles multi-stage : image build (node) → image run (Alpine, non-root)
- expose vs ports : services internes non accessibles de l'extérieur
- Healthchecks + depends_on (ordre de démarrage)
- Makefile ~20 commandes documentées
- **CP concernée** : CP1, CP10, CP11

---

## CI/CD — Détail des workflows

- pr-checks : lint + tests + npm audit sur chaque PR
- develop-ci : suite étendue avec BDD (service PostgreSQL dans le workflow)
- production-deploy : build Docker + déploiement
- Husky non implémenté → CI joue le rôle → évolution prévue (honnêteté)
- **CP concernée** : CP11

---

## Coolify / Déploiement prod

- Pas géré directement par Tony — à clarifier/approfondir avant le passage
- Savoir expliquer le principe : PaaS self-hosted sur Hetzner, déploiement via webhook GitHub
- **CP concernée** : CP10, CP11
