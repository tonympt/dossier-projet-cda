# Brief de design - Support oral GreenRoots (Titre pro CDA)

> **Source de vérité du design du deck reveal.js.** À relire avant de construire/modifier une slide.
> Le brief fixe le cadre et les contraintes ; l'exécution créative (compositions, choix visuels dans le cadre) est libre.
> Contenu texte des 37 slides = `docs/ORAL-CONTENU-SLIDES.md` (validé). Ce brief ne duplique pas le contenu, il définit la forme.

---

## 1. Rôle & objectif

- **Rôle** : graphiste / infographiste qui conçoit un support de présentation professionnel.
- **Livrable** : deck **reveal.js** projeté devant un jury (Titre professionnel Concepteur Développeur d'Applications), ~**40 min**, **37 slides** + bonus, structuré en **6 blocs**.
- **Public** : jury CDA (technique mais généraliste). Objectif : servir le propos de Tony, prouver la maîtrise, valoriser le projet et le candidat.
- **Fil rouge** de l'oral : le tunnel de vente. Démo live en ouverture du bloc 3.
- **Ton visuel** : sobre, professionnel, éditorial, cohérent avec l'app GreenRoots. Pas de « wow » gratuit.

## 2. Base technique (VERROUILLÉE - ne pas réinventer)

- **reveal.js 5.1** en local (`vendor/`), 100 % offline. Plugins chargés : `RevealHighlight`, `RevealNotes` uniquement.
- **Thème = officiel `white`** (`vendor/white.css`, import de police externe retiré) **+ surcouche légère** `theme.css` qui NE surcharge QUE les variables officielles `:root --r-*` + quelques classes de marque. On s'appuie sur le natif ; on ne construit PAS de framework CSS maison.
- **Deck réel** = `index.html` (un seul fichier). `_archive/` = ancien travail sur-ingénié, à ignorer.
- **Config reveal** : `width:1280, height:720` (16:9), `margin:0.08` (zone sûre anti-overscan), `center:false` (contenu ancré en haut ; les héros #cover/.divider/.closing sont recentrés en flex), `hash:true, slideNumber:'c/t', transition:'slide', backgroundTransition:'fade'`.
- **Conditions de présentation** : écran 55" ou moins, 16:9 (1920×1080 ou 4K). Le deck ne rogne pas (ratio conservé) ; tout rognage = **overscan TV** → régler la TV en « Adapter à l'écran / Just Scan / PC mode » + Chrome plein écran (F). Captures/diagrammes en **≥1920 px** (2× si 4K). JAMAIS d'unités `vh` à l'intérieur d'une slide (cassent sous l'auto-échelle).
- **Polices** : Montserrat (woff2 dans `fonts/`, offline) - la police de l'app.
- **Assets** : `assets/logo.png` (badge GreenRoots 3 feuilles). Diagrammes dans `../diagrammes/`.

### Capacités reveal.js à exploiter (natif)
Slides horizontales + verticales (approfondir à la demande) · fragments · **auto-animate** (dont code ligne à ligne) · focus de code par étapes (`data-line-numbers="1|2-3"`) · fonds couleur/image/vidéo/**iframe** (app live possible) · notes orateur + minuteur (touche S) · overview (Esc) · export **PDF** · transitions. Rédaction possible en Markdown.

## 3. Charte & tokens

Définis dans `theme.css :root`. Ne pas introduire d'autres couleurs sans raison.

| Token | Valeur | Usage |
|---|---|---|
| `--gr-ink` | `#14261C` | texte principal, fond des intercalaires |
| `--gr-green` | `#2E7D4F` | accent principal |
| `--gr-deep` / `--r-link-color-dark` | `#265247` | `<strong>`, accents foncés |
| `--gr-soft` `#E7F3E1` / `--gr-softer` `#F1F8EE` | tints clairs (fonds de cartes, onglets) |
| `--gr-glow` | `#8FD6A8` | accents sur fond sombre (intercalaires) |
| `--gr-muted` | `#5C6B61` | texte secondaire (⚠ jamais pour du texte important : contraste) |
| `--gr-line` | `#D7E6D9` | filets, bordures |
| `--r-background-color` | `#FBFDFB` | fond (blanc à biais vert) |
| danger `#C7443A` · warn `#B9770E` | alertes (callouts sécurité) |

**Typo** : Montserrat unique. Poids : **700** titres · **600** sous-titres/eyebrow/labels · **500** chapô/footer · **400** corps. Titres en **casse normale** (pas de MAJ). Échelle via les variables `--r-heading*`. Le **chemin de fichier du code** est en **monospace** (allure machine à écrire), pas en Montserrat.

### Compléments de charte
- **Échelle d'espacement** (rythme vertical constant) : 8 / 16 / 24 / 40 / 64 px. Réutiliser ces pas partout.
- **Icônes** : **`react-icons`** pour toute la prez (on abandonne le principe « Radix UI uniquement »). Set privilégié = **Phosphor** (`pi`, poids *fill* pour un rendu bold), cohérent avec le reste. SVG **inliné** (deck 100 % offline → jamais de CDN/package), couleur verte `--gr-green`, épaisseur homogène. Garder UN set dominant, ne pas mélanger au hasard.
- **Transitions** : `slide` pour les slides, `fade` pour les fonds. Auto-animate réservé (tunnel + KPI). **Bannir** zoom/convex/concave (effet « IA »).
- **Cadrage des captures d'écran** : rendu unique = coins arrondis (~10px) + ombre douce + filet 1px `--gr-line`. Optionnel : barre « navigateur » sobre au-dessus. Toutes les captures (Lighthouse, Figma, responsive, Coolify…) traitées pareil.
- **Diagrammes** : fond blanc, même radius/ombre que les captures, marges homogènes → cohérence des PNG de `diagrammes/`.
- **Palette dataviz** (si graphiques) : séries en `#2E7D4F` / `#8FD6A8` / `#265247` ; neutres `#5C6B61` / `#D7E6D9`. Pas d'autres teintes.
- **Accessibilité / contraste** : viser WCAG AA. `--gr-muted` **interdit** pour un texte important (réservé au secondaire). Taille de corps jamais < ~20 px projeté.
- **Typographie FR** : guillemets « … » (avec espaces insécables), espaces insécables avant `: ; ! ?`, et **jamais `—`** (→ `-`).
- **Do / Don't** : ✅ blanc + vert, layouts natifs, 1 idée/slide, logo réel. ❌ blob/formes dessinées, barres décoratives, emojis, dégradés gratuits, tags CP, texte pâle illisible, slides surchargées.

## 4. Principes & RÈGLES STRICTES

Principes :
- **RÈGLE #1 - Aération / respiration** : toujours du **padding sous le titre** en haut de la slide ; le **corps occupe l'espace restant** (via flex + `gap`, jamais tassé). **Toujours du gap ENTRE les éléments** - jamais compacté. Composition équilibrée, cohérente, jamais surchargée ni vide-déséquilibrée. C'est la règle prioritaire de toutes les slides.
- **Espacement ADAPTATIF, pas de marges fixes géantes** : pour espacer des zones haut/milieu/bas, laisser **flexbox répartir** (`.slide-body.spread` = `justify-content:space-evenly`) plutôt que d'empiler des `margin` fixes (ex. `8em`) qui font déborder le bloc et **compriment** le contenu. Marge minimale sous le titre, le reste s'adapte à la hauteur dispo.
- **Notes orateur (`<aside class="notes">`)** : TOUJOURS en **bullet points** (`<ul><li>`), idées principales **en gras** (`<strong>`) - pour que Tony s'y réfère d'un coup d'œil en speaker view. Vaut pour TOUTES les slides. Alimentées par les « notes orateur » de `docs/ORAL-CONTENU-SLIDES.md`.
- **1 idée par slide.** Le support illustre, l'orateur explique.
- **En-tête ancré en haut** : l'**eyebrow** (label de section, ex. « Conception ») + le **titre de slide** sont toujours en HAUT de la slide. Le **contenu principal** est ensuite **centré verticalement** dans l'espace restant. (Donc : titres en top, corps au milieu - pas tout centré en bloc.)
- **Remplir la slide** : composition équilibrée, pas de contenu tassé avec du vide déséquilibré.
- **Aérer** à l'intérieur : respiration entre les éléments, jamais surchargé.
- **Lisibilité projecteur** avant tout : gros texte, fort contraste (jamais de texte important en `--gr-muted` trop pâle).
- **Natif d'abord** : utiliser les layouts/possibilités de reveal avant d'inventer du CSS.
- **Cohérence produit ↔ support** : mêmes couleurs, police et icônes que l'app.
- **Éviter l'effet « IA »** : sobriété, pas de dégradés/animations gratuits.

Règles strictes (non négociables) :
- **Jamais de tiret cadratin `—`** dans le contenu visible → toujours `-`.
- **Pas d'emojis** → icônes **`react-icons`** (set privilégié **Phosphor** `pi`, poids *fill*), inlinées en SVG, colorées en vert. Un set dominant, pas de mélange au hasard.
- **Toujours "natif d'abord"** : avant de créer un composant CSS (tableau, citation, liste, code…), VÉRIFIER ce que `vendor/white.css` et reveal.js stylent déjà, et bâtir une surcouche légère par-dessus. Ne pas réinventer.
- **Pas de tags CP** sur les slides (c'est le jury qui décide des compétences).
- **Logo réel** (`assets/logo.png`) pour la marque ; pas de forme dessinée/blob.
- **Code** toujours sur fond clair.

## 5. Layouts réutilisables (le système)

Chaque slide se rattache à un archétype. Tenue graphique constante d'un archétype à l'autre.

| Archétype | Quand | Détails |
|---|---|---|
| **Cover** | garde | marque (badge vert + feuille Phosphor + « GreenRoots ») en HAUT-GAUCHE ; grand titre **PRÉSENTATION DE PROJET** (majuscules, accent É) au centre ; sous-titre soutenance + **nom** + **date** (icônes Phosphor *fill*, nom/date même couleur encre) ; **feuille Phosphor unique** en filigrane à droite. Aligné gauche. |
| **Intercalaire** (`.divider`) | entre blocs | `data-background-color="#14261C"`, grand titre blanc **EN MAJUSCULES**, **fil d'Ariane** des 6 blocs (courant en évidence). **Sans libellé « BLOC »**. Footer/n° masqués (`.bare`). |
| **Contenu + cartes** | 2-4 items parallèles | `.cards.c2/.c3`, rôle + titre + phrase courte, icône Radix UI optionnelle. |
| **Contenu + liste** | énumération | liste native, `.fragment` pour révéler. |
| **Code** | extraits | `<pre><code data-line-numbers>` natif + onglet fichier `.code-file` (arbo) + focus par étapes ; légende ①②③ possible. |
| **Figure** | diagrammes | image centrée (`.figure`), ombre douce. |
| **KPI** | chiffres-clés | `.kpi-row`, gros nombre vert + libellé. |
| **2 colonnes** | comparaison | `.split` (ex. Prisma \| SQL). |
| **Tableau** | comparatif | `<table>` stylé (MVP/évolutions…). |
| **Callouts** | nuances/sécurité | encadrés info / attention / danger avec icône. |
| **Statement** | accroche/bilan | phrase forte centrée, peu de texte. |
| **Clôture** | fin | fond vert foncé, logo, « Merci ». |

Animations : sobres. Fragments pour puces/cartes ; focus de code par étapes ; **auto-animate réservé** au tunnel de vente + chiffres-clés (2-3 max sur tout le deck).

### Détail - Intercalaire (validé)
- Fond `#14261C`. **Titre du bloc tout en haut** (grand, blanc, **EN MAJUSCULES**).
- **Gros numéro du bloc** (ex. « 03 ») en filigrane à **droite**, grande taille, faible opacité.
- Sous le titre : **mini-sommaire** = les **titres des slides** du bloc (petite liste, façon table des matières du bloc).
- **Fil d'Ariane des 6 blocs tout en bas** (bottom de la slide) :
  - blocs **déjà passés** = gris, opacité moindre ;
  - blocs **à venir** = blanc ;
  - bloc **courant** = vert glow (`--gr-glow`).
- Pas de libellé « BLOC ». Footer/n° masqués (`.bare`).

### Détail - En-tête de slide de contenu
- L'**eyebrow** (label de section) puis le **titre** en HAUT ; le corps centré verticalement dessous (cf. §4).

### Détail - Code
- Bloc code natif reveal, fond clair.
- **Onglet d'arborescence collé au sommet** de l'encart code (aucun espace ; l'onglet coiffe le bloc, coins supérieurs raccordés).
- Chemin du fichier en **police monospace type machine à écrire** (allure « code »), pas en Montserrat.
- Focus par étapes (`data-line-numbers`), légende ①②③ optionnelle.

## 6. Workflow de rendu / validation (FIABLE)

⚠ Le **deep-link headless** (`index.html#/N`) est **non fiable** (slides parfois blanches = artefact, PAS un bug du deck). Ne jamais juger le rendu ainsi.

Méthodes fiables :
- **1 slide = 1 fichier autonome** : extraire la `<section>`, l'envelopper dans le boilerplate (reset→reveal→white→code-light→theme), rendre en isolé. Révéler les fragments via `.classList.add('visible')` (PAS `Reveal.next()` en auto : casse le bloc code).
- **Tout le deck** : `?print-pdf` → `--print-to-pdf` (déterministe, fragments déployés) puis lire le PDF.
- Chrome headless **ne se termine pas seul** sur une page reveal : lancer en tâche de fond, attendre le PNG, puis tuer le process (`kill`).
- Commande PNG type : `chrome --headless --disable-gpu --force-device-scale-factor=2 --virtual-time-budget=2800 --window-size=1280,720 --hide-scrollbars --screenshot=out.png --user-data-dir=<tmp> file://…`.
- Toujours **montrer le rendu** à Tony avant de considérer une slide validée.

## 7. Inventaire des 37 slides & assets

Structure et contenu détaillés dans `docs/ORAL-CONTENU-SLIDES.md` (6 blocs, 37 slides + bonus). Mapping slide → archétype à établir en mode plan.

Assets :
- **Existants** (`diagrammes/*.png`) : architecture 3 tiers, cas d'usage, MCD/MPD, états commande, séquences (commande, auth), parcours d'achat, wireframe catalogue, activité CRON.
- **À produire par moi** (schémas en HTML rendu) : DICP, 3 états front, 3 usages Redis, pipeline CI/CD, infra Hetzner/Coolify, anonymisation RGPD, sécurité (SQL/XSS, CSRF/BAC), services Docker.
- **Captures à la charge de Tony** : Lighthouse, maquette Figma HF, responsive table⇄cards, board Trello, Coolify, formulaire checkout (erreur + select pays).

## 8. Critères d'acceptation (une slide est « bonne » si…)

- Lisible à 3 m d'un projecteur (taille + contraste).
- Rattachée à un archétype, cohérente avec les autres.
- Respecte toutes les règles strictes (§4).
- Composition équilibrée (remplit la slide, aérée).
- Rendue et **montrée** via la méthode fiable (§6), pas de slide blanche.
- Sert le message clé de la slide (cf. `ORAL-CONTENU-SLIDES.md`).

---
_Références : `docs/ORAL-CONTENU-SLIDES.md` (contenu) · `REVEALJS-REFERENCE.md` (fiche technique reveal) · mémoire `support-oral-revealjs`, `support-oral-regles-design`._
