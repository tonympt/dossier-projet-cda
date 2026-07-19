# Télécommande téléphone (reveal.js-remote)

Contrôle du deck depuis le Pixel, **via le réseau** (pas Bluetooth). Le téléphone ouvre juste
une URL dans Chrome. Testé bout-en-bout le 18/07/2026 (compatible reveal.js 5.1).

## Le serveur
Installé (hors du repo, pour ne pas le polluer) dans :
`/Users/Admin/workspace/github.com/tonympt/reveal-remote-server`
(cloné de github.com/cologneintelligence/reveal.js-remote, déjà buildé : `npm ci && npx gulp`).

Le deck (`index.html`) est déjà câblé : 2 `<script>` (`/socket.io/...`, `/_remote/plugin/remote.js`)
+ un bloc `remote:{…}` dans `Reveal.initialize`, **avec garde** : si le deck est ouvert en
`file://` (double-clic), ces scripts échouent sans conséquence → **mode dégradé clavier**.

## Le jour J - procédure

1. **Réseau** : active le **partage de connexion (hotspot)** du Pixel, puis connecte le **PC** au
   hotspot du Pixel. (Le PC et le téléphone sont alors sur le même réseau, indépendant du WiFi du
   centre - c'est le plus fiable.)
2. **Lancer le serveur** sur le PC :
   ```sh
   /Users/Admin/workspace/github.com/tonympt/reveal-remote-server/start-greenroots.sh
   ```
   Il affiche l'**IP du PC** et l'URL de présentation.
3. **Ouvrir la présentation** dans Chrome sur le PC, à l'adresse **avec l'IP** (pas localhost) :
   `http://<IP-du-PC>:4242/support-oral/index.html`
   → plein écran avec `F`. (Exemple d'IP sur le réseau actuel : `192.168.1.73` - elle change selon
   le réseau ; le script te la redonne à chaque lancement.)
   > Important : ouvrir via l'IP (et pas `localhost`) car le QR code est construit à partir de
   > l'adresse utilisée - avec localhost, le téléphone ne pourrait pas joindre le PC.
4. **Appairer le téléphone** : dans la présentation, appuie sur la touche **`r`** → un **QR code**
   s'affiche → scanne-le avec l'appareil photo du Pixel → Chrome ouvre la **télécommande**
   (`http://<IP>:4242/_remote/ui/?<id>`) : gros boutons ◀ ▶, **slide courante + suivante**,
   **notes orateur** (tes `<aside class="notes">`) et **minuteur**.
5. Tu pilotes depuis le téléphone. (Touche `a` = partager la présentation à l'audience - non utilisé ici.)

## Mode dégradé (toujours dispo)
- Au **clavier** du PC : `→` / `Espace` / `Page↓` = suivant, `←` / `Page↑` = précédent,
  `F` = plein écran, `S` = **vue présentateur** (notes + slide suivante + chrono, fenêtre séparée),
  `B` ou `.` = écran noir, `Échap` = vue d'ensemble.
- Un **clicker USB/Bluetooth** (télécommande de présentation) marche aussi : il envoie ces mêmes
  touches, aucune config.

## Vérifier avant le jour J
- Refaire l'étape 1→4 une fois chez toi (hotspot Pixel), scanner le QR, cliquer suivant/précédent.
- Si le QR ne s'affiche pas : vérifier que l'URL de la présentation utilise bien l'IP (pas localhost)
  et que le PC est bien sur le hotspot du Pixel.
