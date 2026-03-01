# Feature 07 — Tests unitaires de composants React

## Priorité : P2 | Estimation : ~1h

## Contexte
Le frontend utilisait Vitest mais n'avait pas de tests unitaires sur les composants UI React. Les tests existants portaient sur des utilitaires (`useMediaQuery`, `currency formatter`). L'objectif est de démontrer la capacité à tester des composants React avec **Vitest + React Testing Library**, en couvrant deux composants de complexité différente.

## Stack de test

| Outil | Rôle |
|---|---|
| **Vitest** | Test runner (compatible Vite, plus rapide que Jest pour un projet Vite) |
| **jsdom** | Environnement DOM simulé dans Node.js (simule un navigateur) |
| **@testing-library/react** | Utilitaires de rendu et requêtes DOM (render, screen, queries) |
| **@testing-library/jest-dom** | Matchers custom (`toBeDisabled()`, `toHaveValue()`, `toHaveAttribute()`) |
| **@testing-library/user-event** | Simulation d'interactions utilisateur (click, type) de manière réaliste |

## Configuration préalable

### `app/frontend/vite.config.ts`
Ajout de la section `test` dans la config Vite :
```typescript
test: {
  environment: "jsdom",  // Simule un navigateur dans Node.js
  setupFiles: ["./src/test/setup.ts"],  // Charge les matchers jest-dom
},
```

### `app/frontend/src/test/setup.ts`
Fichier existant qui charge `@testing-library/jest-dom` pour les matchers custom comme `toBeDisabled()`.

## Composant 1 — LoadingSkeleton (composant simple, sans interaction)

### Le composant (`components/molecules/LoadingSkeleton.tsx`)
Un composant skeleton de chargement qui affiche N rangées de placeholder animées. Utilisé dans les listes et tableaux pendant le chargement des données.

```tsx
export function LoadingSkeleton({ rows = 6 }: { rows?: number }) {
  return (
    <div className="space-y-3">
      {Array.from({ length: rows }).map((_, index) => (
        <Skeleton key={index} className="h-10 w-full" />
      ))}
    </div>
  );
}
```

### Tests (`components/molecules/__tests__/LoadingSkeleton.test.tsx`)

| Test | Ce qu'il vérifie |
|---|---|
| `should render 6 skeleton rows by default` | La prop `rows` a une valeur par défaut de 6 |
| `should render the specified number of rows` | Passer `rows={3}` affiche bien 3 skeletons |
| `should render 0 rows when rows=0` | Le cas limite `rows={0}` n'affiche rien |
| `should render a wrapper div with spacing` | Le wrapper a la classe CSS `space-y-3` pour l'espacement |

**Technique** : `container.querySelectorAll("[data-slot='skeleton']")` — query par attribut `data-slot` car le composant shadcn/ui Skeleton utilise cet attribut.

## Composant 2 — QuantityInput (composant interactif avec état)

### Le composant (`components/molecules/QuantityInput.tsx`)
Un sélecteur de quantité avec boutons +/- et champ de saisie numérique. Utilisé dans le panier pour modifier la quantité d'articles. Gère les bornes min/max, l'état désactivé, et expose des attributs ARIA pour l'accessibilité.

```
 [ - ]  5  [ + ]
```

### Props
| Prop | Type | Description |
|---|---|---|
| `value` | `number` | Valeur actuelle |
| `onChange` | `(value: number) => void` | Callback quand la valeur change |
| `min` | `number` (défaut: 1) | Valeur minimale |
| `max` | `number` (optionnel) | Valeur maximale |
| `disabled` | `boolean` (défaut: false) | Désactive tous les contrôles |

### Tests (`components/molecules/__tests__/QuantityInput.test.tsx`)

| Test | Ce qu'il vérifie |
|---|---|
| `should render with the initial value` | Le champ affiche la valeur passée en prop |
| `should call onChange when increment button is clicked` | Clic sur "+" → `onChange(value + 1)` |
| `should call onChange when decrement button is clicked` | Clic sur "-" → `onChange(value - 1)` |
| `should disable decrement at min value` | Bouton "-" désactivé quand `value === min` |
| `should disable increment at max value` | Bouton "+" désactivé quand `value === max` |
| `should disable all controls when disabled prop is true` | Tous les contrôles (input + boutons) sont désactivés |
| `should have correct aria attributes` | L'input a `aria-valuemin`, `aria-valuemax`, `aria-valuenow` |

### Techniques utilisées
- **`userEvent.setup()`** : simulation réaliste des clics utilisateur (contrairement à `fireEvent`, `userEvent` simule le comportement navigateur complet : focus, mousedown, mouseup, click)
- **`vi.fn()`** : mock du callback `onChange` pour vérifier qu'il est appelé avec la bonne valeur
- **`screen.getByRole("spinbutton")`** : query par rôle ARIA — le champ `input[type="number"]` a le rôle implicite `spinbutton`
- **`screen.getByLabelText("Augmenter la quantité")`** : query par `aria-label` — vérifie que l'accessibilité est correcte ET sélectionne le bon bouton
- **`toBeDisabled()`** : matcher jest-dom pour vérifier l'état disabled du bouton
- **`toHaveAttribute("aria-valuemin", "1")`** : vérifie les attributs ARIA d'accessibilité

## Résultat CI
```
 ✓ src/components/molecules/__tests__/LoadingSkeleton.test.tsx (4 tests)
 ✓ src/components/molecules/__tests__/QuantityInput.test.tsx (7 tests)

Test Files  8 passed
Tests       60 passed
```

## Branche
`feature/react-component-tests`

## Fichiers modifiés
- `app/frontend/vite.config.ts` (ajout config test)
- `app/frontend/src/components/molecules/__tests__/LoadingSkeleton.test.tsx` (nouveau)
- `app/frontend/src/components/molecules/__tests__/QuantityInput.test.tsx` (nouveau)

## Sections du dossier CDA concernées
- §7.1 (composants UI — tests unitaires React, Vitest + Testing Library)
- §9 (plan de tests — tests composants, jeu d'essai, résultats CI)
- CP2 (interfaces utilisateur — tests unitaires réalisés, accessibilité ARIA vérifiée)
