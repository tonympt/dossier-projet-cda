# Suivi entraînement oral — Jury CDA

> Fichier de suivi des sessions d'entraînement. Permet de reprendre là où on s'est arrêté.

## Règles de l'entraînement

1. **Une question à la fois** — on ne passe à la suivante que quand le point est fait
2. **Après chaque évaluation** : on traite les questions de Tony, on met à jour NOTES-ORAL.md si concept flou, on met à jour ce fichier
3. **Tony peut reformuler** après évaluation → réévaluation
4. **Mode aléatoire** — questions techniques tous domaines, niveau jury expert
5. **Notation /10** — rigueur terminologique exigée
6. **Questions à reposer** : quand Tony dit "note-la pour les prochaines sessions", la question est ajoutée dans "Questions à reposer". Pendant les sessions suivantes, glisser ces questions aléatoirement entre les nouvelles pour vérifier que c'est acquis.

## Dernière session

- **Date** : 2026-03-18
- **Statut** : Q1 évaluée et débriefée, prêt pour Q2

## Bilan par thème

| Thème | Statut | Niveau | Points faibles identifiés |
|-------|--------|--------|--------------------------|
| Requête HTTP bout en bout | Commencé | 6.5/10 | Confusion SSR/serverless, oubli code splitting, callback à solidifier, oubli ValidationPipe + Promise.all |
| Architecture (3 tiers, découplage, SPA vs SSR) | - | - | - |
| NestJS (modules, guards, pipes, interceptors, cycle de vie) | - | - | - |
| React (rendering, hooks, state management, routing) | - | - | - |
| Prisma (requêtes, transactions, relations, migrations) | - | - | - |
| SQL / Modèle de données (ERD, types, index, contraintes) | - | - | - |
| Sécurité (JWT, cookies, OWASP, CORS, CSRF, bcrypt, webhooks) | - | - | - |
| Design Patterns (Singleton, Strategy, State machine, Decorator) | - | - | - |
| Docker / DevOps (compose, multi-stage, CI/CD, healthchecks) | - | - | - |
| Stripe / Paiement (checkout, webhooks, idempotence, réconciliation) | - | - | - |
| Tests (Jest, Vitest, mocks, Testing Library, pyramide) | - | - | - |
| RGPD / Conformité (anonymisation, export, cookies, factures) | - | - | - |
| Performance / Éco-conception (code splitting, cache, Lighthouse) | - | - | - |
| TypeScript (typage, generics, discriminated unions, inférence) | - | - | - |
| Git / Workflow (GitFlow, PRs, hooks, conventions) | - | - | - |

## Historique des questions

### Session 1 — 2026-03-18

| # | Question | Thème | Note | Verdict |
|---|----------|-------|------|---------|
| 1 | Décris le chemin d'une requête catalogue bout en bout | Requête HTTP | 6.5/10 | Flux compris mais erreurs terminologiques (serverless≠SSR), oubli code splitting, callback flou, oubli ValidationPipe + Promise.all |
| 2 | Que se passe-t-il entre le retour du service Prisma et le JSON reçu par le client ? (@Serialize) | NestJS / Interceptors | 6/10 | Raisonnement bon (pourquoi custom), mais confusion pipe/interceptor, mécanisme plainToInstance inconnu |
| 3 | Requête admin PATCH /orders/:id — toutes les couches traversées | Sécurité + Métier | 7/10 | Flux métier solide (state machine, refund, stock), mais guards oubliés — réflexe sécurité à ancrer |
| 4 | Pourquoi unitPrice dans OrderItem alors que le prix est dans Tree ? | Modèle de données | 10/10 | Parfait — snapshot + obligation légale 6 ans |
| 5 | Pourquoi Zustand plutôt que Context API pour le panier ? | React / State | 8.5/10 | Bon fond (re-render, store externe), préciser sélecteurs granulaires + middleware persist |
| 6 | 7 services dev → 3 en prod, où sont passés les autres ? | Docker | 7/10 | Stripe CLI et Prisma Studio OK, mais Redis oublié + "tout dans l'image" trop vague |
| 7 | Pourquoi une table Role plutôt qu'un enum Prisma ? | Modèle de données | 7/10 | Relation one-to-many bien maîtrisée, honnête sur la partie non touchée, hésitant sur enum vs table |
| 8 | Webhook Stripe reçu deux fois — ton code gère ? | Stripe / Paiement | 9/10 | Idempotence + anti-spoofing bien décrits, juste nommer le concept "idempotence" |

## Questions à reposer

> Questions où Tony a eu des difficultés — à reposer aléatoirement dans les prochaines sessions.

- **Q1** : Requête bout en bout — vérifier : SSR vs serverless, code splitting, ValidationPipe, callback
- **SSR vs Serverless** : "Quelle est la différence entre SSR, CSR et serverless ?" (confusion Q1)
- **ValidationPipe** : "Que fait le ValidationPipe global et ses 4 options ?" (oublié en Q1)
- **Promise.all vs Transaction** : "Quelle différence entre Promise.all et une transaction Prisma ?" (clarifié post-Q1)
- **@Serialize / plainToInstance** : "Comment fonctionne la sérialisation des réponses API dans GreenRoots ?" (Q2 — mécanisme à solidifier)
- **Guards en premier** : "Décris les couches de sécurité d'une route admin" — vérifier que Tony commence par les guards spontanément (Q3)
- **JwtAuthGuard détaillé** : "Décris les étapes exactes du JwtAuthGuard quand une requête arrive. Que vérifie-t-il ? Dans quel ordre ?" (Tony n'a pas fait cette partie, à solidifier)
- **RolesGuard détaillé** : "Comment le RolesGuard sait quels rôles sont autorisés sur une route ? Décris le mécanisme." (Reflector + @Roles)
- **Rôle JWT vs BDD** : "Le rôle est dans le JWT. Que se passe-t-il si un admin change le rôle d'un utilisateur alors que son JWT est encore valide ?" (piège jury classique)
- **Blacklist JWT** : "Un utilisateur se déconnecte. Son JWT n'est pas expiré. Comment empêchez-vous sa réutilisation ?"
- **Flux panier** : "Décris le flux complet quand un utilisateur ouvre la page panier — de la réhydratation localStorage jusqu'à l'affichage avec les données serveur."
- **Architecture prod Docker** : "Décris comment les 3 conteneurs communiquent en prod. Qui est accessible depuis internet ? Comment le frontend parle au backend ? Comment le backend se connecte à la base ?"
- **Variables d'env Docker** : "Quelle différence entre les variables VITE_* du frontend et les variables du backend ? Pourquoi l'une est un ARG et l'autre un environment ?"
- **Relations BDD** : "User/Role c'est quel type de relation ? Où est la FK et pourquoi ? Comment tu détermines le type de relation ?"
- **Suppression RGPD** : "Le user est supprimé ou conservé après la suppression de compte ? Que devient le userId dans les commandes ?" (erreur corrigée Q11)
- **CI/CD détaillé** : "Décris les 3 workflows CI. Quand se déclenchent-ils ? Que vérifient-ils ? Quelle est la différence entre pr-checks et develop-ci ?" (Q13 — point faible)
- **Husky** : "Qu'est-ce qui se passe avant même que la CI ne tourne ?" (oublié Q13)
- **Strategy vs Factory** : "Quelle différence entre le pattern Strategy (Passport) et le pattern Factory (StripeModule) dans ton projet ?" (confondu Q16)
- **TanStack Router typé** : "Quel est l'avantage principal de TanStack Router par rapport à React Router ? Donne un exemple concret." (Q20 — pas implémenté par Tony)

| 14 | (Repose) JWT volé après logout — que se passe-t-il ? | Sécurité / Guards | 8.5/10 | Blacklist Redis bien décrite, config Passport détaillée. Confusion payload/body, étapes Passport avant validate oubliées |
| 15 | Code splitting — comment ça marche, que se passe-t-il au clic ? | Performance | 7.5/10 | Concept chunks/lazy compris, manque le mécanisme concret (lazyRouteComponent + import dynamique + cache Nginx) |
| 16 | Passport Local/JWT — quel design pattern ? | Design Patterns | 5/10 | Confondu Factory et Strategy. Le pattern est Strategy (comportement interchangeable), pas Factory (création d'objets) |
| 17 | (Repose) Comment les 3 conteneurs communiquent en prod | Docker | 9/10 | Flux maîtrisé, Coolify/Nginx/réseau Docker bien décrits. Préciser Traefik derrière Coolify |
| 18 | Comment NestJS sait quelle instance injecter ? | NestJS / IoC | 7→8.5/10 | Concept compris (pas de new, module fournit), bonne reformulation des 4 champs @Module après explication |
| 19 | (Repose) Différence pr-checks vs develop-ci | CI/CD | 8/10 | Bonne progression (5→8), déclencheurs et BDD corrects. Oubli seed idempotent + coverage, confusion Prettier/ESLint |
| 20 | TanStack Router typé vs React Router | TypeScript / Frontend | 5/10 | SPA et lazy loading compris, mais pas de réponse sur le typage (argument principal). N'a pas implémenté cette partie |
| 21 | Pourquoi bcrypt et pas SHA-256 ? C'est quoi le 10 ? | Sécurité | 6/10 | Intuition correcte mais confusions : "encryptage" au lieu de hachage, cost factor/salt mélangés, "décrypter" un hash |
| 22 | (Repose) Sérialisation réponses API — pourquoi custom ? | NestJS / Interceptors | 7/10 | Progression (6→7), le pourquoi est compris (Prisma plain objects), flux à mieux articuler |
| 23 | Pourquoi TreeStock séparé de Tree ? | Modèle de données | 6/10 | Argument évolutivité correct mais trop vague — préciser données statiques vs volatiles, contention, stockReservedAt |

## Historique (suite)

| # | Question | Thème | Note | Verdict |
|---|----------|-------|------|---------|
| 9 | Comment testerais-tu syncWithStock() du cartStore ? | Tests | 7.5/10 | Bonnes assertions identifiées, manque les cas limites (stock 0, produit absent) et la structuration par scénario |
| 10 | (Repose) SSR vs CSR vs Serverless ? | Architecture | 6/10 | CSR/SSR globalement compris mais serverless pas acquis, hydratation imprécise, oubli code splitting |
| 11 | Pourquoi pas un simple DELETE FROM users ? | RGPD | 6→9/10 | Erreur initiale (user conservé vs supprimé), bonne reformulation après correction. Transaction + anonymisation maîtrisés |
| 12 | (Repose) Les 4 options du ValidationPipe global | NestJS / Pipes | 8/10 | 4 options bien décrites, confusion pipe/middleware, enableImplicitConversion pas terminé |
| 13 | Workflow Git de la feature au merge en prod | Git / CI/CD | 5/10 | Workflow branching OK, CIs mal maîtrisées (audit inversé, BDD en CI pas mentionnée, prod-deploy sous-estimé), Husky oublié |

## Notes ajoutées à NOTES-ORAL.md

> Traçabilité des ajouts faits pendant l'entraînement.

- **Session 1** : SPA vs SSR vs Serverless (section Architecture), ValidationPipe complété (transform + enableImplicitConversion), Promise.all vs Transaction + Pagination front (section Accès aux données), Reflector dans Guards RBAC, Guards détaillés (JwtAuthGuard + RolesGuard), Zustand vs Context API (section React), fichier EVOLUTIONS.md créé
