# 🌳 Documentation d'Architecture - GreenRoots Frontend

## 📋 Table des matières

1. [Vue d'ensemble](#vue-densemble)
2. [Stack technologique](#stack-technologique)
3. [Structure du projet](#structure-du-projet)
4. [Architecture détaillée](#architecture-détaillée)
5. [Conventions de nommage](#conventions-de-nommage)
6. [Guides d'utilisation](#guides-dutilisation)
7. [Bonnes pratiques](#bonnes-pratiques)
8. [Configuration](#configuration)

---

## Vue d'ensemble

GreenRoots est une plateforme e-commerce éco-responsable pour la vente d'arbres. Le frontend est construit avec React, Vite et suit une architecture hybride combinant **Atomic Design** pour les composants UI et une **architecture feature-based** pour la logique métier.

### Principes directeurs

- **Modularité** : Chaque fonctionnalité est isolée et autonome
- **Réutilisabilité** : Les composants UI sont conçus pour être réutilisés
- **Scalabilité** : L'architecture permet de grandir facilement
- **Type-safety** : TypeScript partout pour la sécurité du code
- **Performance** : Optimisations avec Tanstack Query et lazy loading

---

## Stack technologique

### Core
- **React 18** : Bibliothèque UI
- **TypeScript** : Typage statique
- **Vite** : Build tool moderne et rapide

### Routing & Data Fetching
- **Tanstack Router** : Routing file-based avec type-safety
- **Tanstack Query** : Gestion du state serveur (cache, mutations, etc.)

### UI & Styling
- **Shadcn/ui** : Composants UI accessibles et customisables
- **Tailwind CSS** : Framework CSS utility-first
- **Radix UI** : Primitives accessibles (utilisé par Shadcn)

### State Management
- **Zustand** : Store global léger (pour état UI et données persistantes)

### Validation & Forms
- **Zod** : Schémas de validation TypeScript-first
- **React Hook Form** : Gestion des formulaires avec validation

### Font
- **Font unique** : Une seule typographie pour toute l'application

---

## Structure du projet

```
greenroots-frontend/
├── public/
│   └── fonts/                    # Font unique de l'application
│       └── ta-font.woff2
│
├── src/
│   ├── app/                      # Configuration principale de l'application
│   │   ├── App.tsx              # Composant racine
│   │   ├── router.tsx           # Configuration Tanstack Router
│   │   └── providers.tsx        # Providers (QueryClient, Theme, etc.)
│   │
│   ├── components/              # Composants UI partagés (Atomic Design)
│   │   ├── ui/                  # Composants Shadcn UI (auto-générés)
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── input.tsx
│   │   │   ├── dialog.tsx
│   │   │   └── ...
│   │   │
│   │   ├── atoms/               # Composants atomiques personnalisés
│   │   │   ├── Logo/
│   │   │   │   ├── Logo.tsx
│   │   │   │   └── index.ts
│   │   │   ├── Badge/
│   │   │   ├── Icon/
│   │   │   └── Spinner/
│   │   │
│   │   ├── molecules/           # Composants moléculaires
│   │   │   ├── SearchBar/
│   │   │   │   ├── SearchBar.tsx
│   │   │   │   └── index.ts
│   │   │   ├── TreeCard/
│   │   │   ├── UserAvatar/
│   │   │   └── PriceTag/
│   │   │
│   │   ├── organisms/           # Composants complexes
│   │   │   ├── Header/
│   │   │   │   ├── Header.tsx
│   │   │   │   └── index.ts
│   │   │   ├── TreeGrid/
│   │   │   ├── Footer/
│   │   │   ├── NavigationMenu/
│   │   │   └── CartSummary/
│   │   │
│   │   └── templates/           # Templates de layout
│   │       ├── MainLayout/
│   │       │   ├── MainLayout.tsx
│   │       │   └── index.ts
│   │       ├── DashboardLayout/
│   │       └── AuthLayout/
│   │
│   ├── features/                # Modules par fonctionnalité
│   │   │
│   │   ├── trees/               # Feature: Gestion des arbres
│   │   │   ├── components/      # Composants spécifiques aux arbres
│   │   │   │   ├── TreeFilters.tsx
│   │   │   │   ├── TreeDetails.tsx
│   │   │   │   └── TreeComparison.tsx
│   │   │   ├── hooks/           # Hooks personnalisés
│   │   │   │   ├── useTreeData.ts
│   │   │   │   ├── useTreeMutation.ts
│   │   │   │   └── useTreeFilters.ts
│   │   │   ├── api/             # Appels API
│   │   │   │   └── treesApi.ts
│   │   │   ├── types/           # Types TypeScript
│   │   │   │   └── tree.types.ts
│   │   │   └── index.ts         # API publique de la feature
│   │   │
│   │   ├── auth/                # Feature: Authentification
│   │   │   ├── components/
│   │   │   │   ├── LoginForm.tsx
│   │   │   │   └── RegisterForm.tsx
│   │   │   ├── hooks/
│   │   │   │   ├── useAuth.ts
│   │   │   │   └── useLogin.ts
│   │   │   ├── api/
│   │   │   │   └── authApi.ts
│   │   │   ├── store/           # Store Zustand spécifique
│   │   │   │   └── authStore.ts
│   │   │   ├── types/
│   │   │   │   └── auth.types.ts
│   │   │   └── index.ts
│   │   │
│   │   ├── cart/                # Feature: Panier
│   │   │   ├── components/
│   │   │   │   ├── CartItem.tsx
│   │   │   │   └── CartTotal.tsx
│   │   │   ├── hooks/
│   │   │   │   └── useCart.ts
│   │   │   ├── store/
│   │   │   │   └── cartStore.ts
│   │   │   ├── types/
│   │   │   │   └── cart.types.ts
│   │   │   └── index.ts
│   │   │
│   │   └── orders/              # Feature: Commandes
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── api/
│   │       ├── types/
│   │       └── index.ts
│   │
│   ├── routes/                  # Routes Tanstack Router (file-based)
│   │   ├── __root.tsx           # Route racine avec layout
│   │   ├── index.tsx            # Page d'accueil /
│   │   │
│   │   ├── trees/               # Routes /trees/*
│   │   │   ├── index.tsx        # Liste des arbres /trees
│   │   │   └── $treeId.tsx      # Détail d'un arbre /trees/:treeId
│   │   │
│   │   ├── cart.tsx             # Panier /cart
│   │   ├── checkout.tsx         # Paiement /checkout
│   │   │
│   │   ├── auth/                # Routes d'authentification
│   │   │   ├── login.tsx        # /auth/login
│   │   │   └── register.tsx     # /auth/register
│   │   │
│   │   └── account/             # Routes du compte utilisateur
│   │       ├── index.tsx        # /account
│   │       └── orders.tsx       # /account/orders
│   │
│   ├── hooks/                   # Hooks globaux réutilisables
│   │   ├── useDebounce.ts
│   │   ├── useLocalStorage.ts
│   │   ├── useMediaQuery.ts
│   │   ├── useOnClickOutside.ts
│   │   └── useIntersectionObserver.ts
│   │
│   ├── lib/                     # Bibliothèques et utilitaires
│   │   ├── utils.ts             # Fonctions utilitaires (cn, formatters, etc.)
│   │   ├── api/                 # Configuration API globale
│   │   │   ├── client.ts        # Instance axios/fetch configurée
│   │   │   └── queryClient.ts   # Configuration Tanstack Query
│   │   ├── validations/         # Schémas de validation (Zod)
│   │   │   ├── tree.schema.ts
│   │   │   └── auth.schema.ts
│   │   └── constants.ts         # Constantes globales
│   │
│   ├── store/                   # Store global Zustand
│   │   ├── index.ts             # Composition et export du store
│   │   └── slices/              # Slices de state par domaine
│   │       ├── uiSlice.ts       # État UI global (theme, sidebar, etc.)
│   │       └── cartSlice.ts     # État panier (si besoin de persist)
│   │
│   ├── styles/                  # Styles globaux
│   │   ├── globals.css          # Styles globaux + @tailwind directives
│   │   └── themes/              # Variables de thème
│   │       └── colors.css
│   │
│   ├── types/                   # Types TypeScript globaux
│   │   ├── api.types.ts         # Types pour les réponses API
│   │   ├── common.types.ts      # Types communs
│   │   └── env.d.ts             # Types pour les variables d'env
│   │
│   ├── assets/                  # Assets statiques
│   │   ├── images/
│   │   │   ├── logo.svg
│   │   │   └── placeholder-tree.png
│   │   └── icons/
│   │       └── custom-icons.svg
│   │
│   ├── main.tsx                 # Point d'entrée de l'application
│   └── vite-env.d.ts           # Types Vite
│
├── .env                         # Variables d'environnement (local)
├── .env.example                 # Template des variables d'env
├── components.json              # Configuration Shadcn UI
├── tailwind.config.js           # Configuration Tailwind CSS
├── tsconfig.json                # Configuration TypeScript
├── vite.config.ts              # Configuration Vite
├── package.json
└── README.md
```

---

## Architecture détaillée

### 1. Atomic Design - Composants UI (`/components`)

L'Atomic Design est une méthodologie qui décompose l'interface en 5 niveaux hiérarchiques.

#### 📦 `/components/ui/` - Shadcn UI

Composants générés automatiquement par Shadcn CLI. **Ne pas modifier directement**, utilise les props de customisation.

```bash
# Ajouter un nouveau composant Shadcn
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
```

#### ⚛️ `/components/atoms/` - Atomes

**Définition** : Plus petits composants réutilisables, éléments de base.

**Exemples** :
- `Logo` : Logo de l'application
- `Badge` : Badge de statut personnalisé
- `Icon` : Wrapper pour les icônes
- `Spinner` : Indicateur de chargement

**Règles** :
- Pas de logique métier
- Hautement réutilisables
- Props simples et claires
- Testables unitairement

```tsx
// components/atoms/Logo/Logo.tsx
interface LogoProps {
  size?: 'sm' | 'md' | 'lg';
  className?: string;
}

export const Logo = ({ size = 'md', className }: LogoProps) => {
  const sizeClasses = {
    sm: 'h-6 w-6',
    md: 'h-10 w-10',
    lg: 'h-16 w-16'
  };

  return (
    <img 
      src="/logo.svg" 
      alt="GreenRoots" 
      className={cn(sizeClasses[size], className)}
    />
  );
};
```

#### 🧬 `/components/molecules/` - Molécules

**Définition** : Combinaison de plusieurs atomes pour former une unité fonctionnelle.

**Exemples** :
- `SearchBar` : Input + Button + Icon
- `TreeCard` : Card + Image + Badge + Price
- `UserAvatar` : Avatar + Name + Status

**Règles** :
- Composent plusieurs atomes
- Peuvent avoir une logique simple
- Réutilisables dans différents contextes

```tsx
// components/molecules/SearchBar/SearchBar.tsx
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { Search } from 'lucide-react';

interface SearchBarProps {
  onSearch: (value: string) => void;
  placeholder?: string;
}

export const SearchBar = ({ onSearch, placeholder }: SearchBarProps) => {
  const [value, setValue] = useState('');

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    onSearch(value);
  };

  return (
    <form onSubmit={handleSubmit} className="flex gap-2">
      <Input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder={placeholder}
      />
      <Button type="submit" size="icon">
        <Search className="h-4 w-4" />
      </Button>
    </form>
  );
};
```

#### 🦠 `/components/organisms/` - Organismes

**Définition** : Composants complexes combinant molécules et atomes. Représentent des sections distinctes de l'interface.

**Exemples** :
- `Header` : Navigation complète
- `TreeGrid` : Grille de cartes d'arbres avec filtres
- `Footer` : Footer avec liens et informations
- `CartSummary` : Résumé complet du panier

**Règles** :
- Peuvent avoir de la logique métier
- Souvent connectés à des hooks ou stores
- Représentent des sections majeures

```tsx
// components/organisms/Header/Header.tsx
import { Logo } from '@/components/atoms/Logo';
import { NavigationMenu } from './NavigationMenu';
import { UserAvatar } from '@/components/molecules/UserAvatar';
import { useAuth } from '@/features/auth';

export const Header = () => {
  const { user, isAuthenticated } = useAuth();

  return (
    <header className="border-b">
      <div className="container mx-auto px-4 py-4">
        <div className="flex items-center justify-between">
          <Logo />
          <NavigationMenu />
          {isAuthenticated && <UserAvatar user={user} />}
        </div>
      </div>
    </header>
  );
};
```

#### 📄 `/components/templates/` - Templates

**Définition** : Layouts de pages qui définissent la structure globale sans contenu spécifique.

**Exemples** :
- `MainLayout` : Layout principal avec Header + Footer
- `DashboardLayout` : Layout avec sidebar
- `AuthLayout` : Layout pour pages d'authentification

```tsx
// components/templates/MainLayout/MainLayout.tsx
import { Header } from '@/components/organisms/Header';
import { Footer } from '@/components/organisms/Footer';
import { Outlet } from '@tanstack/react-router';

export const MainLayout = () => {
  return (
    <div className="min-h-screen flex flex-col">
      <Header />
      <main className="flex-1 container mx-auto px-4 py-8">
        <Outlet /> {/* Les routes enfants s'affichent ici */}
      </main>
      <Footer />
    </div>
  );
};
```

---

### 2. Feature-based Architecture (`/features`)

Chaque feature est un **module autonome** contenant toute sa logique.

#### 📁 Structure d'une feature

```
features/trees/
├── components/          # Composants SPÉCIFIQUES à cette feature
├── hooks/              # Hooks personnalisés
├── api/                # Appels API
├── store/              # Store Zustand (optionnel)
├── types/              # Types TypeScript
└── index.ts            # API publique (exports)
```

#### 🔐 Principe d'encapsulation

Le fichier `index.ts` agit comme une **API publique**. Seul ce qui est exporté ici peut être utilisé à l'extérieur.

```typescript
// features/trees/index.ts
// ✅ API Publique - Ce qui peut être utilisé à l'extérieur
export { TreeFilters } from './components/TreeFilters';
export { TreeDetails } from './components/TreeDetails';
export { useTreeData } from './hooks/useTreeData';
export { useTreeMutation } from './hooks/useTreeMutation';
export type { Tree, TreeFilters as TreeFiltersType } from './types/tree.types';

// ❌ Tout le reste reste PRIVÉ à la feature
// TreeInternalComponent, helpers, etc. ne sont pas exportés
```

#### 📝 Exemple complet d'une feature

```typescript
// features/trees/types/tree.types.ts
export interface Tree {
  id: string;
  name: string;
  scientificName: string;
  price: number;
  description: string;
  imageUrl: string;
  category: string;
  stockQuantity: number;
  co2Absorption: number;
}

export interface TreeFilters {
  category?: string;
  minPrice?: number;
  maxPrice?: number;
  search?: string;
}
```

```typescript
// features/trees/api/treesApi.ts
import { apiClient } from '@/lib/api/client';
import type { Tree, TreeFilters } from '../types/tree.types';

export const treesApi = {
  getAll: async (filters?: TreeFilters) => {
    const { data } = await apiClient.get<Tree[]>('/trees', { params: filters });
    return data;
  },

  getById: async (id: string) => {
    const { data } = await apiClient.get<Tree>(`/trees/${id}`);
    return data;
  },

  create: async (tree: Omit<Tree, 'id'>) => {
    const { data } = await apiClient.post<Tree>('/trees', tree);
    return data;
  }
};
```

```typescript
// features/trees/hooks/useTreeData.ts
import { useQuery } from '@tanstack/react-query';
import { treesApi } from '../api/treesApi';
import type { TreeFilters } from '../types/tree.types';

export const useTreeData = (filters?: TreeFilters) => {
  return useQuery({
    queryKey: ['trees', filters],
    queryFn: () => treesApi.getAll(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

export const useTreeById = (id: string) => {
  return useQuery({
    queryKey: ['tree', id],
    queryFn: () => treesApi.getById(id),
    enabled: !!id, // Ne lance la query que si id existe
  });
};
```

```typescript
// features/trees/hooks/useTreeMutation.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { treesApi } from '../api/treesApi';
import type { Tree } from '../types/tree.types';

export const useCreateTree = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (tree: Omit<Tree, 'id'>) => treesApi.create(tree),
    onSuccess: () => {
      // Invalide le cache pour recharger la liste
      queryClient.invalidateQueries({ queryKey: ['trees'] });
    },
  });
};
```

```tsx
// features/trees/components/TreeFilters.tsx
import { Input } from '@/components/ui/input';
import { Select } from '@/components/ui/select';
import type { TreeFilters } from '../types/tree.types';

interface TreeFiltersProps {
  filters: TreeFilters;
  onFiltersChange: (filters: TreeFilters) => void;
}

export const TreeFilters = ({ filters, onFiltersChange }: TreeFiltersProps) => {
  // Composant de filtres spécifique aux arbres
  return (
    <div className="flex gap-4">
      <Input
        placeholder="Rechercher..."
        value={filters.search || ''}
        onChange={(e) => onFiltersChange({ ...filters, search: e.target.value })}
      />
      {/* Autres filtres... */}
    </div>
  );
};
```

---

### 3. Tanstack Router (`/routes`)

**File-based routing** : La structure des fichiers définit automatiquement les routes.

#### 🗺️ Convention de nommage des routes

| Fichier | Route générée | Description |
|---------|---------------|-------------|
| `index.tsx` | `/` | Page d'accueil |
| `about.tsx` | `/about` | Page à propos |
| `trees/index.tsx` | `/trees` | Liste des arbres |
| `trees/$treeId.tsx` | `/trees/:treeId` | Détail dynamique |
| `auth/login.tsx` | `/auth/login` | Login |
| `__root.tsx` | - | Route racine (layout) |

#### 📄 Exemple de route

```tsx
// routes/__root.tsx
import { Outlet, createRootRoute } from '@tanstack/react-router';
import { MainLayout } from '@/components/templates/MainLayout';

export const Route = createRootRoute({
  component: () => (
    <MainLayout>
      <Outlet /> {/* Les routes enfants s'affichent ici */}
    </MainLayout>
  ),
});
```

```tsx
// routes/trees/index.tsx
import { createFileRoute } from '@tanstack/react-router';
import { useTreeData } from '@/features/trees';
import { TreeGrid } from '@/components/organisms/TreeGrid';

export const Route = createFileRoute('/trees/')({
  component: TreesPage,
  // Loader : charge les données avant d'afficher la page
  loader: ({ context }) => {
    context.queryClient.ensureQueryData({
      queryKey: ['trees'],
      queryFn: () => treesApi.getAll(),
    });
  },
});

function TreesPage() {
  const { data: trees, isLoading } = useTreeData();

  if (isLoading) return <Spinner />;

  return (
    <div>
      <h1>Nos arbres</h1>
      <TreeGrid trees={trees} />
    </div>
  );
}
```

```tsx
// routes/trees/$treeId.tsx
import { createFileRoute } from '@tanstack/react-router';
import { useTreeById } from '@/features/trees';

export const Route = createFileRoute('/trees/$treeId')({
  component: TreeDetailPage,
});

function TreeDetailPage() {
  const { treeId } = Route.useParams(); // Type-safe !
  const { data: tree } = useTreeById(treeId);

  return (
    <div>
      <h1>{tree.name}</h1>
      <p>{tree.description}</p>
    </div>
  );
}
```

#### 🔍 Navigation type-safe

```tsx
import { Link } from '@tanstack/react-router';

// ✅ Type-safe - auto-complétion et vérification
<Link to="/trees/$treeId" params={{ treeId: '123' }}>
  Voir l'arbre
</Link>

// ❌ Erreur TypeScript si la route n'existe pas
<Link to="/route-inexistante">Erreur</Link>
```

---

### 4. Tanstack Query - Gestion du state serveur

Tanstack Query gère **toutes les données serveur** (cache, mutations, synchronisation).

#### 🎯 Configuration

```typescript
// lib/api/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1 minute
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
});
```

```tsx
// app/providers.tsx
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/lib/api/queryClient';

export const Providers = ({ children }: { children: React.ReactNode }) => {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};
```

#### 📊 Pattern d'utilisation

**1. Définir l'API** (dans `features/*/api/`)

```typescript
export const treesApi = {
  getAll: () => apiClient.get('/trees'),
  getById: (id: string) => apiClient.get(`/trees/${id}`),
  create: (tree) => apiClient.post('/trees', tree),
  update: (id, tree) => apiClient.put(`/trees/${id}`, tree),
  delete: (id) => apiClient.delete(`/trees/${id}`),
};
```

**2. Créer les hooks** (dans `features/*/hooks/`)

```typescript
// Query (lecture)
export const useTreeData = () => {
  return useQuery({
    queryKey: ['trees'],
    queryFn: treesApi.getAll,
  });
};

// Mutation (écriture)
export const useCreateTree = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: treesApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['trees'] });
    },
  });
};
```

**3. Utiliser dans les composants**

```tsx
function TreesList() {
  const { data, isLoading, error } = useTreeData();
  const createTree = useCreateTree();

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage />;

  return (
    <div>
      {data.map(tree => <TreeCard key={tree.id} tree={tree} />)}
      <Button onClick={() => createTree.mutate(newTree)}>
        Ajouter
      </Button>
    </div>
  );
}
```

---

### 5. Zustand - Store global

Zustand gère l'**état global client-side** (UI, préférences, données persistantes).

#### 🏪 Architecture du store

**Approche recommandée** : Un store global divisé en slices.

```typescript
// store/slices/uiSlice.ts
interface UiSlice {
  theme: 'light' | 'dark';
  sidebarOpen: boolean;
  toggleTheme: () => void;
  toggleSidebar: () => void;
}

export const createUiSlice = (set: any): UiSlice => ({
  theme: 'light',
  sidebarOpen: true,
  
  toggleTheme: () => set((state: UiSlice) => ({ 
    theme: state.theme === 'light' ? 'dark' : 'light' 
  })),
  
  toggleSidebar: () => set((state: UiSlice) => ({ 
    sidebarOpen: !state.sidebarOpen 
  })),
});
```

```typescript
// store/slices/cartSlice.ts
interface CartItem {
  treeId: string;
  quantity: number;
}

interface CartSlice {
  items: CartItem[];
  addItem: (treeId: string) => void;
  removeItem: (treeId: string) => void;
  clearCart: () => void;
}

export const createCartSlice = (set: any): CartSlice => ({
  items: [],
  
  addItem: (treeId) => set((state: CartSlice) => ({
    items: [...state.items, { treeId, quantity: 1 }]
  })),
  
  removeItem: (treeId) => set((state: CartSlice) => ({
    items: state.items.filter(item => item.treeId !== treeId)
  })),
  
  clearCart: () => set({ items: [] }),
});
```

```typescript
// store/index.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { createUiSlice } from './slices/uiSlice';
import { createCartSlice } from './slices/cartSlice';

type Store = UiSlice & CartSlice;

export const useStore = create<Store>()(
  persist(
    (set, get) => ({
      ...createUiSlice(set),
      ...createCartSlice(set),
    }),
    {
      name: 'greenroots-store', // Clé localStorage
      partialize: (state) => ({ 
        // Persiste seulement certaines parties
        theme: state.theme,
        items: state.items,
      }),
    }
  )
);

// Hooks sélecteurs pour éviter les re-renders
export const useTheme = () => useStore((state) => state.theme);
export const useCart = () => useStore((state) => state.items);
export const useToggleTheme = () => useStore((state) => state.toggleTheme);
```

#### 💡 Quand utiliser Zustand vs Tanstack Query ?

| Utilise Zustand pour | Utilise Tanstack Query pour |
|---------------------|----------------------------|
| État UI (theme, sidebar) | Données serveur (arbres, users) |
| Panier (persist local) | API calls |
| Préférences utilisateur | Cache de données |
| État global synchrone | État asynchrone |

---

### 6. Shadcn UI - Composants UI

Shadcn n'est **pas une bibliothèque npm**, mais des composants que tu **copies dans ton projet**.

#### 🎨 Installation et configuration

```bash
# Initialisation
npx shadcn-ui@latest init

# Ajouter des composants
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add input
npx shadcn-ui@latest add dialog
```

Configuration automatique dans `components.json` :

```json
{
  "style": "default",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.js",
    "css": "src/styles/globals.css",
    "baseColor": "slate",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils"
  }
}
```

#### 🎭 Customisation

```tsx
// Utilisation basique
import { Button } from '@/components/ui/button';

<Button>Cliquer</Button>

// Avec variants
<Button variant="destructive">Supprimer</Button>
<Button variant="outline" size="sm">Petit</Button>

// Customisation avec className
<Button className="bg-green-600 hover:bg-green-700">
  Custom
</Button>
```

---

## Conventions de nommage

### 📝 Fichiers et dossiers

| Type | Convention | Exemple |
|------|-----------|---------|
| Composants React | PascalCase | `TreeCard.tsx` |
| Hooks | camelCase | `useTreeData.ts` |
| Utils/helpers | camelCase | `formatPrice.ts` |
| Types | PascalCase | `tree.types.ts` |
| Constants | UPPER_SNAKE_CASE | `API_ENDPOINTS.ts` |
| Dossiers | kebab-case | `tree-filters/` |

### 🏷️ Variables et fonctions

```typescript
// ✅ Bon
const treeName = "Chêne";
const isAvailable = true;
const MAX_TREES = 100;

function calculatePrice(quantity: number) {}
const handleSubmit = () => {};

// ❌ Éviter
const TreeName = "Chêne";
const available = true;
const maxTrees = 100;
```

### 📦 Exports

```typescript
// ✅ Named exports (préféré)
export const TreeCard = () => {};
export const useTreeData = () => {};

// ✅ Index files pour simplifier imports
// features/trees/index.ts
export * from './components';
export * from './hooks';

// ✅ Import
import { TreeCard, useTreeData } from '@/features/trees';
```

---

## Guides d'utilisation

### 🚀 Créer une nouvelle feature

**Étape 1** : Créer la structure

```bash
mkdir -p src/features/ma-feature/{components,hooks,api,types}
touch src/features/ma-feature/index.ts
```

**Étape 2** : Définir les types

```typescript
// features/ma-feature/types/ma-feature.types.ts
export interface MaFeature {
  id: string;
  name: string;
  // ...
}
```

**Étape 3** : Créer l'API

```typescript
// features/ma-feature/api/maFeatureApi.ts
import { apiClient } from '@/lib/api/client';

export const maFeatureApi = {
  getAll: () => apiClient.get('/ma-feature'),
  getById: (id: string) => apiClient.get(`/ma-feature/${id}`),
};
```

**Étape 4** : Créer les hooks

```typescript
// features/ma-feature/hooks/useMaFeatureData.ts
import { useQuery } from '@tanstack/react-query';
import { maFeatureApi } from '../api/maFeatureApi';

export const useMaFeatureData = () => {
  return useQuery({
    queryKey: ['ma-feature'],
    queryFn: maFeatureApi.getAll,
  });
};
```

**Étape 5** : Exporter l'API publique

```typescript
// features/ma-feature/index.ts
export { useMaFeatureData } from './hooks/useMaFeatureData';
export type { MaFeature } from './types/ma-feature.types';
```

---

### 🧩 Créer un nouveau composant

**Composant Atom (simple)**

```tsx
// components/atoms/Badge/Badge.tsx
import { cn } from '@/lib/utils';

interface BadgeProps {
  variant?: 'default' | 'success' | 'warning' | 'error';
  children: React.ReactNode;
  className?: string;
}

export const Badge = ({ variant = 'default', children, className }: BadgeProps) => {
  const variants = {
    default: 'bg-gray-100 text-gray-800',
    success: 'bg-green-100 text-green-800',
    warning: 'bg-yellow-100 text-yellow-800',
    error: 'bg-red-100 text-red-800',
  };

  return (
    <span className={cn(
      'px-2 py-1 rounded-full text-xs font-medium',
      variants[variant],
      className
    )}>
      {children}
    </span>
  );
};

// components/atoms/Badge/index.ts
export { Badge } from './Badge';
```

**Composant Molecule (avec logique)**

```tsx
// components/molecules/PriceTag/PriceTag.tsx
import { Badge } from '@/components/atoms/Badge';
import { formatPrice } from '@/lib/utils';

interface PriceTagProps {
  price: number;
  currency?: string;
  isPromotion?: boolean;
  promotionPrice?: number;
}

export const PriceTag = ({ 
  price, 
  currency = '€', 
  isPromotion = false,
  promotionPrice 
}: PriceTagProps) => {
  return (
    <div className="flex items-center gap-2">
      {isPromotion && promotionPrice ? (
        <>
          <span className="line-through text-gray-500">
            {formatPrice(price, currency)}
          </span>
          <span className="text-xl font-bold text-green-600">
            {formatPrice(promotionPrice, currency)}
          </span>
          <Badge variant="success">Promo</Badge>
        </>
      ) : (
        <span className="text-xl font-bold">
          {formatPrice(price, currency)}
        </span>
      )}
    </div>
  );
};

// components/molecules/PriceTag/index.ts
export { PriceTag } from './PriceTag';
```

---

### 🛣️ Créer une nouvelle route

**Route simple**

```tsx
// routes/about.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/about')({
  component: AboutPage,
});

function AboutPage() {
  return (
    <div>
      <h1>À propos de GreenRoots</h1>
      <p>Notre mission...</p>
    </div>
  );
}
```

**Route avec paramètre dynamique**

```tsx
// routes/trees/$treeId.tsx
import { createFileRoute } from '@tanstack/react-router';
import { useTreeById } from '@/features/trees';

export const Route = createFileRoute('/trees/$treeId')({
  component: TreeDetailPage,
  // Loader pour précharger les données
  loader: ({ params, context }) => {
    return context.queryClient.ensureQueryData({
      queryKey: ['tree', params.treeId],
      queryFn: () => treesApi.getById(params.treeId),
    });
  },
});

function TreeDetailPage() {
  const { treeId } = Route.useParams(); // Type-safe
  const { data: tree, isLoading } = useTreeById(treeId);

  if (isLoading) return <Spinner />;

  return (
    <div>
      <h1>{tree.name}</h1>
      <img src={tree.imageUrl} alt={tree.name} />
      <p>{tree.description}</p>
      <PriceTag price={tree.price} />
    </div>
  );
}
```

**Route avec recherche (search params)**

```tsx
// routes/trees/index.tsx
import { createFileRoute } from '@tanstack/react-router';
import { z } from 'zod';

const treeSearchSchema = z.object({
  category: z.string().optional(),
  minPrice: z.number().optional(),
  maxPrice: z.number().optional(),
  search: z.string().optional(),
});

export const Route = createFileRoute('/trees/')({
  component: TreesPage,
  validateSearch: treeSearchSchema, // Validation type-safe
});

function TreesPage() {
  const search = Route.useSearch(); // Type-safe
  const navigate = Route.useNavigate();
  
  const { data: trees } = useTreeData(search);

  const handleFilterChange = (newFilters: typeof search) => {
    navigate({ search: newFilters });
  };

  return (
    <div>
      <TreeFilters filters={search} onFiltersChange={handleFilterChange} />
      <TreeGrid trees={trees} />
    </div>
  );
}
```

---

### 🎣 Utiliser Tanstack Query

**Query simple (GET)**

```typescript
import { useQuery } from '@tanstack/react-query';

const { data, isLoading, error, refetch } = useQuery({
  queryKey: ['trees'], // Clé unique pour le cache
  queryFn: () => treesApi.getAll(),
  staleTime: 5 * 60 * 1000, // 5 minutes
  gcTime: 10 * 60 * 1000, // 10 minutes (anciennement cacheTime)
  retry: 3,
});
```

**Query avec paramètres**

```typescript
const { data } = useQuery({
  queryKey: ['trees', filters], // Clé change si filters change
  queryFn: () => treesApi.getAll(filters),
  enabled: !!filters, // Ne lance que si filters existe
});
```

**Mutation (POST/PUT/DELETE)**

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';

const queryClient = useQueryClient();

const createTree = useMutation({
  mutationFn: (tree: NewTree) => treesApi.create(tree),
  
  onSuccess: (data) => {
    // Invalide le cache pour refetch
    queryClient.invalidateQueries({ queryKey: ['trees'] });
    
    // Ou met à jour directement
    queryClient.setQueryData(['tree', data.id], data);
    
    toast.success('Arbre créé !');
  },
  
  onError: (error) => {
    toast.error('Erreur lors de la création');
  },
});

// Utilisation
const handleCreate = () => {
  createTree.mutate(newTreeData);
};
```

**Prefetch (optimisation)**

```typescript
// Dans un loader de route
loader: ({ context }) => {
  // Précharge les données avant d'afficher la page
  return context.queryClient.ensureQueryData({
    queryKey: ['trees'],
    queryFn: treesApi.getAll,
  });
}
```

---

### 🏪 Utiliser Zustand

**Accéder au store**

```tsx
import { useStore } from '@/store';

function MyComponent() {
  // ❌ Mauvais : récupère tout le store (re-render à chaque changement)
  const store = useStore();
  
  // ✅ Bon : sélecteur (re-render seulement si theme change)
  const theme = useStore((state) => state.theme);
  const toggleTheme = useStore((state) => state.toggleTheme);
  
  return (
    <button onClick={toggleTheme}>
      Theme: {theme}
    </button>
  );
}
```

**Hook personnalisé pour le store**

```typescript
// store/index.ts
export const useTheme = () => useStore((state) => ({
  theme: state.theme,
  toggleTheme: state.toggleTheme,
}));

// Utilisation
const { theme, toggleTheme } = useTheme();
```

**Actions asynchrones**

```typescript
// store/slices/authSlice.ts
interface AuthSlice {
  user: User | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

export const createAuthSlice = (set: any): AuthSlice => ({
  user: null,
  
  login: async (credentials) => {
    const user = await authApi.login(credentials);
    set({ user });
  },
  
  logout: () => set({ user: null }),
});
```

---

## Bonnes pratiques

### ✅ Général

1. **Colocate** : Garde le code proche de son utilisation
   ```
   ✅ features/trees/components/TreeCard.tsx
   ❌ components/TreeCard.tsx (si utilisé que dans trees)
   ```

2. **DRY (Don't Repeat Yourself)** : Si tu copies du code 3 fois, crée une abstraction

3. **Single Responsibility** : Un fichier = une responsabilité

4. **Type tout** : Utilise TypeScript partout, pas de `any`

5. **Imports absolus** : Utilise les alias (`@/...`) plutôt que les chemins relatifs

### ✅ Composants

```tsx
// ✅ Bon
interface Props {
  tree: Tree;
  onSelect: (id: string) => void;
}

export const TreeCard = ({ tree, onSelect }: Props) => {
  return (
    <Card onClick={() => onSelect(tree.id)}>
      <h3>{tree.name}</h3>
    </Card>
  );
};

// ❌ Éviter
export const TreeCard = (props: any) => { // any à éviter
  return <Card onClick={props.onClick}>...</Card>; // props mal structurés
};
```

### ✅ Hooks

```typescript
// ✅ Bon : Hook simple et réutilisable
export const useTreeData = (filters?: TreeFilters) => {
  return useQuery({
    queryKey: ['trees', filters],
    queryFn: () => treesApi.getAll(filters),
  });
};

// ❌ Éviter : Hook qui fait trop de choses
export const useEverything = () => {
  const trees = useQuery(...);
  const user = useQuery(...);
  const cart = useStore(...);
  // Trop de responsabilités
};
```

### ✅ État

**Règle d'or** : Commence local, passe au global si nécessaire

```tsx
// ✅ État local pour UI simple
const [isOpen, setIsOpen] = useState(false);

// ✅ Tanstack Query pour données serveur
const { data } = useTreeData();

// ✅ Zustand pour état global client
const cart = useStore((state) => state.cart);
```

### ✅ Performance

1. **Mémoization** : Utilise `useMemo` et `useCallback` judicieusement

```tsx
const expensiveValue = useMemo(
  () => heavyCalculation(data),
  [data]
);

const handleClick = useCallback(
  () => doSomething(id),
  [id]
);
```

2. **Lazy loading** : Charge les routes à la demande

```tsx
// routes/admin/index.tsx
export const Route = createFileRoute('/admin/')({
  component: lazy(() => import('./AdminPage')),
});
```

3. **Sélecteurs Zustand** : Évite les re-renders inutiles

```tsx
// ✅ Bon
const theme = useStore((state) => state.theme);

// ❌ Mauvais
const { theme, cart, user } = useStore(); // Re-render si n'importe quoi change
```

### ✅ Code review checklist

Avant de commit, vérifie :

- [ ] Pas de `console.log` restants
- [ ] Pas de `any` TypeScript
- [ ] Composants < 200 lignes
- [ ] Fichiers bien organisés dans la bonne feature/dossier
- [ ] Types exportés et réutilisables
- [ ] Pas de code dupliqué
- [ ] Imports triés et propres
- [ ] Nommage cohérent avec les conventions

---

## Configuration

### 📦 Vite Config

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { TanStackRouterVite } from '@tanstack/router-vite-plugin';
import path from 'path';

export default defineConfig({
  plugins: [
    react(),
    TanStackRouterVite(), // Génère les routes automatiquement
  ],
  
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@features': path.resolve(__dirname, './src/features'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@lib': path.resolve(__dirname, './src/lib'),
      '@assets': path.resolve(__dirname, './src/assets'),
      '@routes': path.resolve(__dirname, './src/routes'),
    },
  },
  
  server: {
    port: 3000,
    open: true,
  },
});
```

### 🎨 Tailwind Config

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: ['class'],
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        // Couleurs Shadcn (CSS variables)
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        // Couleurs custom pour GreenRoots
        green: {
          50: '#f0fdf4',
          500: '#22c55e',
          600: '#16a34a',
          700: '#15803d',
        },
      },
      fontFamily: {
        sans: ['TaFontPreferee', 'system-ui', 'sans-serif'],
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};
```

### 📝 TypeScript Config

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    /* Path mapping */
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@features/*": ["./src/features/*"],
      "@hooks/*": ["./src/hooks/*"],
      "@lib/*": ["./src/lib/*"],
      "@assets/*": ["./src/assets/*"],
      "@routes/*": ["./src/routes/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 🔐 Variables d'environnement

```bash
# .env.example
VITE_API_URL=http://localhost:8000/api
VITE_APP_NAME=GreenRoots
VITE_ENABLE_DEVTOOLS=true
```

```bash
# .env (à créer localement, pas dans git)
VITE_API_URL=http://localhost:8000/api
VITE_APP_NAME=GreenRoots
VITE_ENABLE_DEVTOOLS=true
```

```typescript
// src/lib/constants.ts
export const API_URL = import.meta.env.VITE_API_URL;
export const APP_NAME = import.meta.env.VITE_APP_NAME;
export const ENABLE_DEVTOOLS = import.meta.env.VITE_ENABLE_DEVTOOLS === 'true';
```

### 📋 Schémas de validation Zod

**Zod** est une bibliothèque de validation de schémas TypeScript-first. Elle permet de :
- Valider les données de formulaires
- Valider les réponses API
- Générer automatiquement des types TypeScript
- Intégrer avec React Hook Form

#### Installation

```bash
npm install zod
npm install @hookform/resolvers # Pour l'intégration React Hook Form
```

#### Structure des schémas

```
lib/
└── validations/
    ├── tree.schema.ts      # Schémas pour les arbres
    ├── auth.schema.ts      # Schémas d'authentification
    ├── cart.schema.ts      # Schémas panier
    └── common.schema.ts    # Schémas réutilisables
```

#### Schémas communs réutilisables

```typescript
// lib/validations/common.schema.ts
import { z } from 'zod';

// Email
export const emailSchema = z
  .string()
  .min(1, 'L\'email est requis')
  .email('Email invalide');

// Password
export const passwordSchema = z
  .string()
  .min(8, 'Le mot de passe doit contenir au moins 8 caractères')
  .regex(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
    'Le mot de passe doit contenir une majuscule, une minuscule et un chiffre'
  );

// Téléphone français
export const phoneSchema = z
  .string()
  .regex(/^(?:(?:\+|00)33|0)\s*[1-9](?:[\s.-]*\d{2}){4}$/, 'Numéro de téléphone invalide');

// Prix
export const priceSchema = z
  .number()
  .positive('Le prix doit être positif')
  .min(0.01, 'Le prix minimum est 0.01€');

// Quantité
export const quantitySchema = z
  .number()
  .int('La quantité doit être un nombre entier')
  .positive('La quantité doit être positive')
  .min(1, 'La quantité minimum est 1');

// Date future
export const futureDateSchema = z
  .date()
  .refine((date) => date > new Date(), 'La date doit être dans le futur');
```

#### Schémas pour les arbres

```typescript
// lib/validations/tree.schema.ts
import { z } from 'zod';
import { priceSchema, quantitySchema } from './common.schema';

// Énumération des catégories
export const TreeCategoryEnum = z.enum([
  'FRUITIER',
  'ORNAMENTAL',
  'FORESTIER',
  'ARBUSTE',
]);

// Schéma complet d'un arbre
export const treeSchema = z.object({
  id: z.string().uuid(),
  name: z
    .string()
    .min(2, 'Le nom doit contenir au moins 2 caractères')
    .max(100, 'Le nom ne peut pas dépasser 100 caractères'),
  scientificName: z
    .string()
    .min(2, 'Le nom scientifique est requis')
    .optional(),
  description: z
    .string()
    .min(10, 'La description doit contenir au moins 10 caractères')
    .max(2000, 'La description ne peut pas dépasser 2000 caractères'),
  price: priceSchema,
  stockQuantity: quantitySchema,
  category: TreeCategoryEnum,
  imageUrl: z
    .string()
    .url('URL d\'image invalide')
    .optional(),
  height: z
    .number()
    .positive('La hauteur doit être positive')
    .optional(),
  co2Absorption: z
    .number()
    .nonnegative('L\'absorption de CO2 ne peut pas être négative')
    .optional(),
  plantingDifficulty: z.enum(['FACILE', 'MOYEN', 'DIFFICILE']).optional(),
  sunExposure: z.enum(['PLEIN_SOLEIL', 'MI_OMBRE', 'OMBRE']).optional(),
  wateringFrequency: z.enum(['FAIBLE', 'MOYEN', 'FREQUENT']).optional(),
  createdAt: z.date(),
  updatedAt: z.date(),
});

// Schéma pour la création (sans id, createdAt, updatedAt)
export const createTreeSchema = treeSchema.omit({
  id: true,
  createdAt: true,
  updatedAt: true,
});

// Schéma pour la mise à jour (tout optionnel sauf id)
export const updateTreeSchema = treeSchema.partial().required({ id: true });

// Schéma pour les filtres de recherche
export const treeFiltersSchema = z.object({
  search: z.string().optional(),
  category: TreeCategoryEnum.optional(),
  minPrice: z.number().nonnegative().optional(),
  maxPrice: z.number().nonnegative().optional(),
  inStock: z.boolean().optional(),
  sortBy: z.enum(['name', 'price', 'createdAt']).optional(),
  sortOrder: z.enum(['asc', 'desc']).optional(),
  page: z.number().int().positive().default(1),
  limit: z.number().int().positive().max(100).default(20),
});

// Types TypeScript générés automatiquement
export type Tree = z.infer<typeof treeSchema>;
export type CreateTree = z.infer<typeof createTreeSchema>;
export type UpdateTree = z.infer<typeof updateTreeSchema>;
export type TreeFilters = z.infer<typeof treeFiltersSchema>;
export type TreeCategory = z.infer<typeof TreeCategoryEnum>;
```

#### Schémas d'authentification

```typescript
// lib/validations/auth.schema.ts
import { z } from 'zod';
import { emailSchema, passwordSchema, phoneSchema } from './common.schema';

// Schéma de connexion
export const loginSchema = z.object({
  email: emailSchema,
  password: z.string().min(1, 'Le mot de passe est requis'),
  rememberMe: z.boolean().optional(),
});

// Schéma d'inscription
export const registerSchema = z
  .object({
    email: emailSchema,
    password: passwordSchema,
    confirmPassword: z.string(),
    firstName: z
      .string()
      .min(2, 'Le prénom doit contenir au moins 2 caractères')
      .max(50, 'Le prénom ne peut pas dépasser 50 caractères'),
    lastName: z
      .string()
      .min(2, 'Le nom doit contenir au moins 2 caractères')
      .max(50, 'Le nom ne peut pas dépasser 50 caractères'),
    phone: phoneSchema.optional(),
    acceptTerms: z
      .boolean()
      .refine((val) => val === true, 'Vous devez accepter les conditions'),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: 'Les mots de passe ne correspondent pas',
    path: ['confirmPassword'], // Erreur sur le champ confirmPassword
  });

// Schéma de réinitialisation de mot de passe
export const resetPasswordSchema = z
  .object({
    token: z.string().min(1, 'Token invalide'),
    password: passwordSchema,
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: 'Les mots de passe ne correspondent pas',
    path: ['confirmPassword'],
  });

// Schéma de demande de réinitialisation
export const forgotPasswordSchema = z.object({
  email: emailSchema,
});

// Types générés
export type LoginInput = z.infer<typeof loginSchema>;
export type RegisterInput = z.infer<typeof registerSchema>;
export type ResetPasswordInput = z.infer<typeof resetPasswordSchema>;
export type ForgotPasswordInput = z.infer<typeof forgotPasswordSchema>;
```

#### Schémas pour le panier et les commandes

```typescript
// lib/validations/cart.schema.ts
import { z } from 'zod';
import { quantitySchema } from './common.schema';

// Item du panier
export const cartItemSchema = z.object({
  treeId: z.string().uuid(),
  quantity: quantitySchema,
  price: z.number().positive(),
});

// Panier complet
export const cartSchema = z.object({
  items: z.array(cartItemSchema),
  total: z.number().nonnegative(),
});

// Adresse de livraison
export const addressSchema = z.object({
  street: z.string().min(5, 'L\'adresse doit contenir au moins 5 caractères'),
  city: z.string().min(2, 'La ville est requise'),
  postalCode: z
    .string()
    .regex(/^\d{5}$/, 'Le code postal doit contenir 5 chiffres'),
  country: z.string().default('France'),
  additionalInfo: z.string().max(200).optional(),
});

// Schéma de checkout
export const checkoutSchema = z.object({
  shippingAddress: addressSchema,
  billingAddress: addressSchema.optional(),
  useSameAddress: z.boolean().default(true),
  paymentMethod: z.enum(['CARD', 'PAYPAL', 'TRANSFER']),
  deliveryNotes: z.string().max(500).optional(),
});

// Types générés
export type CartItem = z.infer<typeof cartItemSchema>;
export type Cart = z.infer<typeof cartSchema>;
export type Address = z.infer<typeof addressSchema>;
export type CheckoutInput = z.infer<typeof checkoutSchema>;
```

#### Utilisation avec React Hook Form

```tsx
// features/auth/components/LoginForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { loginSchema, type LoginInput } from '@/lib/validations/auth.schema';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';

export const LoginForm = () => {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginInput>({
    resolver: zodResolver(loginSchema), // Validation Zod
    defaultValues: {
      email: '',
      password: '',
      rememberMe: false,
    },
  });

  const onSubmit = async (data: LoginInput) => {
    try {
      // data est typé et validé automatiquement
      await authApi.login(data);
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <Label htmlFor="email">Email</Label>
        <Input
          id="email"
          type="email"
          {...register('email')}
          aria-invalid={!!errors.email}
        />
        {errors.email && (
          <p className="text-sm text-red-600 mt-1">{errors.email.message}</p>
        )}
      </div>

      <div>
        <Label htmlFor="password">Mot de passe</Label>
        <Input
          id="password"
          type="password"
          {...register('password')}
          aria-invalid={!!errors.password}
        />
        {errors.password && (
          <p className="text-sm text-red-600 mt-1">{errors.password.message}</p>
        )}
      </div>

      <div className="flex items-center">
        <input
          id="rememberMe"
          type="checkbox"
          {...register('rememberMe')}
          className="mr-2"
        />
        <Label htmlFor="rememberMe">Se souvenir de moi</Label>
      </div>

      <Button type="submit" disabled={isSubmitting} className="w-full">
        {isSubmitting ? 'Connexion...' : 'Se connecter'}
      </Button>
    </form>
  );
};
```

#### Validation des réponses API

```typescript
// features/trees/api/treesApi.ts
import { apiClient } from '@/lib/api/client';
import { treeSchema, type Tree } from '@/lib/validations/tree.schema';
import { z } from 'zod';

// Schéma de réponse paginée
const paginatedTreesSchema = z.object({
  data: z.array(treeSchema),
  total: z.number(),
  page: z.number(),
  limit: z.number(),
});

export const treesApi = {
  getAll: async (filters?: TreeFilters) => {
    const { data } = await apiClient.get('/trees', { params: filters });
    
    // Validation de la réponse API
    const validated = paginatedTreesSchema.parse(data);
    return validated;
  },

  getById: async (id: string) => {
    const { data } = await apiClient.get(`/trees/${id}`);
    
    // Validation avec gestion d'erreur
    try {
      const validated = treeSchema.parse(data);
      return validated;
    } catch (error) {
      if (error instanceof z.ZodError) {
        console.error('Erreur de validation:', error.errors);
        throw new Error('Données invalides reçues du serveur');
      }
      throw error;
    }
  },

  create: async (tree: CreateTree) => {
    // Validation avant l'envoi
    const validated = createTreeSchema.parse(tree);
    
    const { data } = await apiClient.post('/trees', validated);
    return treeSchema.parse(data);
  },
};
```

#### Validation côté Tanstack Router (Search Params)

```tsx
// routes/trees/index.tsx
import { createFileRoute } from '@tanstack/react-router';
import { treeFiltersSchema } from '@/lib/validations/tree.schema';

export const Route = createFileRoute('/trees/')({
  component: TreesPage,
  validateSearch: (search) => treeFiltersSchema.parse(search), // Validation Zod
});

function TreesPage() {
  const filters = Route.useSearch(); // Type-safe et validé
  // filters.page est garanti d'être un nombre entre 1 et Infinity
  // filters.category est garanti d'être une valeur valide de l'enum
  
  return <div>...</div>;
}
```

#### Transformations avec Zod

```typescript
// lib/validations/tree.schema.ts

// Transformer une string en nombre
export const priceInputSchema = z.object({
  price: z
    .string()
    .transform((val) => parseFloat(val))
    .pipe(priceSchema), // Puis valider avec le schéma de prix
});

// Transformer une date string en Date
export const dateInputSchema = z.object({
  date: z
    .string()
    .transform((str) => new Date(str))
    .pipe(z.date()),
});

// Nettoyer les espaces
export const trimmedStringSchema = z
  .string()
  .transform((str) => str.trim())
  .pipe(z.string().min(1, 'Le champ ne peut pas être vide'));
```

#### Validation conditionnelle

```typescript
// lib/validations/auth.schema.ts

// Schéma d'adresse conditionnelle
export const orderSchema = z
  .object({
    useSameAddress: z.boolean(),
    shippingAddress: addressSchema,
    billingAddress: addressSchema.optional(),
  })
  .refine(
    (data) => {
      // Si useSameAddress est false, billingAddress est requise
      if (!data.useSameAddress && !data.billingAddress) {
        return false;
      }
      return true;
    },
    {
      message: 'L\'adresse de facturation est requise',
      path: ['billingAddress'],
    }
  );
```

#### Helper de validation

```typescript
// lib/utils/validation.ts
import { z } from 'zod';

/**
 * Valide des données avec un schéma Zod
 * Retourne les données validées ou null avec les erreurs
 */
export function validateData<T extends z.ZodTypeAny>(
  schema: T,
  data: unknown
): { success: true; data: z.infer<T> } | { success: false; errors: z.ZodError } {
  try {
    const validated = schema.parse(data);
    return { success: true, data: validated };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error };
    }
    throw error;
  }
}

// Utilisation
const result = validateData(loginSchema, formData);
if (result.success) {
  console.log(result.data); // Typé automatiquement
} else {
  console.error(result.errors.format());
}
```

#### Messages d'erreur personnalisés

```typescript
// lib/validations/custom-errors.ts
import { z } from 'zod';

// Configuration globale des messages
z.setErrorMap((issue, ctx) => {
  if (issue.code === z.ZodIssueCode.invalid_type) {
    if (issue.expected === 'string') {
      return { message: 'Ce champ doit être du texte' };
    }
  }
  if (issue.code === z.ZodIssueCode.too_small) {
    if (issue.minimum === 1) {
      return { message: 'Ce champ est requis' };
    }
  }
  return { message: ctx.defaultError };
});
```

#### Bonnes pratiques Zod

1. **Centraliser les schémas** : Un fichier par domaine dans `/lib/validations/`

2. **Réutiliser les schémas communs** : Email, password, téléphone, etc.

3. **Générer les types** : Utilise `z.infer<typeof schema>` pour les types TypeScript

4. **Valider les réponses API** : Toujours valider les données venant du backend

5. **Messages clairs** : Toujours fournir des messages d'erreur explicites

6. **Transformer les données** : Utilise `.transform()` pour nettoyer/formater

7. **Validation côté client ET serveur** : Zod fonctionne aussi en backend Node.js

### 🎯 Client API

```typescript
// lib/api/client.ts
import axios from 'axios';
import { API_URL } from '@/lib/constants';

export const apiClient = axios.create({
  baseURL: API_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Intercepteur pour ajouter le token
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Intercepteur pour gérer les erreurs
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Rediriger vers login
      window.location.href = '/auth/login';
    }
    return Promise.reject(error);
  }
);
```

### 📱 Point d'entrée

```tsx
// main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { RouterProvider, createRouter } from '@tanstack/react-router';
import { QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { queryClient } from '@/lib/api/queryClient';
import { routeTree } from './routeTree.gen'; // Auto-généré
import '@/styles/globals.css';

// Créer le router
const router = createRouter({
  routeTree,
  context: {
    queryClient,
  },
  defaultPreload: 'intent', // Précharge au hover
});

// Type-safety pour le router
declare module '@tanstack/react-router' {
  interface Register {
    router: typeof router;
  }
}

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router} />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </React.StrictMode>
);
```

---

## 🚀 Démarrage

### Installation

```bash
# Cloner le repo
git clone [url-repo]
cd greenroots-frontend

# Installer les dépendances
npm install

# Installer Zod et React Hook Form
npm install zod react-hook-form @hookform/resolvers

# Configurer les variables d'env
cp .env.example .env

# Initialiser Shadcn UI (si pas déjà fait)
npx shadcn-ui@latest init

# Démarrer le serveur de dev
npm run dev
```

### Scripts disponibles

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "type-check": "tsc --noEmit"
  }
}
```

---

## 📚 Ressources

### Documentation officielle
- [React](https://react.dev/)
- [Vite](https://vitejs.dev/)
- [Tanstack Router](https://tanstack.com/router)
- [Tanstack Query](https://tanstack.com/query)
- [Zustand](https://zustand-demo.pmnd.rs/)
- [Shadcn/ui](https://ui.shadcn.com/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Zod](https://zod.dev/)
- [React Hook Form](https://react-hook-form.com/)

### Atomic Design
- [Atomic Design by Brad Frost](https://atomicdesign.bradfrost.com/)

---

## 📄 Licence

Ce document fait partie du projet GreenRoots.

**Dernière mise à jour** : Novembre 2024
**Version** : 1.0.0
