# Référentiel reveal.js - support oral GreenRoots

> Compilé depuis la doc officielle (revealjs.com) pour ne pas dépendre de la mémoire.
> Sert de catalogue des capacités + conventions du projet. reveal.js **5.1** (offline dans `vendor/`).
> Design : voir `BRIEF-DESIGN-ORAL.md`. Contenu des slides : `docs/ORAL-CONTENU-SLIDES.md`.

## Fichiers du projet
- `index.html` = le deck. `theme.css` = surcouche GreenRoots. `vendor/white.css` = thème officiel (base).
- `vendor/` : reveal.js 5.1, reset.css, reveal.css, highlight.js (+ code-light.css), notes.js. `fonts/` : Montserrat. `assets/logo.png`.
- Ordre CSS : reset → reveal → white → code-light → theme.

## Rendu / prévisualisation (MÉTHODE FIABLE)
- **Présenter** : ouvrir `index.html` dans Chrome. `F` plein écran · `S` speaker view · `Esc` overview · `G` jump-to-slide · `Alt+clic` zoom.
- ⚠ **Le deep-link headless `#/N` est NON fiable** (slides parfois blanches = artefact). Pour valider un rendu :
  - **1 slide = 1 fichier isolé** (extraire la `<section>`, wrapper avec le boilerplate, révéler fragments via `.classList.add('visible')`, PAS `Reveal.next()`).
  - ou **`?print-pdf`** → `--print-to-pdf` (déterministe) puis lire le PDF.
- Chrome headless ne se termine pas seul sur reveal : lancer en tâche de fond, attendre le PNG, `kill`.
- PNG type : `chrome --headless --disable-gpu --force-device-scale-factor=2 --virtual-time-budget=2800 --window-size=1280,720 --hide-scrollbars --screenshot=out.png --user-data-dir=<tmp> file://…`.

## Config (`Reveal.initialize({...})`)
- Dimensions : `width:1280, height:720, margin:0.06`. `center:true` (centrage vertical natif).
- Navigation : `hash:true`, `slideNumber:'c/t'`, `navigationMode:'default'|'linear'|'grid'`, `jumpToSlide:true`.
- Transitions : `transition:'slide'`, `transitionSpeed:'default'|'fast'|'slow'`, `backgroundTransition:'fade'`.
- Auto-animate : `autoAnimateDuration:1.0`, `autoAnimateEasing:'ease'`, `autoAnimateUnmatched:true`.
- Vue : `view:'scroll'` + `scrollProgress`, `scrollSnap`, `scrollLayout`, `scrollActivationWidth`.
- Notes/PDF : `showNotes:true|'separate-page'`, `pdfMaxPagesPerSlide:1`, `pdfSeparateFragments:false`, `defaultTiming`, `totalTime`.
- Média : `autoPlayMedia:null|true|false`, `viewDistance` (lazy-load), `preloadIframes`.
- Plugins chargés : `RevealHighlight`, `RevealNotes`.

## Structure des slides
- Un slide = `<section>` dans `.slides`.
- **Verticales** (grouper / approfondir à la demande) : imbriquer des `<section>` dans un `<section>` parent → nav ↓.
  ```html
  <section>Horizontale</section>
  <section>
    <section>Verticale 1</section>
    <section>Verticale 2</section>
  </section>
  ```
- `navigationMode` : `default` (H = ←→, V = ↓↑) · `linear` (tout en ←→) · `grid`.

## Fragments (apparition progressive)
```html
<p class="fragment">fade-in (défaut)</p>
<p class="fragment fade-up">monte en apparaissant</p>       <!-- fade-down/left/right -->
<p class="fragment fade-out">visible puis disparaît</p>
<p class="fragment fade-in-then-out">apparaît puis part à l'étape suivante</p>
<p class="fragment fade-in-then-semi-out">…puis 50% d'opacité</p>
<p class="fragment grow">grossit</p>  <p class="fragment shrink">rétrécit</p>
<p class="fragment strike">barré</p>  <p class="fragment semi-fade-out">passe à 50%</p>
<span class="fragment highlight-red">rouge</span>          <!-- highlight-green/blue -->
<span class="fragment highlight-current-green">vert le temps d'une étape</span>
```
- Ordre : `data-fragment-index="1"` (plusieurs éléments peuvent partager un index).
- **Imbriqués** : empiler des `.fragment` pour enchaîner les effets sur un même contenu.
- **Custom** (v4.5+) : `.fragment.custom.monEffet` + `.fragment.custom.monEffet.visible` en CSS.
- Événements : `Reveal.on('fragmentshown'|'fragmenthidden', e => e.fragment)`.

## Auto-Animate (morphing entre 2 slides)
```html
<section data-auto-animate><h1>Titre</h1></section>
<section data-auto-animate><h1 style="color:red">Titre</h1></section>   <!-- même texte → morph -->
```
- **Appariement** : texte + type de nœud identiques ; média par `src` ; sinon `data-id="x"`.
- Attributs : `data-auto-animate-duration`, `-delay`, `-easing`, `-restart` (coupe la chaîne), `-unmatched="false"` (ne pas fade-in les non-appariés), `data-auto-animate-id` (grouper des séquences).
- **Code ligne à ligne** : deux `<pre data-id="x"><code data-line-numbers>` → reveal anime l'ajout/déplacement de lignes.
- **Listes** : les `<li>` s'animent individuellement (ajout/retrait).
- Événement : `Reveal.on('autoanimate', e => …)`. Chez nous : construire le tunnel + faire grossir un KPI.

## Code (coloration + focus pas-à-pas)
```html
<pre><code class="language-typescript" data-trim data-line-numbers="1-6|2-3|9" data-ln-start-from="12">
…
</code></pre>
```
- `data-trim` (retire les espaces) · `data-noescape` (garde `<` `>`) · `class="language-…"`.
- `data-line-numbers` : numéros. Valeur = lignes surlignées (`"3,8-10"`). **Étapes** séparées par `|` (chaque `|` = un clic ; les lignes hors focus passent à 40 % d'opacité).
- `data-ln-start-from="12"` : décale la numérotation (extrait réel).
- Code avec `<>` : envelopper dans `<script type="text/template">…</script>`.
- Thème highlight : **github clair** (`vendor/code-light.css`).

## Layout helpers (natifs)
- `class="r-fit-text"` : texte auto-dimensionné au max sans déborder (titres géants, chiffres).
- `class="r-stretch"` : média qui remplit la hauteur restante (1 seul, descendant direct du slide).
- `class="r-stack"` : empile des éléments centrés au même endroit (+ fragments → superposer/permuter des visuels, ex. wireframe → maquette).
- `class="r-frame"` : cadre décoratif (effet hover si dans un `<a>`).

## Backgrounds (par slide, sur `<section>`)
- `data-background-color="#14261C"` (intercalaires).
- `data-background-gradient="linear-gradient(...)"` (linear/radial/conic).
- `data-background-image="x.png"` + `-size` (cover) / `-position` / `-repeat` / `-opacity`.
- `data-background-video="x.mp4"` + `-video-loop` `-video-muted` `-opacity`.
- `data-background-iframe="url"` + `data-background-interactive` (embarquer l'app live !).
- `data-background-transition="fade|slide|zoom|convex|concave|none"`.
- Parallax : config `parallaxBackgroundImage/-Size/-Horizontal/-Vertical`.

## Média
- **Autoplay** : `<video data-autoplay>` (ou config `autoPlayMedia:true`). Pause auto en quittant le slide (sauf `data-ignore`).
- **Lazy-load** : `data-src` au lieu de `src` (charge selon `viewDistance`). Idem `<iframe data-src>`.
- **YouTube/Vimeo** : autoplay auto détecté. Iframes plein écran → utiliser un background iframe.
- **Lightbox** : `data-preview-image` / `-video` / `-link` pour ouvrir en plein écran au clic.
- L'iframe reçoit `slide:start` / `slide:stop` en postMessage.

## Transitions
- Global : `transition:'none|fade|slide|convex|concave|zoom'` + `backgroundTransition`.
- Par slide : `data-transition="zoom"` · `data-transition-speed="fast|slow"` · entrée/sortie séparées `data-transition="slide-in fade-out"`.

## Vues & navigation
- **Scroll view** (reveal 5) : `view:'scroll'` (ou `?view=scroll`) → deck en page scrollable (lecture autonome, mobile). Options : `scrollProgress`, `scrollSnap:'mandatory'|'proximity'|false`, `scrollLayout:'full'|'compact'`.
- **Jump-to-slide** : `G` → taper un n° (`5`), un `6/2`, ou un id (`the-end`) → Entrée. Config `jumpToSlide`.
- **Overview** : `Esc` (vue d'ensemble en grille).

## Speaker view / notes / timing
- **`S`** → fenêtre séparée : notes + preview slide suivante + chrono écoulé + horloge + **pacing** (vert/rouge/bleu).
- Notes : `<aside class="notes">…</aside>` ou `data-notes="…"` sur la `<section>`.
- Timing : config `defaultTiming` (s/slide), `totalTime`, ou `data-timing` par slide.
- `showNotes:true` affiche les notes pour tous ; `'separate-page'` pour le PDF.
- Dual-screen : étendre l'affichage → deck sur le projecteur, speaker view sur le Mac (synchronisés).

## Export PDF (filet de sécurité)
1. Ouvrir `index.html?print-pdf` dans **Chrome**.
2. Cmd+P → « Enregistrer au format PDF » → **Paysage** → Marges **Aucune** → cocher **Graphiques d'arrière-plan**.
- Config : `pdfMaxPagesPerSlide:1`, `pdfSeparateFragments:false`, `showNotes:'separate-page'`.
- CLI : `decktape`. Chrome/Chromium uniquement.

## Conventions GreenRoots
Voir **`BRIEF-DESIGN-ORAL.md`** (source de vérité design). En bref : thème `white` officiel + surcouche · Montserrat + vert · center:true · en-tête (eyebrow+titre) en haut, corps centré · jamais `—` (→ `-`) · pas d'emojis (→ icônes Radix UI vertes) · pas de CP sur slides · logo réel · animations sobres (fragments, focus code, auto-animate réservé au tunnel + KPI).
