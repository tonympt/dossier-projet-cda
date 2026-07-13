# MLD - Modèle Logique de Données (GreenRoots)

> Niveau **logique** (relationnel), dérivé du MCD (`mcd-mocodo.txt`) par application des
> règles de passage Merise. Indépendant du SGBD : pas encore de types physiques, d'index,
> ni d'enums PostgreSQL (→ ceux-ci apparaissent au MPD).
>
> **Notation** : clé primaire en **gras** ; clé étrangère préfixée de `#`.
> Les attributs techniques (tokens de sécurité, horodatage, `paymentIntentId`,
> `stockReservedAt`) sont réintroduits au MPD, pas ici — on garde le périmètre du MCD
> pour que la transformation MCD→MLD ne montre QUE du structurel (FK, tables, contraintes).

## Les 9 relations

- **Role** ( **id**, name )
- **User** ( **id**, email, password, firstName, lastName, avatarUrl, isEmailVerified, #roleId )
- **Country** ( **id**, code, name, currency, isBillingAvailable, isPlantingAvailable )
- **Location** ( **id**, name, region, coordinates, description, imageUrl, #countryId )
- **Category** ( **id**, name )
- **Tree** ( **id**, name, scientificName, description, imageUrl, price, heightMax, stockQuantity, lowStockThreshold, #categoryId, #locationId )
- **Order** ( **id**, totalAmount, totalExclTax, totalVatAmount, currency, contactEmail, invoiceNumber, invoiceDate, paymentStatus, orderStatus, #userId )
- **OrderItem** ( **id**, #orderId, #treeId, quantity, unitPrice, vatRate ) — `UNIQUE(orderId, treeId)`
- **OrderAddress** ( **id**, #orderId, type, firstName, lastName, street, streetLine2, postalCode, city, countryCode, phone ) — `UNIQUE(orderId, type)`

> Note : `stockQuantity` / `lowStockThreshold` restent dans `Tree` au MLD. Le split en table
> `TreeStock` (partition verticale) est une **optimisation physique** → introduite au MPD (cf. point 4).

## Règles de passage MCD → MLD appliquées

| # | Règle | Application dans GreenRoots |
|---|-------|-----------------------------|
| R1 | Chaque **entité** devient une **relation (table)** | Role, User, Country, Location, Category, Tree, Order, OrderAddress |
| R2 | L'**identifiant** de l'entité devient la **clé primaire** | `id` de chaque table (clé de substitution, cf. ci-dessous) |
| R3 | Association **1:n** → la **PK du côté (x,1) descend en FK** côté (x,n) | `HAS` → `#roleId` dans User ; `PLACES` → `#userId` dans Order ; `BELONGS_TO` → `#categoryId` dans Tree ; `PLANTED_AT` → `#locationId` dans Tree ; `LOCATED_IN` → `#countryId` dans Location ; `HAS_ADDRESS` → `#orderId` dans OrderAddress |
| R4 | Association **n:n porteuse** → **nouvelle table** avec les 2 FK + les attributs portés | `CONTAINS` → table **OrderItem** (`#orderId`, `#treeId`, quantity, unitPrice, vatRate) |
| R5 | Les **cardinalités max = 1** répétées deviennent des **contraintes d'unicité** | `UNIQUE(orderId, type)` sur OrderAddress (au plus 1 adresse par type) ; `UNIQUE(orderId, treeId)` sur OrderItem (une ligne par arbre et par commande) |

## Points de conception notables (vocabulaire jury)

1. **Clés de substitution (surrogate keys)** — Toutes les tables ont un `id` auto-incrémenté
   comme PK, y compris `OrderItem` (PK = `id` + `UNIQUE(orderId, treeId)`).
   *Dérivation « pure » Merise :* `OrderItem` aurait une **PK composite** (`orderId`, `treeId`).
   *Choix retenu :* clé de substitution → FK plus simples à référencer, convention ORM (Prisma),
   stabilité si le couple métier évolue. La contrainte d'unicité garantit quand même l'unicité métier.

2. **`OrderAddress.countryCode` n'est PAS une clé étrangère** (pas de `#`) — c'est un attribut
   texte figé (**snapshot** de l'adresse au moment de la commande). À comparer avec
   `Location.countryId` qui, lui, EST une vraie FK vers un référentiel vivant (`Country`).
   → Asymétrie **assumée** : on normalise le référentiel, on dénormalise l'historique.

3. **`type` (OrderAddress) et `paymentStatus`/`orderStatus` (Order)** restent des attributs
   « logiques » ici ; ils deviendront des **types énumérés PostgreSQL** au MPD.

4. **Split `TreeStock` reporté au MPD** — au MLD, le stock reste dans `Tree` (`stockQuantity`,
   `lowStockThreshold`). La séparation en relation `TreeStock` liée en 1:1 est une
   **partition verticale**, une **optimisation physique** motivée par des cycles de vie d'accès
   différents (stock écrit souvent / catalogue lu souvent) et la concurrence (verrou sur la ligne
   de stock). Ces raisons étant **physiques**, sa place est au **MPD**, pas au MLD — qui demeure
   une dérivation logique pure. *(Choix de périmètre : justif physique au niveau physique.)*

5. **Dénormalisation volontaire — historisation vs redondance** — deux cas à ne pas confondre :
   - `OrderItem.unitPrice` / `vatRate` : **nécessité d'historisation**, PAS de la redondance.
     `Tree.price` évolue dans le temps ; il faut **figer** le prix payé au moment de l'achat,
     sinon un recalcul ultérieur donnerait le prix courant, pas le prix facturé.
   - `Order.totalAmount` / `totalExclTax` / `totalVatAmount` : **vraie dénormalisation** — ce sont
     des **sommes recalculables** depuis les lignes déjà figées (`totalAmount = totalExclTax +
     totalVatAmount`). On les stocke **délibérément** (pas « arbitrairement ») pour : (a) l'intégrité
     de la **facture** (document légal figé à l'émission) ; (b) la **performance** (éviter un `SUM`
     à chaque lecture). Même logique de **snapshot** que `countryCode` (point 2).
   - À revendiquer devant le jury : une dénormalisation **consciente et argumentée** est valorisée ;
     une redondance **subie** est sanctionnée.
