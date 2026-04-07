# Performance — Code-splitting par route

## Problème identifié

Lighthouse sur le build production (`vite preview`) signalait deux points sur toutes les pages :

| Diagnostic | Impact |
|---|---|
| **Unused JavaScript** | 202–218 KiB gaspillés sur chaque page |
| **Render blocking** | 1,050 ms de savings estimés sur `/` |

### Cause racine

Le projet utilise **TanStack Router en mode code-based**. Chaque route est définie dans un fichier séparé, puis assemblée dans `main.tsx` via `.addChildren()`.

Le problème : **36 des 37 routes importaient leur composant statiquement** (`import { Home } from "..."` en haut du fichier). Puisque `main.tsx` importe tous les fichiers de route, la chaîne d'imports statiques entraîne **tous** les composants dans un seul chunk :

```
main.tsx
  → routes/index.tsx          → import { Home }
  → routes/about.tsx          → import { About }
  → routes/trees/trees.tsx    → import { TreeCatalog }
  → routes/admin/*.tsx        → import { AdminTreesPage, ... }
  → routes/profile/*.tsx      → import { MyProfilePage, ... }
  → ...
```

**Résultat :** un chunk unique `index.js` à 1,136 KB contenant l'intégralité de l'app. Sur `/`, seul `Home` + `CrossSellingCard` sont utilisés — le reste (admin, profil, auth, autres pages) est chargé mais jamais exécuté.

Le CSS Tailwind (81 KB) était aussi un seul fichier, chargé en bloquant le rendu avant que le premier pixel ne soit affiché.

---

## Solution : lazy loading sur toutes les routes

Chaque route avec un import de composant externe a été convertie pour utiliser l'option `lazy` de TanStack Router :

```typescript
// Avant
import { About } from '@/features/about/About'

export const aboutRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/about',
  component: About,
})

// Après
export const aboutRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/about',
  lazy: async () => {
    const { About } = await import('@/features/about/About')
    return { component: About }
  },
})
```

### Routes converties (25 total)

| Groupe | Routes |
|---|---|
| Public | `/`, `/about`, `/legal`, `/privacy`, `/terms` |
| Catalogue | `/trees`, `/tree/$id` |
| Auth | `/login`, `/register`, `/forgot-password`, `/reset-password` |
| Checkout | `/cart`, `/order-confirmation` |
| Profil | `/profile/me`, `/profile/me/orders`, `/profile/me/orders/$id` |
| Admin | `trees`, `trees/new`, `trees/edit/$id`, `locations`, `locations/new`, `locations/edit/$id`, `categories`, `orders`, `orders/$id` |

### Routes non touchées

| Route | Raison |
|---|---|
| `__root.tsx` | Layout racine — doit être statique |
| `_authenticated.tsx` | Layout auth avec guard — toujours nécessaire quand authentifié |
| `_auth-guard.tsx` | Utilise `<Outlet />` inline |
| `auth/logout.tsx` | Composant défini dans le même fichier |
| `auth/verify-email.tsx` | Composant défini dans le même fichier |
| Layout routes (`profile.tsx`, `profile.me.tsx`, `orders.tsx`) | Utilisent `<Outlet />` inline |
| Redirect routes (`profile.index.tsx`, `admin.index.tsx`) | `component: () => null` |
| `checkout/checkout.tsx` | Déjà lazy depuis une optimisation précédente |

### Pourquoi ça marche

Quand une route est lazy, Vite crée un **chunk séparé** pour elle à la compilation. Le chunk principal ne contient plus que le code partagé (React, React Query, TanStack Router, Header, Footer, etc.). Chaque page charge uniquement son propre chunk au moment de la navigation.

Le router a déjà `defaultPreload: "intent"` configuré : les chunks sont **préchargés au hover** sur les liens, donc pas de délai perçu lors de la navigation.

---

## Résultats

### Taille des chunks

| | Avant | Après |
|---|---|---|
| Chunk principal (`index.js`) | 1,136 KB (330 KB gzip) | 755 KB (220 KB gzip) |
| Chunks par route | — | 1–27 KB chacun |

### Scores Lighthouse (build production, `vite preview`)

| Page | Perf avant | Perf après | Δ | Access | Best | SEO |
|---|---|---|---|---|---|---|
| `/` | 77 | 95 | +18 | 100 | 96 | 100 |
| `/trees` | 74 | 93 | +19 | 100 | 96 | 100 |
| `/about` | 81 | 95 | +14 | 100 | 96 | 100 |
| `/terms` | 88 | 93 | +5 | 100 | 96 | 100 |
| `/legal` | 86 | 95 | +9 | 100 | 96 | 100 |
| `/privacy` | 85 | 92 | +7 | 100 | 96 | 100 |

### Unused JS

| Page | Avant | Après |
|---|---|---|
| `/` | 202 KiB | 126 KiB |
| `/trees` | 208 KiB | 126 KiB |
| `/about` | 218 KiB | 126 KiB |

Les 126 KiB restants sont dans le chunk partagé (dépendances tierces comme React, React Query, zustand) — ils ne peuvent pas être réduits davantage par du code-splitting.

### Render blocking

| | Avant | Après |
|---|---|---|
| Savings estimés (/) | 1,050 ms | 450 ms |

---

## Ce qui reste

| Point | Détail |
|---|---|
| **CSS (304 ms)** | Le fichier Tailwind est généré globalement — Vite ne peut pas le découper par route. Pour aller plus loin : **critical CSS inlining** avec un plugin comme `critters` (extraire le CSS above-the-fold, le mettre inline dans `<head>`, charger le reste en async). |
| **126 KiB unused JS** | Dans le chunk partagé. Réductible avec `manualChunks` dans la config Rollup pour isoler les grosses dépendances tierces, mais avec un retour sur investissement décroissant. |
