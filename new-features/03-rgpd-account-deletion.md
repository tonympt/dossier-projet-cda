# Feature 03 — RGPD : Suppression de compte + anonymisation

## Priorité : P2 | Estimation : ~3-4h

## Contexte juridique
- **RGPD Article 17** (droit à l'effacement) : l'utilisateur doit pouvoir demander la suppression de ses données personnelles
- **Code de Commerce L123-22** : obligation de conservation des pièces comptables pendant **6 ans**
- **Conflit** : on ne peut ni supprimer les commandes (obligation comptable) ni garder les données personnelles (RGPD)
- **Solution** : pattern **anonymisation** — les commandes restent en base (montants, articles, factures) mais toutes les données permettant d'identifier la personne sont remplacées par des valeurs génériques

## Conséquences sur les données

### Ce qui est SUPPRIMÉ (définitivement)
| Table | Données | Raison |
|---|---|---|
| `User` | Compte complet (email, nom, prénom, mot de passe, avatar, tokens) | Droit à l'effacement RGPD |
| `Cart` + `CartItem` | Panier et son contenu | Plus d'utilisateur = plus de panier |

### Ce qui est ANONYMISÉ (conservé sans données personnelles)
| Champ | Avant | Après | Raison |
|---|---|---|---|
| `Order.contactEmail` | tony@gmail.com | `deleted@anonymized.local` | Conservation comptable, email anonymisé |
| `Order.userId` | 42 | 42 (orphelin, user supprimé) | FK reste mais ne pointe plus vers rien |
| `OrderAddress.firstName` | "Tony" | "Utilisateur supprimé" | Anonymisation identité |
| `OrderAddress.lastName` | "Memponteil" | "" | Anonymisation identité |
| `OrderAddress.street` | "12 rue des Lilas" | "Adresse supprimée" | Anonymisation adresse |
| `OrderAddress.postalCode` | "69001" | "00000" | Anonymisation localisation |
| `OrderAddress.city` | "Lyon" | "Ville supprimée" | Anonymisation localisation |
| `OrderAddress.phone` | "06 12 34 56 78" | null | Suppression numéro |

### Ce qui est CONSERVÉ intact
| Donnée | Raison |
|---|---|
| `Order.totalAmount`, `totalExclTax`, `totalVatAmount` | Montants = données comptables obligatoires |
| `Order.invoiceNumber`, `invoiceDate` | Numéro de facture = pièce comptable |
| `Order.orderStatus`, `paymentStatus` | Suivi comptable |
| `OrderItem` (quantité, prix unitaire, arbre) | Détail de la commande = pièce comptable |
| `OrderAddress.countryCode` | Code pays (pas une donnée personnelle, nécessaire pour la TVA) |

### Pourquoi anonymisation et pas CASCADE DELETE ?
Un `CASCADE DELETE` supprimerait les commandes, ce qui violerait l'obligation comptable de 6 ans. L'anonymisation permet de respecter les **deux** obligations légales simultanément : la commande existe toujours comptablement, mais on ne peut plus identifier **qui** l'a passée.

## Flux utilisateur
1. L'utilisateur va sur `/profile/me`
2. Il voit la section rouge "Supprimer mon compte" en bas de page
3. Il clique sur "Supprimer mon compte" → un AlertDialog s'ouvre
4. Il saisit son mot de passe pour confirmer (sécurité contre les suppressions accidentelles)
5. Clic "Supprimer définitivement"
6. **Backend** : vérification mot de passe (bcrypt) → transaction Prisma (anonymisation → suppression)
7. **Frontend** : cache TanStack Query vidé → logout Zustand → redirection vers l'accueil
8. Toast "Votre compte a été supprimé avec succès."

## Vérification en base après suppression
```sql
-- Commandes anonymisées (user supprimé)
SELECT id, "userId", "contactEmail", "orderStatus"
FROM "order" WHERE "contactEmail" = 'deleted@anonymized.local';

-- Adresses anonymisées
SELECT oa."firstName", oa."lastName", oa.street, oa.city
FROM order_address oa
JOIN "order" o ON oa."orderId" = o.id
WHERE o."contactEmail" = 'deleted@anonymized.local';

-- User supprimé
SELECT * FROM "user" WHERE id = <ancien_id>;  -- 0 résultat
```

## Modifications techniques

### Backend

#### `app/backend/src/auth/dtos/delete-account.dto.ts` (nouveau)
- DTO `DeleteAccountDto` avec validation `@IsString()` + `@IsNotEmpty()` sur le champ `password`
- Le mot de passe est requis comme mesure de sécurité (empêche une suppression accidentelle ou par un tiers ayant accès à la session)

#### `app/backend/src/users/users.service.ts`
- Nouvelle méthode `deleteAccount(userId)` : **transaction Prisma atomique** qui exécute 4 opérations dans l'ordre :
  1. `updateMany` sur `OrderAddress` (via relation `order.userId`) → anonymisation
  2. `updateMany` sur `Order` → contactEmail anonymisé
  3. `deleteMany` sur `Cart` → suppression panier
  4. `delete` sur `User` → suppression du compte
- Si une étape échoue, tout est rollback automatiquement (c'est le principe de la transaction)

#### `app/backend/src/auth/auth.service.ts`
- Nouvelle méthode `deleteAccount(userId, password)` :
  1. Récupère le profil utilisateur
  2. Récupère le user avec mot de passe hashé (via `findOneByEmailForAuth`)
  3. Compare le mot de passe fourni avec le hash en base (`bcrypt.compare`)
  4. Si valide : appelle `usersService.deleteAccount()` (la transaction)
  5. Log : `Compte supprimé (RGPD) pour l'utilisateur ID: X`

#### `app/backend/src/auth/auth.controller.ts`
- Nouvel endpoint `DELETE /auth/me`
  - Guards : `JwtAuthGuard` (utilisateur connecté obligatoire)
  - Body : `{ password: string }` (validé par `DeleteAccountDto`)
  - Réponse 200 : `{ message: "Votre compte a été supprimé..." }`
  - Réponse 401 : mot de passe incorrect
  - Swagger documenté

### Frontend

#### `app/frontend/src/features/users/api/usersApi.ts`
- Nouvelle méthode `deleteMyAccount(password)` → `DELETE /auth/me` avec le password dans le body

#### `app/frontend/src/features/users/components/DeleteAccountSection.tsx` (nouveau)
- Section avec bordure rouge (`border-red-200`) pour signaler le danger
- Texte explicatif sur l'irréversibilité et l'anonymisation
- Bouton `variant="destructive"` (rouge)
- `AlertDialog` shadcn/ui avec :
  - Titre + description d'avertissement
  - Champ mot de passe
  - Bouton "Supprimer définitivement" désactivé si le champ est vide ou si la mutation est en cours
  - Bouton "Annuler" pour fermer
- `useMutation` TanStack Query avec :
  - `onSuccess` : `queryClient.clear()` + `logout()` + `navigate({ to: "/" })` + toast succès
  - `onError` : toast erreur (ex: "Le mot de passe est incorrect")
  - `onSettled` : reset du mot de passe + fermeture du dialog

#### `app/frontend/src/features/users/pages/MyProfilePage.tsx`
- Import et ajout de `<DeleteAccountSection />` après le `<PasswordForm />`

## Corrections annexes incluses dans cette branche

### `.gitattributes` (nouveau)
- `* text=auto eol=lf` : force les line endings LF dans Git, quelle que soit la plateforme (Mac/Windows)
- Corrige les erreurs Prettier en CI causées par le mix CRLF/LF

### Normalisation Prettier (199 fichiers)
- Passage de Prettier sur tous les fichiers source backend + frontend
- Conversion CRLF → LF + tabs → spaces
- Garantit que la CI lint passe de manière cohérente

### `Dockerfile.prisma`
- Simplifié pour Prisma 7 : `prisma generate` au runtime (pas au build) car le volume fournit tout
- Corrige l'erreur `Cannot find module 'prisma/config'`

### `invoice.service.ts`
- Suppression des imports inutilisés (`Image`, `path`) qui faisaient échouer le lint en CI

## Branche
`feature/rgpd-account-deletion`

## Fichiers modifiés
- `app/backend/src/auth/dtos/delete-account.dto.ts` (nouveau)
- `app/backend/src/auth/auth.controller.ts`
- `app/backend/src/auth/auth.service.ts`
- `app/backend/src/users/users.service.ts`
- `app/backend/Dockerfile.prisma`
- `app/frontend/src/features/users/api/usersApi.ts`
- `app/frontend/src/features/users/components/DeleteAccountSection.tsx` (nouveau)
- `app/frontend/src/features/users/pages/MyProfilePage.tsx`
- `.gitattributes` (nouveau)
- 199 fichiers normalisés (prettier)
- `app/backend/src/orders/invoice.service.ts` (imports inutilisés)

## Sections du dossier CDA concernées
- §7.2 (composant métier — transaction Prisma d'anonymisation, pattern anonymisation vs cascade)
- §7.3 (accès aux données — transaction atomique multi-tables, `updateMany` + `deleteMany` + `delete`)
- §8.5 (RGPD — droit à l'effacement Art. 17, conciliation avec obligation comptable L123-22)
