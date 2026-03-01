# Feature 04 — RGPD : Export de données personnelles

## Priorité : P2 | Estimation : ~2-3h

## Contexte juridique
- **RGPD Article 15** (droit d'accès) : toute personne a le droit d'obtenir une copie de l'ensemble de ses données personnelles traitées par le responsable du traitement
- **Format** : les données doivent être fournies dans un format structuré, couramment utilisé et lisible par machine (article 20 — portabilité)
- **Choix JSON** : format universel, lisible par machine et par l'humain, facilement importable dans d'autres systèmes

## Données exportées

### Contenu du fichier JSON
Le fichier exporté contient **toutes les données personnelles** de l'utilisateur, structurées en sections :

```json
{
  "exportDate": "2026-03-01T14:30:00.000Z",
  "user": {
    "id": 42,
    "email": "tony@gmail.com",
    "firstName": "Tony",
    "lastName": "Memponteil",
    "avatarUrl": "https://res.cloudinary.com/...",
    "isEmailVerified": true,
    "roleId": 1,
    "createdAt": "2025-11-15T10:00:00.000Z",
    "updatedAt": "2026-02-28T16:45:00.000Z",
    "role": { "name": "user" },
    "orders": [
      {
        "id": 15,
        "orderStatus": "CONFIRMED",
        "paymentStatus": "PAID",
        "totalAmount": 12000,
        "totalExclTax": 10000,
        "totalVatAmount": 2000,
        "currency": "EUR",
        "contactEmail": "tony@gmail.com",
        "invoiceNumber": "GR-20260301-00015",
        "invoiceDate": "2026-03-01T14:00:00.000Z",
        "createdAt": "2026-03-01T13:55:00.000Z",
        "updatedAt": "2026-03-01T14:00:00.000Z",
        "items": [
          {
            "quantity": 2,
            "unitPrice": 5000,
            "vatRate": 2000,
            "tree": {
              "name": "Chêne sessile",
              "scientificName": "Quercus petraea"
            }
          }
        ],
        "addresses": [
          {
            "type": "BILLING",
            "firstName": "Tony",
            "lastName": "Memponteil",
            "street": "12 rue des Lilas",
            "streetLine2": null,
            "postalCode": "69001",
            "city": "Lyon",
            "countryCode": "FR",
            "phone": "06 12 34 56 78"
          }
        ]
      }
    ],
    "cart": {
      "createdAt": "2026-03-01T12:00:00.000Z",
      "updatedAt": "2026-03-01T12:05:00.000Z",
      "items": [
        {
          "quantity": 1,
          "tree": {
            "name": "Érable champêtre",
            "scientificName": "Acer campestre"
          }
        }
      ]
    }
  }
}
```

### Ce qui est INCLUS
| Donnée | Détail |
|---|---|
| Profil utilisateur | email, prénom, nom, avatar, date de création, rôle |
| Commandes | statut, montants (HT/TVA/TTC), numéro de facture, dates |
| Articles de commande | quantité, prix unitaire, TVA, nom de l'arbre |
| Adresses de commande | adresses de facturation et livraison |
| Panier actuel | articles et quantités |

### Ce qui est EXCLU (sécurité)
| Donnée | Raison |
|---|---|
| `password` | Hash bcrypt — donnée technique, pas personnelle |
| `emailVerificationTokenHash` | Token de sécurité interne |
| `emailVerificationTokenExpiry` | Donnée technique interne |
| `passwordResetTokenHash` | Token de sécurité interne |
| `passwordResetTokenExpiry` | Donnée technique interne |

Ces exclusions sont gérées automatiquement par le `select` Prisma (pas de `*` — uniquement les champs explicitement listés).

## Flux utilisateur
1. L'utilisateur va sur `/profile/me`
2. Il voit la section "Exporter mes données" (entre le formulaire de mot de passe et la suppression de compte)
3. Il clique sur "Télécharger mes données"
4. Le bouton passe en état "Export en cours..."
5. Un fichier `greenroots-mes-donnees-2026-03-01.json` est téléchargé automatiquement
6. Toast de confirmation : "Vos données ont été exportées avec succès."

## Modifications techniques

### Backend

#### `app/backend/src/users/users.service.ts`
- Nouvelle méthode `exportUserData(userId)` :
  - Requête Prisma unique avec `select` explicite (pas d'include `*`)
  - Jointures : `orders` (avec `items.tree` et `addresses`), `cart` (avec `items.tree`), `role`
  - Tri commandes par date décroissante
  - Retourne `{ exportDate, user }` ou `null` si user introuvable

#### `app/backend/src/auth/auth.service.ts`
- Nouvelle méthode `exportUserData(userId)` :
  - Appelle `usersService.exportUserData()`
  - Lance `NotFoundException` si utilisateur introuvable
  - Log d'audit : `Export de données RGPD pour l'utilisateur ID: X`

#### `app/backend/src/auth/auth.controller.ts`
- Nouvel endpoint `GET /auth/me/export`
  - Guards : `JwtAuthGuard` (utilisateur connecté obligatoire)
  - Header `Content-Disposition: attachment; filename="greenroots-export-{timestamp}.json"`
  - Header `Content-Type: application/json`
  - Réponse : JSON direct (pas de Serialize interceptor — c'est un fichier)
  - Swagger documenté
  - Pas besoin de mot de passe (contrairement à la suppression) : la consultation de ses propres données ne nécessite que l'authentification

### Frontend

#### `app/frontend/src/features/users/api/usersApi.ts`
- Nouvelle méthode `exportMyData()` → `GET /auth/me/export` avec `responseType: "blob"` pour récupérer le fichier

#### `app/frontend/src/features/users/components/ExportDataSection.tsx` (nouveau)
- Section avec bordure neutre (slate) et icône Download
- Texte explicatif mentionnant l'Article 15 RGPD
- Bouton `variant="outline"` "Télécharger mes données"
- `useMutation` TanStack Query avec :
  - `onSuccess` : création d'un lien temporaire (`URL.createObjectURL`) → click → nettoyage (`revokeObjectURL`) + toast succès
  - `onError` : toast erreur

#### `app/frontend/src/features/users/pages/MyProfilePage.tsx`
- Import et ajout de `<ExportDataSection />` entre `<PasswordForm />` et `<DeleteAccountSection />`

## Différences avec la suppression de compte (feature 03)
| Aspect | Suppression (feature 03) | Export (feature 04) |
|---|---|---|
| Méthode HTTP | DELETE | GET |
| Mot de passe requis | Oui (action destructive) | Non (consultation) |
| Conséquence | Irréversible, suppression + anonymisation | Lecture seule, aucune modification |
| Réponse | Message JSON | Fichier JSON téléchargé |
| UI | AlertDialog avec confirmation | Bouton simple |

## Branche
`feature/rgpd-data-export`

## Fichiers modifiés
- `app/backend/src/users/users.service.ts`
- `app/backend/src/auth/auth.service.ts`
- `app/backend/src/auth/auth.controller.ts`
- `app/frontend/src/features/users/api/usersApi.ts`
- `app/frontend/src/features/users/components/ExportDataSection.tsx` (nouveau)
- `app/frontend/src/features/users/pages/MyProfilePage.tsx`

## Sections du dossier CDA concernées
- §7.2 (composant métier — requête Prisma avec jointures multiples, select sécurisé)
- §8.5 (RGPD — droit d'accès Art. 15, format structuré Art. 20, exclusion des données sensibles)
