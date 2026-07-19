# Contenu du support oral — GreenRoots (CDA)

> Contenu validé slide par slide (message clé, texte de la slide, notes orateur, éléments visuels).
> Mis à jour après CHAQUE slide validée. Structure = plan validé (voir mémoire `plan-oral-greenroots`).
>
> **Paramètres** : ~40 min (~37 slides + bonus) · fil rouge = tunnel de vente · démo live en ouverture du bloc 3 · cadre pédagogique assumé + ambition production-ready.

---

## BLOC 1 — Introduction / cadrage

### Slide 1 — Page de garde ✅
- **Contenu** : réutilise la page de garde du dossier de projet (logo GreenRoots + titre + infos).
- **Infos** : Memponteil Tony · Centre de formation O'clock · Formation 06/2024 → 04/2026 · Titre CDA.
- **Notes orateur** : saluer le jury, se présenter, annoncer le projet (bref, présentation perso détaillée en slide 2). Debout, poser le ton.
- **Visuel** : logo `app/frontend/src/assets/images/logo.png`. Fond blanc épuré.

### Slide 2 — Mon parcours ✅
- **Message clé** : trajectoire cohérente vers « Concepteur Développeur » — « concevoir puis construire » est le fil de la vie pro.
- **Texte de la slide** (frise) :
  - De la finance au développement
  - 9 ans dans le secteur financier — montage financier
  - 2023 — formation DWWM : le déclic, une vocation confirmée
  - 2024-2026 — alternance CDA au Groupe LDLC : 2 ans sur l'ERP interne (Delphi), transverse à tous les métiers du groupe
  - « J'ai toujours aimé concevoir, puis construire. »
- **Notes orateur** : origine (finance, rigueur) → déclic (DWWM 2023, vocation) → saut (alternance LDLC, ERP Delphi multi-métiers) → transition vers GreenRoots. (Le CDI concepteur intégrateur IA @LDLC depuis avril 2026 est gardé pour la conclusion, slide 36.)
- **Visuel** : frise chronologique horizontale simple (€ → </> → 🎓). À produire dans l'outil de slides.

### Slide 3 — Le projet GreenRoots ✅
- **Message clé** : en 1 min, quoi / pour qui / quelle valeur, sans technique.
- **Texte de la slide** :
  - GreenRoots — acheter un arbre, contribuer à la reforestation
  - Le concept : plateforme e-commerce pour acheter des arbres destinés à la reforestation
  - Pour qui : particuliers sensibles à l'écologie · entreprises (RSE)
  - La promesse : achat simple + transparence (quel arbre, où il est planté)
- **Notes orateur** : accroche « le pourquoi » — beaucoup veulent agir pour l'environnement mais passer à l'acte est flou ; GreenRoots rend la reforestation aussi simple qu'un achat en ligne. Cadre assumé : cas d'école CDA mené avec l'ambition d'un projet production-ready.
- **Visuel** : sobre — logo + 3 pictos (🛒 achat simple · 🌱 reforestation · 🔎 transparence). À produire.

### Slide 4 — Périmètre & acteurs ✅
- **Message clé** : périmètre borné en ingénieur (MVP complet vs évolutions écartées) + poser les 3 rôles (hiérarchie d'accès).
- **Texte de la slide** :
  - Gauche : bloc pseudo-code `extends` (fond clair, monospace) :
    ```
    👤 Visiteur
         consulterCatalogue()
         ajouterAuPanier()

    🔑 Utilisateur  extends  Visiteur
         passerCommande()
         gérerProfil()
         voirHistorique()

    🛠️ Admin  extends  Utilisateur
         gérerArbres()      // CRUD
         gérerCommandes()
    ```
  - Droite : tableau MVP / Évolutions
    | ✅ MVP (livré) | 🔮 Évolutions (hors périmètre) |
    |---|---|
    | Catalogue + fiche arbre | Carte interactive de plantation |
    | Authentification complète | Suivi détaillé de la pousse |
    | Panier → Checkout **paiement Stripe** → Confirmation | Multilingue |
    | Espace utilisateur (profil, historique) | Parrainage |
    | Back-office admin | Portail partenaires |
- **Notes orateur** : 3 rôles en héritage d'accès (guards NestJS dans le code, PAS d'héritage POO — le pseudo-code illustre la hiérarchie des droits). Cœur du MVP = parcours d'achat complet avec vrai paiement Stripe (fil rouge). Bornage assumé. **Stripe présenté comme faisant partie du MVP** (pas de « c'était prévu simulé »).
- **Visuel** : bloc pseudo-code + tableau, à produire. Emojis cohérents partout.

### Slide 5 — Organisation & méthode ✅ `CP4`
- **Message clé** : démarche Agile réelle (équipe structurée, cycles courts, outils collaboratifs) — coche CP4.
- **Texte de la slide** (3 zones) :
  - Équipe — 5 personnes : Marie (PO) · Léo (SM) · Vincent (Lead Dev Back) · **Tony (Lead Dev Front — moi)** · Naomi (Full-stack)
  - Méthode — Agile/Scrum, 4 sprints d'1 semaine :
    ```
    Sprint 0          Sprint 1          Sprint 2          Sprint 3
    Conception        Fondations        MVP complet       Qualité & prod
    CDC · maquettes   React · NestJS    e-commerce        refactor · CI/CD
    archi · BDD · US  Docker · BDD      de bout en bout   audits sécu/SEO/a11y
    ```
  - Outils — Trello (Kanban, backlog) · Git/GitHub (branches, PR + code review obligatoire) · conventions (commits, ESLint + Prettier)
- **Notes orateur** : cadence 1 semaine, cérémonies (daily, revue, rétro). Le fil des sprints raconte une histoire (Sprint 0 = conception, Sprint 3 = qualité). Chaque feature en PR relue par un pair. Rôle = Lead Dev Front (a touché aux deux côtés).
- **Visuel** : frise des 4 sprints (à produire) + capture du board Trello (ressource externe, à sortir de Trello).
- **Option** : slide bonus « CP4 en détail » (Trello annoté + exemple PR/code review + Git flow) si le jury creuse — non retenue par défaut.

---

## BLOC 2 — Conception

### Slide 6 — Analyse des besoins ✅
> (Slide 6 dans la numérotation du plan ; le bloc 2 va des slides 6 à 13.)
- **Message clé** : partir d'un besoin exprimé (CDC) formalisé méthodiquement en user stories.
- **Texte de la slide** :
  - Du cahier des charges aux user stories
  - CDC rédigé en Sprint 0 — le besoin côté produit
  - Formalisé en **28 user stories** → 6 Visiteur · 14 Utilisateur · 7 Admin (+1 partagée)
  - Format : « En tant que [rôle], je veux [action] afin de [bénéfice] »
  - Les user stories ont alimenté le backlog Trello
  - 3 exemples réels (extraits du CDC) :
    - 👤 « En tant que visiteur, je veux consulter le catalogue des arbres afin de voir les arbres disponibles à l'achat. »
    - 🔑 « En tant qu'utilisateur, je veux compléter mes informations bancaires afin de payer mes produits. »
    - 🛠️ « En tant qu'admin, je veux ajouter un arbre au stock afin qu'il soit mis en vente. »
- **Notes orateur** : CDC → US → backlog Trello (pont analyse ↔ gestion de projet). Transition vers le diagramme de cas d'usage.
- **Visuel** : format US + 3 cartes (une par rôle). Optionnel : capture backlog Trello.

### Slide 7 — Diagramme de cas d'usage ✅
- **Message clé** : synthèse visuelle « qui fait quoi » + hiérarchie d'accès + maîtrise UML (include/extend). Formalisation du pseudo-code de la slide 4.
- **Contenu du diagramme** (`annexe-4-cas-utilisation.png` d'origine — on garde tel quel) :
  - 3 acteurs en héritage : Utilisateur hérite Visiteur, Admin hérite Utilisateur
  - Visiteur : consulter catalogue · voir fiche · gérer panier · s'inscrire · se connecter · réinitialiser mdp
  - Utilisateur (+hérite) : passer commande · consulter commandes · gérer profil · supprimer compte · exporter données (RGPD)
  - Admin (+hérite) : gérer arbres · catégories · localisations · commandes
  - Relations : « Passer commande » `<<include>>` « Vérifier le stock » et « Payer via Stripe » ; « Gérer les commandes » ↔ « Rembourser via Stripe » `<<extend>>`
- **Notes orateur** :
  - Héritage acteurs = hiérarchie d'accès (guards NestJS, pas d'héritage POO).
  - include (obligatoire) vs extend (optionnel/conditionnel) — formule prête.
  - Précision extend (si jury pointe le sens) : « Le remboursement étend la gestion de commande ; en UML strict la flèche extend va de l'extension vers le cas de base, donc de "Rembourser via Stripe" vers "Gérer les commandes". » (le PNG d'origine montre le sens inverse — à expliquer verbalement)
  - Pont code : « Passer commande include Vérifier le stock » = ce que fait `confirmCheckout` (fil rouge slide 20).
- **Visuel** : ✅ `annexe-4-cas-utilisation.png` (existe).

### Slide 8 — Architecture 3 tiers ✅ `CP6`
- **Message clé** : vue macro — architecture multicouche répartie, découplée et sécurisée (1er pilier CP6).
- **Texte de la slide** :
  - Une architecture 3 tiers découplée
  - ① Présentation — SPA React (Vite), dans le navigateur
  - ② Logique métier — API REST NestJS
  - ③ Données — PostgreSQL (+ PostGIS) · Redis (cache)
  - Nginx en reverse proxy : sert le front + route `/api` vers le back
  - Chaque couche ne communique qu'avec la suivante
- **Notes orateur** : définir « 3 tiers » (3 couches distinctes, déployables indépendamment) ; découplage front/back (SPA pure + API REST, pas Next.js) ; rôle Nginx (sert le front + route /api, le navigateur ne parle jamais direct au back/BDD) ; Redis = couche cache ; pourquoi (séparation responsabilités, scalabilité, sécurité). Détails : sécu par couche (slide 12 DICP), archi applicative (slide 10).
- **Visuel** : ✅ `annexe-1-architecture.png` (existe).

### Slide 9 — Justification de la stack ✅ `CP6`
- **Message clé** : aucun choix technique dû au hasard — chaque techno répond à un critère (2e pilier CP6).
- **Idée structurante** : 3 fils directeurs → 🔒 type-safety bout en bout · 🌱 éco-conception & perf · ♿ accessibilité.
- **Texte de la slide** : bandeau 3 critères + tableau (Techno / Rôle / Pourquoi) :
  - React + Vite (front) : écosystème mature, composants réutilisables, Vite rapide + moins de CPU
  - TanStack Router/Query : routing type-safe, cache intelligent
  - Tailwind + shadcn/ui : import sélectif (éco) + Radix UI (accessibilité native) + flexible
  - NestJS : architecture modulaire, DI, TypeScript
  - PostgreSQL + Prisma : SQL robuste + PostGIS + Prisma type-safe + migrations
  - Redis : cache + anti-brute-force + blacklist JWT
  - Stripe : webhooks source de vérité, aucune donnée bancaire chez nous
- **Notes orateur** : dérouler les 3 fils directeurs (pas lire le tableau). shadcn/ui = choix emblématique cochant les 3 critères. Rappeler découplage (pas Next.js). Stack complète/versions → slide 32/annexe. Redis présenté comme LE choix d'archi (pas « V2 »).
- **Visuel** : à produire — bandeau 3 pictos + tableau (logos optionnels).

### Slide 10 — Architecture applicative ✅ `CP6`
- **Message clé** : zoom à l'intérieur de chaque brique (après la vue 3 tiers) — structuration en couches et patterns, back + front (3e pilier CP6).
- **Texte de la slide** (2 colonnes) :
  - Backend modulaire (NestJS) : trajet Controller → Service → Prisma (rôle de chaque couche) · injection de dépendances (IoC) · ~14 modules par domaine · patterns Module/Guards/Decorators/State machine
  - Frontend React : Atomic Design (atoms shadcn/ui → molecules → organisms) · feature-based · séparation des 3 états (Zustand local / TanStack Query serveur / RHF formulaire)
- **Notes orateur** : dérouler le trajet de la donnée (sans lire de code) ; DI = découplage + testabilité (mock Prisma). **Honnêteté archi** : « modulaire en couches, PAS hexagonale au sens strict — service dépend direct de Prisma, choix pragmatique » (cf. NOTES-ORAL). Front : Atomic Design + feature-based ; point fort = séparation des 3 états (revient slide 18).
- **Visuel** : à produire (boîtes dans l'outil de slides) — 2 colonnes : couches back empilées + pyramide Atomic Design & 3 états.

### Slide 11 — Sécurité by design (DICP) ✅ `CP6`
- **Message clé** : sécurité pensée dès la conception, structurée selon les 4 indicateurs DICP (critère CP6 ANSSI/OWASP).
- **Texte de la slide** (4 quadrants) :
  - D Disponibilité : healthchecks Docker · cron réconciliation · dégradation gracieuse Redis (fallback BDD)
  - I Intégrité : transactions Prisma atomiques (stock/paiement) · validation DTO · contraintes BDD (FK, unicité, enums)
  - C Confidentialité : JWT HttpOnly+Secure+SameSite · bcrypt · guards RBAC · safeSelect (pas de mdp exposé)
  - P Preuve : factures numérotées · dates facturation · historique conservé (anonymisation sans suppression)
  - Défense en profondeur : Zod (client) → DTO (API) → Prisma (ORM paramétré) · conforme ANSSI + OWASP Top 10:2025
- **Notes orateur** : DICP = grille ANSSI/EBIOS (Disponibilité, Intégrité, Confidentialité, Preuve) ; 1 exemple par lettre ; défense en profondeur (validation à chaque couche). **Vue "by design" (démarche)** — les protections par faille = bloc sécurité slides 25-28 (ne pas se répéter).
- **Visuel** : à produire — 4 quadrants DICP + flèche défense en profondeur ; logos ANSSI/OWASP.

### Slide 12 — MERISE : MCD & MLD ✅ `CP7`
- **Message clé** : démarche MERISE (conceptuel → logique via règles de passage), méthode préférée du jury (1er pilier CP7).
- **Texte de la slide** (2 temps) :
  - ① MCD (le QUOI, indépendant techno) : entités + associations + cardinalités (1-N, N-N). Relations clés : User → Order → OrderItem → Tree ; Tree → Category / Location → Country. → diagramme `corps-mcd.png`.
  - ② MLD (dérivé par règles de passage) : **9 relations**, notation PK souligné + `#`FK, **attributs abrégés** :
    ```
    Role (id, name)
    User (id, email, #roleId, …)
    Country (id, code, name, …)
    Location (id, name, #countryId, …)
    Category (id, name)
    Tree (id, name, price, stockQuantity, #categoryId, #locationId, …)
    Order (id, paymentStatus, orderStatus, #userId, …)
    OrderItem (id, #orderId, #treeId, quantity, …) — UNIQUE(orderId, treeId)
    OrderAddress (id, #orderId, type, city, #countryCode, …) — UNIQUE(orderId, type)
    ```
    > (…) = autres attributs — détail complet des 9 relations dans le dossier
- **Notes orateur** : MERISE = 3 niveaux (MCD → MLD → MPD slide 13). MCD = le QUOI (montrer 1-2 associations). Passage MCD→MLD = associations deviennent tables/FK, PK/FK apparaissent (exemples OrderItem/OrderAddress + leurs UNIQUE). Ne pas lire les 9 relations. Teaser slide 13 : types PostgreSQL, enums, index, split Tree → TreeStock.
- **Visuel** : ✅ `MCD/corps-mcd.png` (MCD). MLD = 9 relations abrégées (monospace, PK/#FK en évidence), déjà dans le Word.

### Slide 13 — MPD / Prisma ✅ `CP7`
- **Message clé** : passage au niveau physique — le MLD devient un schéma concret PostgreSQL/Prisma avec les choix physiques (types, enums, index, optimisations). Boucle MERISE complète.
- **Texte de la slide** :
  - Nommage camelCase (code) ↔ snake_case (PostgreSQL) via `@map`/`@@map`
  - Types : prix en `Int` (centimes, standard Stripe, pas de float) · enums OrderStatus/PaymentStatus/AddressType · PostGIS pour coordonnées
  - Optimisation : split `Tree` → `TreeStock` (partition verticale) → 10 modèles (vs 9 au MLD)
  - Index `@@index` sur colonnes filtrées (userId, categoryId, paymentStatus)
  - 24 migrations versionnées
  - Extrait Prisma commenté (model Order : Int centimes, enum, @map, @@index)
- **Notes orateur** : MPD = physique (dépend du SGBD). @map fait le pont camelCase↔snake_case. Prix en Int/centimes (standard Stripe, évite float). Enums = machines à états (lien slide 22). TreeStock = partition verticale (stock change souvent) → 9 relations MLD → 10 modèles MPD. Index sur colonnes filtrées. 24 migrations = traçabilité.
- **Visuel** : ✅ `annexe-3-mpd.png` (MPD). Extrait Prisma (fond clair). Schéma complet + dictionnaire → annexe.

---

## BLOC 3 — Fil rouge : le tunnel de vente

### Slide 14 — 🎬 Démo live : parcours d'achat ✅
- **Rôle** : montrer le produit fini en action avant de le décortiquer (ouverture du fil rouge).
- **URL app déployée** : https://y12mkjriooggbyqn0mys3ap8.167.233.213.222.sslip.io/trees
- **Slide** : minimale — « 🎬 Démo — Le parcours d'achat » + URL (~2-3 min).
- **Script démo** : catalogue (+ filtre) → fiche arbre → ajout panier → checkout (adresse + carte test Stripe `4242 4242 4242 4242`) → confirmation (PAID).
- **Notes** : intro « parcours en conditions réelles sur l'app déployée en prod » ; rester tunnel de vente ; **déjà connecté** (compte test) ; Stripe en mode test ; **démo à froid — vérifiée, ça marche** ; répéter 2× au chrono.
- **Plan de secours** : vidéo/captures (annexe E) prêtes si souci réseau.

### Slide 15 — Vue d'ensemble du tunnel ✅
- **Message clé** : poser la carte du parcours + annoncer les 3 couches à traverser (slide de transition, ~30 s).
- **Texte de la slide** : `Panier → Checkout → Paiement Stripe → Confirmation` ; suivre la donnée à travers 🎨 Front (React) / ⚙️ Métier (NestJS) / 🗄️ Données (Prisma+Redis).
- **Notes orateur** : pourquoi ce fil rouge (fonctionnalité la plus complète, traverse les 3 couches, vraies décisions d'archi) ; annoncer le plan (maquettage → front → back → données) ; teaser transaction atomique + webhook source de vérité.
- **Visuel** : ✅ `corps-parcours-achat.png` (existe) OU frise épurée 4 étapes + 3 couches en légende.

### Slide 16 — Maquettage ✅ `CP5`
- **Message clé** : prouver la démarche de conception UI progressive (wireframe → maquette HF), appliquée au catalogue. Montrer le cheminement du design (Benoît).
- **Texte de la slide** : « Du wireframe à la maquette haute-fidélité » → ① Wireframe (Balsamiq) · ② Maquette HF (Figma). Conforme au produit final (cf. démo).
- **Notes orateur** : démarche progressive (wireframe = structure/zoning des blocs sans style → maquette HF = rendu final) ; conformité maquette ↔ réalité (jury peut vérifier via démo) ; outil Figma en Sprint 0.
- **Visuels** :
  - ✅ **Wireframe catalogue** : `diagrammes/corps-wireframe-catalogue.png` (+ source `.html` reproductible via Chrome headless : `--headless --screenshot --window-size=1700,1200 --force-device-scale-factor=2`). Reconstitué (source Balsamiq perdue), fidèle au frontend.
  - 🔧 **Maquette HF** : Tony l'a, sur le **catalogue** (confirmé) → à **exporter depuis Figma**. Appariement idéal : wireframe catalogue → HF catalogue (évolution du même écran).

### Slide 17 — Front (1) : le formulaire checkout ✅ `CP2`
- **Message clé** : UI de formulaire maîtrisée — validation typée (Zod + messages FR) + composants accessibles (shadcn/ui), synchronisation type-safe schéma↔form.
- **Texte de la slide** (2 extraits de code réels, fond clair) :
  - ① Schéma Zod `checkoutFormSchema` (contactEmail email + messages FR, min/max…) + `type CheckoutFormValues = z.infer<...>`
  - ② `<FormField>` shadcn/ui (FormLabel + FormControl/Input + FormMessage relié ARIA)
  - Points clés : RHF + Zod (zodResolver) · pays via TanStack Query (staleTime 1h) · form verrouillé pendant paiement
- **Notes orateur** : Zod = validation + typage (z.infer → schéma/form toujours sync) ; messages FR dans le schéma ; shadcn/ui (Radix) = label + erreur en ARIA (RGAA) ; défense en profondeur (validation client redoublée par DTO API) ; verrouillage form = machine à états (slide 18).
- **Visuels** : 2 extraits de code (fond clair) + 🔧 **screenshot réel — Tony le capture** : formulaire checkout avec **select pays** + **message d'erreur** de validation.
  - Fichiers source réels : `src/features/checkout/schemas/checkoutSchema.ts`, `src/features/checkout/CheckoutForm.tsx`, `src/features/checkout/api/checkoutApi.ts` (staleTime 1h).

### Slide 18 — Front (2) : état & asynchrone ✅ `CP2`
- **Message clé** : gestion d'état moderne — séparer 3 natures d'état (local/serveur/formulaire) + synchroniser panier↔stock.
- **Texte de la slide** : tableau 3 états → Local (Zustand + persist, localStorage, TTL 24h) · Serveur (TanStack Query, polling 5 min, refetch focus) · Formulaire (RHF + Zod). + extrait réel `syncWithStock` (réconciliation : introuvable→garde / stock OK→rien / insuffisant→ajuste quantité + notif).
- **Notes orateur** : ne pas tout mettre dans un état global, séparer par nature ; panier local peut être périmé → syncWithStock réconcilie avec le stock serveur + prévient l'utilisateur (dialog). Couvre CP2 (async, événements, state management).
- **Visuels** : 🔧 schéma 3 états (flèche sync panier↔stock) + extrait `syncWithStock` (fond clair). Sources : `cartStore.ts` (CART_EXPIRATION_MS 24h, syncWithStock), `useCartProducts.tsx` (staleTime/refetchInterval 5 min, refetchOnWindowFocus).

### Slide 19 — ⭐ Qualité front ✅ `CP2`
- **Message clé** : qualité d'interface mesurée et outillée (perf, a11y, SEO, design system, responsive, tests) = preuve concrète de CP2. NB : angle couverture CP2, PAS « passion front » (Tony full-stack).
- **Texte de la slide** (4 volets) :
  - ① **Lighthouse** — 1 capture = **4 scores** : Performance 95+ · Accessibilité 100 · Best Practices · SEO 100
  - ② **Atomic Design** — schéma classique (atoms→molecules→organisms). Tony montrera le schéma connu.
  - ③ **Responsive adaptatif** — `OrdersTable` table (desktop) → cartes (mobile) via `useMediaQuery` (changement de composant, pas juste CSS). → screenshot.
  - ④ **Accessibilité** — Radix/shadcn ARIA natif + aria-labels + `QuantityInput` (role spinbutton, aria-valuemin/max/now), testée (Vitest + TL).
- **Notes orateur** : Lighthouse = 4 catégories en 1 run. **Nuances** (cf. NOTES-ORAL) : a11y 100 = automatisé (~30-40% WCAG), pas RGAA complet ; SEO 100 = hygiène, mais SPA/CSR = limites SEO (SSR = évolution). Responsive intelligent (change de composant). Honnêteté RGAA.
- **Visuels** : 🔧 **capture Lighthouse** (Tony) · schéma Atomic Design classique (Tony) · 🔧 **screenshot responsive** table⇄cards (Tony) · pyramide/illustration à produire.

### Slide 20 — Back (1) : confirmCheckout ✅ `CP3`
- **Message clé** : pièce maîtresse métier — réserver le stock de façon atomique, sécurisée, idempotente, avant le paiement.
- **Texte de la slide** : extrait réel condensé `confirmCheckout` (`checkout.service.ts`) avec 4 repères : ① `$transaction` (atomique, rollback) ② sécurité (`order.userId !== userId` → Forbidden) ③ idempotence (`stockReservedAt`/PROCESSING → double-clic) ④ valider tout le stock avant d'écrire (`issues[]`, sinon rien décrémenté) → puis décrément atomique + `stockReservedAt`+`PROCESSING`.
- **Notes orateur** : transaction tout-ou-rien ; vérif propriétaire (Broken Access Control OWASP) ; idempotence double-clic ; collecte des issues avant écriture (1 manque → rien réservé, dialog front) ; retour structuré `{success, issues}`. Lien : « include Vérifier le stock » (slide 7) ; équivalent SQL du decrement → slide 23.
- **Visuel** : extrait annoté ①②③④ (fond clair). Source `checkout.service.ts`.

### Slide 21 — Back (2) : Stripe, le webhook source de vérité ✅ `CP3`
- **Message clé** : le résultat du paiement est décidé par Stripe et notifié via webhook (jamais par le navigateur), webhook sécurisé + idempotent.
- **Texte de la slide** : extrait réel condensé (`payment.controller.ts` + `payment.service.ts`) : `@Post('webhook') @SkipThrottle()` + rawBody ; ① signature HMAC (`constructEvent`) ; switch 3 events (succeeded → PAID+CONFIRMED+facture+email / payment_failed → restaure stock en transaction / canceled) ; ② anti-spoofing (`paymentIntentId` match) ; ③ idempotence (déjà PAID → return).
- **Notes orateur** : source de vérité (front attend confirmation serveur via polling, empêche simulation client) ; signature HMAC = authenticité (rawBody requis) ; anti-spoofing = pas de manip orderId ; idempotence = retries Stripe ; SkipThrottle (ne jamais rate-limiter un webhook) ; résilience = cron réconciliation (OrdersCleanupService) pour webhooks perdus.
- **Visuels** : extrait annoté (fond clair) + ✅ `annexe-5-sequence-commande.png` (séquence tunnel avec webhook).

### Slide 22 — Diagramme d'états : cycle de vie commande ✅ `CP3` `CP6`
- **Message clé** : cycle de vie modélisé en machine à états explicite, transitions contrôlées dans le code (pas d'état incohérent). Pont conception↔implémentation.
- **Texte de la slide** : 2 machines parallèles. OrderStatus : PENDING→CONFIRMED(webhook payé)→PLANTED(admin, final) ; PENDING/CONFIRMED→CANCELLED. PaymentStatus : PENDING→PROCESSING(stock réservé)→PAID(webhook)→REFUNDED(annul. admin) ; PROCESSING→FAILED(webhook échec, stock restauré, retry). Lien : webhook pilote PaymentStatus → déclenche OrderStatus.
- **Notes orateur** : 2 dimensions d'état ; transitions autorisées via constante `ORDER_STATUS_TRANSITIONS` (orders.constants.ts), `updateOrderStatus` refuse le reste ; cas annulation admin = remboursement Stripe + restauration stock + REFUNDED en transaction ; pont vers enums Prisma (13), confirmCheckout (20), webhook (21) ; pattern State machine.
- **Visuel** : ✅ `corps-etats-commande.png`.

### Slide 23 — Données & SQL : transaction ORM ↔ SQL ✅ `CP8`
- **Message clé** : prouver qu'on comprend le SQL derrière l'ORM — la transaction confirmCheckout en Prisma ET en SQL équivalent (attente explicite de Benoît).
- **Texte de la slide** (2 colonnes fond clair) : Prisma (`$transaction`, `findUnique` include imbriqués, `treeStock.update decrement`, `order.update`) | SQL (BEGIN ; SELECT + JOINs orders/order_items/trees/stock ; UPDATE stock quantity-X ; UPDATE orders ; COMMIT).
- **Notes orateur** : `$transaction`=BEGIN…COMMIT/ROLLBACK (atomicité) ; include=JOINs ; decrement=UPDATE atomique (pas de race condition) ; requêtes paramétrées `$1,$2` → injection SQL impossible (OWASP A03) ; `$queryRaw` dispo pour SQL pur.
- **Visuel** : 2 colonnes Prisma | SQL côte à côte. Tables réelles : `orders`, `order_items`, `trees`, `stock` (TreeStock @@map("stock"), col `tree_id`/`quantity`).

### Slide 24 — Données NoSQL : Redis ✅ `CP8`
- **Message clé** : Redis couvre l'exigence NoSQL de CP8 avec 3 cas concrets (bon outil clé/valeur en mémoire).
- **Texte de la slide** : ① Cache API cache-aside (pays TTL 24h, catégories TTL 1h + invalidation écriture, dégradation gracieuse → fallback BDD) ; ② Brute force login (compteur IP/email + TTL) ; ③ Blacklist JWT (token blacklisté au logout, TTL = durée restante, vérif chaque requête).
- **Notes orateur** : Redis NoSQL clé/valeur en mémoire (~0.1ms) ; cache-aside (miss→BDD→remplit cache) ; dégradation gracieuse (cache = optimisateur, pas SPOF) ; brute force complète le rate-limit ; blacklist résout la non-révocabilité du JWT stateless. Couvre CP8 SQL+NoSQL.
- **Visuel** : 🔧 schéma 3 usages Redis (cache-aside + brute force + blacklist). Optionnel : extrait cache-aside.

## BLOC 4 — Passages obligatoires

### Slide 25 — Sécurité : la démarche (OWASP / ANSSI) ✅ `transverse`
- **Message clé** : sécurité traitée par démarche structurée (audit OWASP Top 10:2025 + ANSSI en Sprint 3, cycle amélioration continue). À DISTINGUER de slide 11 (DICP = by design/conception) ; ici = démarche d'audit.
- **Texte de la slide** : organisée par menace (pas par techno) ; référentiels OWASP Top 10:2025 + ANSSI ; audit Sprint 3 backend ; cycle identifier→corriger→re-vérifier.
- **Notes orateur** : audit backend Sprint 3 ; exemple cycle veille→action (tokens reset en clair → audit → hashés) ; honnêteté (audit surtout backend, frontend = axe, mais XSS majeur traité via cookie HttpOnly) ; annonce slides 26-28 (protections par faille). Réf : `SECURITY_AUDIT_BACKEND.md`.
- **Visuel** : 🔧 grille OWASP Top 10:2025 + logos ANSSI/OWASP + cycle audit→correctif→re-vérif.

### Slide 26 — Sécurité : authentification ✅ `CP3`
- **Message clé** : authentification sécurisée en profondeur (le « second fil » auth, comme vecteur de sécurité).
- **Texte de la slide** : JWT cookie HttpOnly+Secure+SameSite=Lax (migration localStorage → anti-XSS) · bcrypt 10 rounds · tokens reset/vérif hashés · blacklist JWT Redis (logout) · brute force Redis (IP/email) · Passport 2 stratégies (Local/Jwt).
- **Notes orateur** : décision clé cookie HttpOnly (token illisible au JS = XSS neutralisé, SameSite = anti-CSRF) ; bcrypt salt+hash ; tokens hashés (fuite BDD inexploitable) ; blacklist révoque le JWT ; brute force ; Passport pattern Strategy (Local login / Jwt routes protégées, guard décide).
- **Visuel** : ✅ `annexe-6-sequence-authentification.png`. Optionnel : extrait `getCookieOptions()` / `bcrypt.compare`.

### Slide 27 — Sécurité : injection SQL & XSS ✅ `transverse`
- **Message clé** : deux failles majeures neutralisées à plusieurs niveaux (défense en profondeur).
- ⚠️ **SANS CODE sur la slide** (préférence Tony) — concepts uniquement, le « où dans le code » en notes orateur.
- **Texte de la slide** : ① Injection SQL (OWASP A03) : Prisma requêtes paramétrées · ValidationPipe (whitelist/forbidNonWhitelisted) · DTO class-validator. ② XSS : React échappe par défaut · Helmet (CSP + headers) · Zod client · JWT cookie HttpOnly (token non volable).
- **Notes orateur** : Prisma paramétré (pas de concaténation, cf. slide 23) + ValidationPipe nettoie/rejette ; React échappe (pas de dangerouslySetInnerHTML), Helmet CSP, cookie HttpOnly = token protégé même si XSS ; défense en profondeur (client→API→ORM→headers).
- **Visuel** : 🔧 schéma 2 failles × leurs couches de protection (icônes, PAS de code).

### Slide 28 — Sécurité : CSRF & Broken Access Control ✅ `transverse`
- **Message clé** : contrôler qui peut faire quoi — CSRF (requêtes forgées) + Broken Access Control (risque n°1 OWASP). SANS CODE.
- **Texte de la slide** : ① CSRF : CORS restrictif (origine unique via FRONTEND_URL) + cookie SameSite=Lax (pas d'envoi cross-site). ② Broken Access Control (A01) : guards par rôle (JwtAuthGuard+RolesGuard USER/ADMIN) + vérification propriétaire (order.userId===userId) + @Roles/@CurrentUser.
- **Notes orateur** : CSRF = requête à l'insu depuis un autre site → CORS + SameSite ; BAC = risque #1 OWASP, 2 niveaux (rôle + propriétaire) ; suite logique de l'auth (qui tu es vs ce que tu as le droit de faire).
- **Visuel** : 🔧 schéma CSRF (bloqué SameSite/CORS) + contrôle d'accès (rôles + propriétaire). Sans code.

### Slide 29 — Tests : stratégie ✅ `CP9`
- **Message clé** : stratégie de tests unitaires ciblée sur la logique critique, honnête sur ses limites (ni intégration ni E2E). SANS code (conceptuel).
- **⚠️ Correction** : **QUE des tests unitaires** (pas de pyramide, pas d'intégration, pas d'E2E).
- **Texte de la slide** : Fait = 211 tests unitaires (151 back Jest / 60 front Vitest+TL), priorité logique critique (checkout/paiement/panier), CI sur chaque PR. Manque (assumé) = intégration (BDD test), E2E (Cypress/Playwright), seuil de couverture.
- **Notes orateur** : choix pragmatique (4 sprints) ; connaître les concepts manquants (intégration = flux réel + BDD ; E2E = parcours navigateur/non-régression) = axes prioritaires ; outils Jest/Vitest+TL. Chiffre 211 à confirmer.
- **Visuel** : 🔧 2 blocs « Unitaires 211 ✅ / Intégration-E2E à venir » (PAS de pyramide). Capture Jest+Vitest (annexe F).

### Slide 30 — Jeu d'essai : tunnel de vente ✅ `CP9`
- **Message clé** : fonctionnalité critique couverte par un jeu d'essai complet (nominal + erreurs). Jeu d'essai fonctionnel attendu par CP9.
- **Texte de la slide** : ① tableau jeu d'essai (ajout panier, stock insuffisant, épuisé, paiement réussi/échoué webhook, signature invalide, accès autre user → Forbidden, panier expiré) ; ② 1 test réel court (cartStore.syncWithStock : addItem 5, stock 3 → quantité ajustée à 3).
- **Notes orateur** : couvre nominal + cas d'erreur ; pas d'écart ; le test montré = arrange/act/assert ; tests en CI sur chaque PR. Code : UNIQUEMENT ici dans le bloc tests (validé par Tony).
- **Visuel** : tableau jeu d'essai + 1 extrait de test (fond clair). Capture Jest/Vitest (annexe F).

### Slide 31 — RGPD ✅ `transverse`
- **Message clé** : RGPD concret + arbitrage anonymiser plutôt que supprimer (concilie effacement art.17 et conservation comptable). SANS code.
- **Texte de la slide** : ① Effacement (art.17) = anonymisation (pas cascade delete) : identifiants → génériques (« Utilisateur supprimé », deleted@anonymized.local), comptable conservé, transaction atomique 4 étapes. ② Portabilité (art.20) = export JSON (`GET /auth/me/export`). ③ Minimisation : safeSelect (jamais password/token), pages légales, pas de tracking → pas de bannière cookies.
- **Notes orateur** : anonymisation vs cascade (comptabilité) ; transaction atomique ; portabilité JSON ; safeSelect ; pas de tracking = exemption ePrivacy ; facturation CGI art.289 (lien slide 13).
- **Visuel** : 🔧 schéma anonymisation avant/après (Jean Dupont → Utilisateur supprimé, comptable 🔒 conservé) + pictos export/minimisation. Sans code.

## BLOC 5 — Déploiement & DevOps

### Slide 32 — Conteneurisation (Docker + Makefile) ✅ `CP1` `CP10`
- **Message clé** : environnement reproductible et documenté (Docker Compose dev/prod + Makefile + multi-stage). Base CP1, amorce CP10.
- **Texte de la slide** : ① Docker Compose dev 7 services (front, back, PostGIS, Redis, Nginx, Stripe CLI, Prisma Studio, volumes/hot reload) vs prod 4 services (expose au lieu de ports, healthchecks). ② Makefile ~20 commandes (make dev/migrate/seed/lint). ③ Multi-stage (Alpine, non-root) + Nginx gzip/cache 1 an + validation Joi env (fail-fast).
- **Notes orateur** : reproductibilité (fini « ça marche chez moi ») ; dev optimisé dev vs prod durci ; make dev = onboarding 1 commande ; multi-stage = image légère + surface réduite ; Joi fail-fast.
- **Visuel** : 🔧 schéma services dev(7)/prod(4) + liste commandes make. Sans code.

### Slide 33 — CI/CD : 3 workflows GitHub Actions ✅ `CP11`
- **Message clé** : chaîne CI/CD automatisant qualité + déploiement (3 workflows). Cœur CP11.
- **Texte de la slide** : ① pr-checks (PR) : lint + tests + npm audit critical (garde-fou merge). ② develop-ci (develop) : suite étendue. ③ production-deploy (main) : build Docker → webhook Coolify → health check HTTP.
- **Notes orateur** : CI = lint/tests/audit à chaque PR (bloque le merge) ; CD = build + déploiement Coolify + health check ; 3 étages PR/develop/prod ; interprétation des rapports CI ; audit-level=critical justifié. Lien slide 34.
- **Visuel** : 🔧 schéma pipeline (PR→checks→merge→build→deploy→health check). Sources `.github/workflows/`.

### Slide 34 — Déploiement prod : Hetzner + Coolify ✅ `CP10` `CP11`
- **Message clé** : déploiement réel fait de bout en bout par Tony (VPS + PaaS auto-hébergé) + sauvegarde/restauration. À valoriser.
- **Texte de la slide** : infra VPS Hetzner + Coolify (orchestre Docker) · domaine sslip.io + Nginx · pipeline production-deploy → webhook Coolify → build/deploy → health check · backup pg_dump/pg_restore · sécu infra (HTTPS, expose, non-root, Joi).
- **Notes orateur** : « déploiement réalisé de A à Z moi-même » (argument fort) ; Coolify = Heroku maison ; backup CP7 aussi. **Honnêteté** : reste à câbler le webhook Stripe en prod (OK en local via Stripe CLI), backup validé en local → à finaliser avant le passage. CP10 (procédure documentée) + CP11 (mise en prod).
- **Visuel** : 🔧 schéma infra (Hetzner→Coolify→conteneurs→Nginx→HTTPS) + captures Coolify/app live (Tony).

## BLOC 6 — Conclusion

### Slide 35 — Perspectives / axes d'amélioration ✅
- **Message clé** : projet quasi production-ready, manques identifiés et priorisés (posture mature).
- **Texte de la slide** : Tests (intégration + E2E Cypress/Playwright + seuil couverture) · Fonctionnel (carte interactive PostGIS, portail partenaires, parrainage, multilingue) · Technique (SSE vs polling, sync panier serveur, SSR pour SEO, Husky) · Prod (finaliser webhook Stripe, monitoring).
- **Notes orateur** : ne pas tout lire, piocher 3-4 exemples ; « MVP solide livré, voici ce qu'on ferait avec plus de temps » ; montre la priorisation.
- **Visuel** : 🔧 liste thématique (Tests/Fonctionnel/Technique/Prod).

### Slide 36 — Bilan personnel ✅
- **Message clé** : reconversion aboutie + aboutissement concret.
- **Texte de la slide** : montée en compétence full-stack (conception→prod) · travail Agile/rigueur · **aboutissement = CDI Groupe LDLC, Concepteur Intégrateur IA (depuis avril 2026)**.
- **Notes orateur** : bilan humain sincère ; le CDI = boucle bouclée (rappel slide 2 « concevoir puis construire ») ; ton perso.
- **Visuel** : sobre, phrase forte + éventuellement frise slide 2 se terminant sur le CDI.

### Slide 37 — Merci / clôture ✅
- **Message clé** : signal de fin explicite (Benoît).
- **Texte de la slide** : « Merci de votre attention. » + « Je suis à votre disposition pour vos questions. »
- **Notes orateur** : clôture nette pour passer la main au jury, pas de blanc. Debout, posé.
- **Visuel** : logo GreenRoots + « Merci ».

## 🎁 Slides BONUS (après conclusion, pour les questions)
Veille techno (npm audit avant/après, Mozilla Observatory) · Dictionnaire de données · Facture PDF (InvoiceService, TVA, CGI art.289) · Cloudinary upload signé · Passport Strategy (Local vs JWT) · Comparatifs SPA/SSR/Serverless & TanStack vs React Router · Diagramme d'activité CRON (`annexe-7`).

## État global
- **Contenu des 37 slides + bonus : VALIDÉ et transcrit.** ✅ (Blocs 1 à 6 complets)
- **Prochaine étape** : construction du support (template + visuels à produire) + captures/exports à faire par Tony.

> Voir le plan complet en mémoire (`plan-oral-greenroots`) et la couverture des 11 CPs.
