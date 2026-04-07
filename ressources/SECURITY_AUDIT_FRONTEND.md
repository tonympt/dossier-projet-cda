# Audit de Sécurité Frontend - OWASP Top 10:2025

**Date:** 2026-01-29
**Version:** 1.1
**Auditeur:** Claude Code
**Scope:** Application React/TypeScript (app/frontend)
**Référentiel:** OWASP Top 10:2025

---

## Résumé Exécutif

L'application frontend présente une **base de sécurité solide** avec TypeScript strict et validation Zod, mais comporte plusieurs **vulnérabilités critiques** nécessitant une attention immédiate.

### Score par Catégorie OWASP

| Catégorie OWASP | Score | État |
|-----------------|-------|------|
| A01 - Broken Access Control | 6/10 | 🟡 |
| A02 - Security Misconfiguration | 5/10 | 🔴 |
| A03 - Software Supply Chain | 8/10 | 🟢 |
| A04 - Cryptographic Failures | 4/10 | 🔴 |
| A05 - Injection | 9/10 | 🟢 |
| A06 - Insecure Design | 7/10 | 🟡 |
| A07 - Authentication Failures | 5/10 | 🔴 |
| A08 - Software/Data Integrity | 7/10 | 🟡 |
| A09 - Logging & Alerting | 6/10 | 🟡 |
| A10 - Error Handling | 5/10 | 🔴 |

### Synthèse des Failles

| Criticité | Nombre |
|-----------|--------|
| 🔴 Critique | 5 |
| 🟡 Moyen | 6 |
| 🟢 Conforme | 7 |

---

## A01:2025 - Broken Access Control

> Failles permettant aux utilisateurs de contourner les restrictions d'accès et d'accéder à des ressources ou données non autorisées.

### ✅ Points Conformes

**Protection des Routes Authentifiées**
- Fichier: `app/frontend/src/routes/_authenticated.tsx`
```typescript
beforeLoad: ({ location }) => {
  const isAuthenticated = selectIsAuthenticated(useAuthStore.getState());
  if (!isAuthenticated) {
    throw redirect({
      to: "/login",
      search: { redirect: location.href },
    });
  }
},
```

**Protection des Routes Admin**
- Fichier: `app/frontend/src/routes/admin/admin.tsx`
- Vérification du rôle ADMIN/SUPERADMIN avant accès

---

### 🟡 A01-01 - Open Redirect Potentiel

**Fichier:** `app/frontend/src/routes/_authenticated.tsx`

**Problème:** `location.href` peut contenir une URL malveillante externe.

```typescript
search: { redirect: location.href } // ❌ Pas de validation
```

**Impact:** Un attaquant peut rediriger l'utilisateur vers un site de phishing après authentification.

**Recommandation:**
```typescript
function isValidRedirectPath(path: string): boolean {
  try {
    const url = new URL(path, window.location.origin);
    return url.origin === window.location.origin;
  } catch {
    return path.startsWith('/');
  }
}

search: {
  redirect: isValidRedirectPath(location.href) ? location.href : "/"
}
```

---

### 🟡 A01-02 - Vérification des Rôles Côté Client

**Fichier:** `app/frontend/src/routes/admin/admin.tsx`

**Problème:** Vérification côté client uniquement. Le rôle peut être modifié via DevTools.

**Impact:** Contournement de l'interface admin (mais pas des APIs si backend sécurisé).

**Note:** La vérification backend est essentielle et doit être documentée/vérifiée.

---

## A02:2025 - Security Misconfiguration

> Configuration inadéquate de systèmes, serveurs ou applications laissant des vulnérabilités exploitables.

### 🔴 A02-01 - Content-Security-Policy Manquante

**Problème:** Aucune CSP configurée pour le frontend.

**Impact:**
- Vulnérabilité aux attaques XSS
- Injection de scripts externes possibles
- Clickjacking non protégé

**Recommandation (à configurer côté serveur):**
```
Content-Security-Policy:
  default-src 'self';
  script-src 'self';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https://res.cloudinary.com;
  connect-src 'self' https://api.stripe.com https://api.cloudinary.com;
  frame-ancestors 'none';
```

---

### 🔴 A02-02 - Devtools Exposés en Production

**Fichiers:**
- `app/frontend/src/main.tsx`
- `app/frontend/src/components/layouts/RootLayout.tsx`

```typescript
<ReactQueryDevtools initialIsOpen={false} />
<TanStackRouterDevtools />
```

**Impact:**
- Exposition du cache des requêtes API
- Historique de navigation visible
- Paramètres sensibles exposés
- Structure de l'application révélée

**Recommandation:**
```typescript
// main.tsx
{import.meta.env.DEV && <ReactQueryDevtools initialIsOpen={false} />}

// RootLayout.tsx
{import.meta.env.DEV && <TanStackRouterDevtools />}
```

---

### 🟡 A02-03 - Configuration Vite Dev Exposée

**Fichier:** `app/frontend/vite.config.ts`

```typescript
server: {
  host: "0.0.0.0",  // ⚠️ Accessible de partout sur le réseau
}
```

**Impact:** En développement, le serveur est accessible depuis n'importe quelle IP du réseau local.

**Recommandation:** Utiliser `localhost` en développement.

---

## A03:2025 - Software Supply Chain Failures

> Défaillances dans les dépendances logicielles, bibliothèques tierces ou processus de mise à jour.

### ✅ Points Conformes

- `package-lock.json` présent
- Dépendances modernes et maintenues (@tanstack, zod, axios)
- Pas de scripts npm suspects
- Préfixe `VITE_` utilisé correctement pour les variables d'environnement

---

### 🟡 A03-01 - Audit de Dépendances Non Automatisé

**Problème:** Pas de `npm audit` configuré dans le pipeline CI/CD.

**Recommandation:**
```json
"scripts": {
  "audit:security": "npm audit --audit-level=moderate"
}
```

---

## A04:2025 - Cryptographic Failures

> Défauts de mise en œuvre ou absence de chiffrement, exposant les données sensibles.

### 🔴 A04-01 - Token JWT en localStorage (Texte Clair)

**Fichier:** `app/frontend/src/features/auth/store/authStore.ts`

**Problème:** Le token d'accès est stocké en **localStorage** via Zustand persist, en texte clair.

```typescript
export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({...}),
    {
      name: "greenroots-auth",
      partialize: (state) => ({
        accessToken: state.accessToken,  // ⚠️ Stocké en localStorage
        user: state.user,
      }),
    }
  )
);
```

**Impact:**
- Accès direct aux tokens en cas d'attaque XSS
- Pas de protection HttpOnly
- Pas de chiffrement
- Token accessible via `localStorage.getItem('greenroots-auth')`

**Recommandation:**
```typescript
// Utiliser des cookies HttpOnly côté backend
// Stocker uniquement en mémoire côté frontend
export const useAuthStore = create<AuthState>()((set) => ({
  accessToken: null, // En mémoire uniquement, rechargé via refresh token
  // ...
}));
```

---

### 🟡 A04-02 - Déconnexion et Nettoyage Incomplet

**Fichier:** `app/frontend/src/features/auth/components/LogoutComponent.tsx`

**Problème:** Nettoyage fragmenté entre localStorage et Zustand store.

**Impact:** Tokens résiduels potentiellement accessibles après déconnexion.

---

## A05:2025 - Injection

> Injection de code malveillant exploitant des entrées utilisateur non validées.

### ✅ Points Conformes

**Validation Côté Client avec Zod**
- Utilisation cohérente de **Zod** pour la validation
- Contraintes de longueur définies (max 64, 255 caractères)
- Validation email stricte
- Regex de complexité pour les mots de passe

```typescript
// Register Schema - Bien sécurisé
export const registerSchema = z.object({
  firstName: z.string().min(1).max(64),
  lastName: z.string().min(1).max(64),
  email: z.string().min(1).email().max(255),
  password: z.string()
    .min(8)
    .regex(passwordRegex, "Au moins 1 majuscule, minuscule, chiffre et caractère spécial"),
});
```

**Protection XSS Native React**
- **Aucun `dangerouslySetInnerHTML`** détecté
- **Aucun `innerHTML`** détecté
- **Aucun `eval()`** détecté
- Toutes les données rendues via JSX (échappement automatique)

---

### 🟡 A05-01 - Construction d'URLs Sans Sanitization

**Fichier:** `app/frontend/src/routes/auth/verify-email.tsx`

```typescript
const response = await fetch(
  `${import.meta.env.VITE_API_URL}/auth/verify-email?token=${token}`,
);
```

**Impact:** Injection potentielle dans les paramètres d'URL si token non validé.

**Recommandation:**
```typescript
const url = new URL(`${import.meta.env.VITE_API_URL}/auth/verify-email`);
url.searchParams.set('token', token);
const response = await fetch(url.toString());
```

---

## A06:2025 - Insecure Design

> Failles dues à des choix de conception défaillants ou absence de contrôles de sécurité dès la conception.

### ✅ Points Conformes

- Architecture React moderne avec séparation des concerns
- Utilisation de React Query pour la gestion des données
- React Hook Form pour les formulaires
- TypeScript strict activé

---

### 🟡 A06-01 - Pas de Rate Limiting Côté Client

**Problème:** Les formulaires peuvent être soumis rapidement sans throttle/debounce visible.

**Impact:** Facilite les attaques de brute force (si backend non protégé).

**Recommandation:** Ajouter un debounce sur les soumissions de formulaires critiques.

---

## A07:2025 - Authentication Failures

> Défaillances des mécanismes d'authentification permettant l'accès non autorisé.

### ✅ Points Conformes

- Validation des credentials avec Zod
- Redirection vers login si non authentifié
- Gestion de l'état d'authentification centralisée

---

### 🔴 A07-01 - Pas de Refresh Token Visible

**Problème:** Aucune implémentation de refresh token côté frontend.

**Impact:**
- Token JWT expire → utilisateur déconnecté brutalement
- Pas de rotation de tokens
- Session longue = token longtemps valide si compromis

**Recommandation:** Implémenter refresh token avec rotation automatique.

---

### 🟡 A07-02 - Pas de Protection Contre Session Fixation

**Problème:** Pas de régénération de session après authentification réussie.

**Impact:** Vulnérabilité théorique si attaquant fixe un token avant connexion.

---

## A08:2025 - Software or Data Integrity Failures

> Absence de vérification de l'intégrité du logiciel ou des données.

### ✅ Points Conformes

- Pas de désérialisation dangereuse (JSON.parse sur données non fiables)
- Pas d'évaluation de code dynamique (eval)
- Validation des données entrantes avec Zod

---

### 🟡 A08-01 - Pas de Vérification d'Intégrité des Assets

**Problème:** Pas de Subresource Integrity (SRI) sur les scripts externes.

**Recommandation:** Ajouter SRI pour les CDN externes si utilisés.

---

## A09:2025 - Security Logging and Alerting Failures

> Insuffisance de journalisation et de monitoring pour détecter et réagir aux incidents.

### 🔴 A09-01 - Console.error Exposant Variables d'Environnement

**Fichier:** `app/frontend/src/lib/cloudinary.ts`

```typescript
console.error("Available env vars:", import.meta.env);
```

**Impact:** Exposition de toutes les variables d'environnement VITE_ dans la console navigateur.

**Recommandation:** Supprimer cette ligne immédiatement.

---

### 🟡 A09-02 - Pas de Logging des Erreurs de Sécurité

**Problème:** Aucun service de logging/monitoring (Sentry, Datadog) configuré.

**Impact:** Impossible de détecter les tentatives d'attaque ou erreurs de sécurité.

**Recommandation:** Intégrer Sentry ou équivalent pour le monitoring des erreurs.

---

## A10:2025 - Mishandling of Exceptional Conditions

> Gestion inadéquate des erreurs et conditions exceptionnelles révélant des informations sensibles.

### 🔴 A10-01 - Exposition des Messages d'Erreur Backend

**Fichier:** `app/frontend/src/lib/errors.ts`

```typescript
export function getErrorMessage(error: unknown): string {
  const axiosError = error as AxiosError<{ message?: string | string[] }>;
  const message = axiosError.response?.data?.message;

  if (typeof message === "string") return message;
  // Messages d'erreur backend remontés directement à l'utilisateur
}
```

**Impact:**
- Peut exposer des détails techniques (structure BDD, chemins serveur)
- Information Disclosure
- Aide les attaquants à comprendre l'architecture

**Recommandation:**
```typescript
export function getErrorMessage(error: unknown): string {
  const axiosError = error as AxiosError<{ message?: string | string[] }>;
  const status = axiosError.response?.status;

  // Mapper vers des messages génériques
  const errorMessages: Record<number, string> = {
    400: "Données invalides",
    401: "Identifiants incorrects",
    403: "Accès refusé",
    404: "Ressource introuvable",
    500: "Erreur serveur. Veuillez réessayer.",
  };

  return errorMessages[status ?? 0] || "Une erreur est survenue";
}
```

---

## Points Positifs Globaux

| Aspect | Statut | Catégorie OWASP |
|--------|--------|-----------------|
| TypeScript strict | ✅ | A05, A06 |
| Validation Zod | ✅ | A05 |
| Protection XSS (pas de dangerouslySetInnerHTML) | ✅ | A05 |
| Routes protégées | ✅ | A01 |
| Clés publiques uniquement exposées | ✅ | A02 |
| Dépendances modernes et maintenues | ✅ | A03 |
| React Hook Form pour les formulaires | ✅ | A05, A06 |

---

## Plan de Remédiation par Priorité OWASP

### P0 - Critique (< 1 semaine)

| Faille | Catégorie | Action |
|--------|-----------|--------|
| A04-01 | Cryptographic | Migrer tokens vers HttpOnly Cookies |
| A02-02 | Misconfiguration | Conditionner devtools à `import.meta.env.DEV` |
| A10-01 | Error Handling | Filtrer les messages d'erreur backend |
| A09-01 | Logging | Supprimer `console.error(import.meta.env)` |
| A02-01 | Misconfiguration | Configurer CSP au niveau serveur |

### P1 - Important (< 1 mois)

| Faille | Catégorie | Action |
|--------|-----------|--------|
| A01-01 | Access Control | Implémenter protection open-redirect |
| A07-01 | Authentication | Implémenter refresh token avec rotation |
| A05-01 | Injection | Utiliser URLSearchParams pour construction d'URLs |
| A02-03 | Misconfiguration | Restreindre host Vite à localhost |

### P2 - Amélioration (< 3 mois)

| Faille | Catégorie | Action |
|--------|-----------|--------|
| A03-01 | Supply Chain | Audit de dépendances automatisé (npm audit ci) |
| A09-02 | Logging | Intégrer Sentry ou équivalent |
| A06-01 | Design | Ajouter rate limiting côté client |

---

## Checklist de Vérification

```
A01 - Broken Access Control:
[ ] Open-redirect protection implémentée
[ ] Vérification backend des rôles documentée

A02 - Security Misconfiguration:
[ ] CSP configurée
[ ] Devtools conditionnés à DEV
[ ] Host Vite restreint à localhost

A03 - Software Supply Chain:
[ ] npm audit clean
[ ] Pipeline CI avec audit automatique

A04 - Cryptographic Failures:
[ ] Token stocké en HttpOnly Cookie
[ ] Refresh token implémenté

A05 - Injection:
[ ] URLSearchParams utilisé partout

A07 - Authentication Failures:
[ ] Refresh token avec rotation
[ ] Session regeneration après login

A09 - Logging & Alerting:
[ ] console.error supprimé
[ ] Sentry/monitoring intégré

A10 - Error Handling:
[ ] Messages d'erreur filtrés
[ ] Pas d'exposition de stack traces
```

---

## Ressources

- [OWASP Top 10:2025](https://owasp.org/Top10/2025/)
- [OWASP Cheat Sheet - XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP Cheat Sheet - Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [React Security Best Practices](https://snyk.io/blog/10-react-security-best-practices/)

---

*Rapport généré automatiquement par Claude Code - Référentiel OWASP Top 10:2025*
