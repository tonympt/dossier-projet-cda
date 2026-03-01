# Feature 05 — Migration JWT : localStorage vers HttpOnly Cookies

## Priorité : P1 | Estimation : ~4h

## Contexte — Pourquoi cette migration ?

### Le problème (localStorage)
Le JWT était stocké dans **localStorage** et injecté dans chaque requête via un interceptor Axios (`Authorization: Bearer ...`). Le problème : `localStorage` est accessible par **n'importe quel JavaScript** exécuté sur la page. Si une faille XSS existe (injection de script malveillant), le token peut être volé et utilisé depuis n'importe quelle machine.

### La solution (HttpOnly cookies)
Le JWT est maintenant posé dans un **cookie HttpOnly** par le backend via le header `Set-Cookie`. Le navigateur le stocke automatiquement et l'envoie automatiquement à chaque requête. Le JavaScript n'y a **jamais accès** (flag `HttpOnly`), donc même en cas de faille XSS, le token reste invisible.

### Comparaison

| Aspect | Avant (localStorage) | Après (HttpOnly cookie) |
|---|---|---|
| **Stockage** | `localStorage["greenroots-auth"]` contient le JWT | Cookie HttpOnly dans le navigateur |
| **Visible par JS ?** | Oui (`localStorage.getItem(...)`) | Non (flag `HttpOnly`) |
| **Envoi au serveur** | Manuel (interceptor Axios ajoutait `Authorization: Bearer`) | Automatique (le navigateur envoie le cookie) |
| **Risque XSS** | Token volable | Token invisible |
| **CSRF** | Pas de risque | Protégé par `SameSite=Lax` |
| **Logout** | Frontend supprimait le token du store | Backend supprime le cookie + blacklist Redis |

### Les 3 flags de sécurité du cookie

| Flag | Valeur | Rôle |
|---|---|---|
| `HttpOnly` | `true` | Interdit l'accès JavaScript au cookie |
| `Secure` | `true` en prod, `false` en dev | Cookie envoyé uniquement sur HTTPS |
| `SameSite` | `Lax` | Cookie non envoyé sur les requêtes cross-site POST/PUT/DELETE (protection CSRF) |

## Modifications techniques

### Backend (6 fichiers modifiés, 1 nouveau)

#### `app/backend/src/auth/utils/cookie.util.ts` (nouveau)
- Constante `COOKIE_NAME = 'access_token'` — nom unique du cookie
- Fonction `getCookieOptions(isProduction: boolean)` qui retourne les flags du cookie :
  - `httpOnly: true` — invisible au JavaScript
  - `secure: isProduction` — HTTPS uniquement en prod (en dev, localhost est exempté)
  - `sameSite: 'lax'` — protection CSRF
  - `path: '/'` — cookie disponible sur toutes les routes
  - `maxAge: 24 * 60 * 60 * 1000` — 1 jour, aligné avec `JWT_EXPIRATION`
- Helper centralisé pour garantir la cohérence des options entre login/logout/delete

#### `app/backend/src/main.ts`
- **Import** de `cookie-parser` (middleware Express pour parser les cookies entrants)
- **Ajout** de `app.use(cookieParser())` avant Helmet et CORS
- **CORS** : ajout de `credentials: true` — **obligatoire** pour que le navigateur envoie le cookie cross-origin
- **CORS** : retrait de `'Authorization'` des `allowedHeaders` (plus nécessaire, le token circule dans le cookie)

#### `app/backend/src/auth/auth.service.ts`
- `login()` retourne maintenant `{ token: string, message: string }` au lieu de `{ access_token: string }`
- Le `token` est retourné au controller pour qu'il le place dans le cookie
- Le `message` ("Connexion réussie.") est retourné au frontend dans le body de la réponse

#### `app/backend/src/auth/dtos/login-response.dto.ts`
- Remplacé `access_token: string` par `message: string`
- Le token ne circule plus jamais dans le body de la réponse — il est uniquement dans le `Set-Cookie`

#### `app/backend/src/auth/auth.controller.ts`
- **Injection** de `ConfigService` dans le constructeur pour déterminer si on est en production
- **`POST /auth/login`** :
  - Ajout de `@Res({ passthrough: true })` pour accéder à l'objet `Response` Express tout en gardant le serializer
  - Appel `res.cookie(COOKIE_NAME, token, getCookieOptions(...))` pour poser le cookie
  - Retourne `{ message }` (sans le token)
- **`POST /auth/logout`** :
  - Lecture du token depuis `req.cookies[COOKIE_NAME]` au lieu de `req.headers.authorization`
  - Appel `res.clearCookie(COOKIE_NAME, ...)` pour supprimer le cookie du navigateur
  - Blacklist Redis du token (comme avant)
- **`DELETE /auth/me`** (suppression compte) :
  - Ajout de `res.clearCookie()` après la suppression pour nettoyer le cookie

#### `app/backend/src/auth/strategies/jwt.strategy.ts`
- **Extracteur custom** `extractJwtFromCookie(req)` remplace `ExtractJwt.fromAuthHeaderAsBearerToken()`
- Lit le JWT depuis `req.cookies[COOKIE_NAME]` au lieu du header `Authorization`
- `passReqToCallback: true` pour accéder à la requête dans `validate()`
- Dans `validate()` : vérifie la blacklist avec le token lu depuis le cookie

#### `package.json`
- Ajout de `cookie-parser` et `@types/cookie-parser` dans les dépendances

### Frontend (5 fichiers modifiés)

#### `app/frontend/src/lib/api/client.ts`
- **Ajout** de `withCredentials: true` dans la config Axios — le navigateur envoie automatiquement le cookie
- **Suppression** de l'interceptor `request` qui ajoutait `Authorization: Bearer ...`
- L'interceptor `response` (401 → logout + redirect) est conservé tel quel

#### `app/frontend/src/features/auth/store/authStore.ts`
- **Suppression** de `accessToken` du state Zustand
- **Renommage** de `setAuth({ accessToken, user })` en `setUser(user)` — ne prend plus que le `user`
- `logout()` : met juste `user: null`
- `selectIsAuthenticated` : retourne `Boolean(state.user)` au lieu de `Boolean(state.accessToken)`
- Le `user` reste dans le `partialize` (persisté en localStorage via Zustand) — seul le **token** disparaît du localStorage

#### `app/frontend/src/features/auth/hooks/useLoginMutation.tsx`
- Plus besoin d'extraire `data.access_token` de la réponse (le token est dans le cookie, pas dans le body)
- Après le login : appelle `GET /auth/me` pour récupérer le profil complet (le cookie est envoyé automatiquement)
- Appelle `setUser(hydratedUser)` au lieu de `setAuth({ accessToken, user })`

#### `app/frontend/src/components/layouts/AuthenticatedLayout.tsx`
- Query key simplifiée : `["authMe"]` au lieu de `["authMe", accessToken]`
- `enabled: Boolean(user)` au lieu de `Boolean(accessToken)`
- `setUser(hydratedUser)` au lieu de `setAuth({ accessToken, user })`
- **Fix boucle infinie** : le useEffect comparait l'objet `user` en dépendance, mais `setUser` créait un nouvel objet à chaque appel → re-render infini. Corrigé en lisant le state courant via `useAuthStore.getState().user` et en comparant avec `JSON.stringify` avant de mettre à jour.

#### `app/frontend/src/routes/admin/admin.tsx`
- Remplacé `if (state.accessToken)` par `if (state.user)`
- `setUser(hydratedUser)` au lieu de `setAuth({ accessToken, user })`

### Fichiers qui n'ont PAS changé
- `_authenticated.tsx` / `_auth-guard.tsx` — utilisent `selectIsAuthenticated` qui a été mis à jour à la source
- `Header.tsx` — utilise `selectIsAuthenticated` et `user`, les deux fonctionnent toujours
- `DeleteAccountSection.tsx` — appelle `logout()` qui a été mis à jour
- `useLogoutMutation.ts` — appelle `POST /auth/logout` (fonctionne avec les cookies) puis `logout()`
- `token-blacklist.service.ts` — reçoit le token brut, ne se soucie pas de sa provenance

## Scénario refresh page

Avec localStorage : Zustand rechargeait le token depuis localStorage → l'app savait qu'on était connecté.

Avec HttpOnly cookies : Zustand recharge le `user` depuis localStorage (persist middleware) → `selectIsAuthenticated` retourne `true` → `AuthenticatedLayout` appelle `GET /auth/me` (cookie envoyé automatiquement) → si le cookie est valide, le profil revient et l'app fonctionne. Si le cookie est expiré/absent, le 401 déclenche le logout.

Le user data reste dans le localStorage Zustand (pour le nom, avatar, rôle dans le header). Seul le **token** disparaît du localStorage — c'est exactement le gain de sécurité recherché.

## Bug rencontré et résolu

### Boucle infinie React (`Maximum update depth exceeded`)
- **Cause** : dans `AuthenticatedLayout`, le `useEffect` avait `user` dans ses dépendances. `setUser()` créait un nouvel objet → le state changeait → le composant re-rendait → le useEffect se relançait → `setUser()` encore → boucle infinie.
- **Symptôme** : le navigateur crashait avec `Maximum update depth exceeded`, même si les requêtes réseau fonctionnaient correctement (cookie envoyé, /auth/me répondait 200).
- **Fix** : retrait de `user` des deps du useEffect. Lecture du state courant via `useAuthStore.getState().user` (accès direct, pas un hook). Comparaison `JSON.stringify(current) !== JSON.stringify(hydratedUser)` avant d'appeler `setUser`. Le useEffect ne se relance que quand `meQuery.data` change.

## Vérification

1. **Login** : DevTools > Application > Cookies → cookie `access_token` existe avec flags `HttpOnly`, `SameSite=Lax`
2. **Requêtes authentifiées** : DevTools > Network → le cookie est envoyé automatiquement dans `Cookie: access_token=...`
3. **XSS impossible** : Console → `document.cookie` → le cookie `access_token` n'apparaît PAS
4. **Refresh page** : le user reste connecté
5. **Logout** : le cookie disparaît, redirection login
6. **Suppression compte** : le cookie disparaît aussi

## Branche
`feature/jwt-httponly-cookies`

## Fichiers modifiés
- `app/backend/src/auth/utils/cookie.util.ts` (nouveau)
- `app/backend/src/auth/auth.controller.ts`
- `app/backend/src/auth/auth.service.ts`
- `app/backend/src/auth/dtos/login-response.dto.ts`
- `app/backend/src/auth/strategies/jwt.strategy.ts`
- `app/backend/src/main.ts`
- `app/backend/package.json` + `package-lock.json`
- `app/frontend/src/lib/api/client.ts`
- `app/frontend/src/features/auth/store/authStore.ts`
- `app/frontend/src/features/auth/hooks/useLoginMutation.tsx`
- `app/frontend/src/components/layouts/AuthenticatedLayout.tsx`
- `app/frontend/src/routes/admin/admin.tsx`

## Sections du dossier CDA concernées
- §7.2 (composant métier — stratégie JWT custom, helper centralisé des options cookie)
- §8.1/§8.2 (sécurité — protection XSS par cookie HttpOnly, CSRF par SameSite, cookie Secure en prod)
- §8.5 (RGPD — le token n'est plus accessible côté client)
- §6.2 (choix techniques — cookie-parser, withCredentials, CORS credentials)
