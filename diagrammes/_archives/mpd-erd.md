# MPD — Modèle Physique de Données (GreenRoots)

> Niveau **physique** (PostgreSQL). Représentation de ce que définit `schema.prisma`.
> Chaque table affiche **4 colonnes** : `Type` · `Champ` · `Clé` (PK/FK/UK) · `Contraintes`
> (nullabilité, valeur par défaut, référence de la FK, note). Sans mention `NULL`, le champ
> est obligatoire (`NOT NULL`). Types = types PostgreSQL réels (VARCHAR(n), TEXT, INTEGER,
> TIMESTAMP, BOOLEAN, geometry, types énumérés).
>
> ⚠️ Le diagramme ne peut pas tout exprimer (unicités composites, `ON DELETE`, index,
> séquences) → voir la section **« Contraintes physiques complémentaires »** dessous.
> Le MPD **autoritaire et complet** reste le DDL des migrations : `prisma/migrations/*/migration.sql`.

```mermaid
erDiagram

    PaymentStatus {
        enum PENDING
        enum PROCESSING
        enum PAID
        enum FAILED
        enum REFUNDED
    }

    OrderStatus {
        enum PENDING
        enum CONFIRMED
        enum PLANTED
        enum CANCELLED
    }

    AddressType {
        enum BILLING
        enum DELIVERY
    }

    roles {
        INTEGER id PK "auto-increment"
        VARCHAR(100) name UK "NOT NULL"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    users {
        INTEGER id PK "auto-increment"
        VARCHAR(255) email UK "NOT NULL"
        VARCHAR(255) password "NOT NULL, bcrypt"
        VARCHAR(64) first_name "NOT NULL"
        VARCHAR(64) last_name "NOT NULL"
        VARCHAR(500) avatar_url "NULL"
        INTEGER role_id FK "ref roles(id)"
        BOOLEAN is_email_verified "defaut false"
        VARCHAR(64) email_verification_token_hash UK "NULL, SHA-256"
        TIMESTAMP email_verification_token_expiry "NULL"
        VARCHAR(64) password_reset_token_hash UK "NULL, SHA-256"
        TIMESTAMP password_reset_token_expiry "NULL"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    countries {
        INTEGER id PK "auto-increment"
        VARCHAR(5) code UK "NOT NULL, ISO 3166-1"
        VARCHAR(100) name "NOT NULL"
        VARCHAR(3) currency "defaut EUR"
        BOOLEAN is_billing_available "defaut false"
        BOOLEAN is_planting_available "defaut false"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    locations {
        INTEGER id PK "auto-increment"
        VARCHAR(64) name "NOT NULL"
        VARCHAR(64) region "NULL"
        geometry coordinates "PostGIS, NOT NULL"
        TEXT description "NOT NULL"
        VARCHAR(255) image_url "NULL"
        INTEGER country_id FK "ref countries(id)"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    categories {
        INTEGER id PK "auto-increment"
        VARCHAR(64) name UK "NOT NULL"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    trees {
        INTEGER id PK "auto-increment"
        VARCHAR(64) name "NOT NULL"
        VARCHAR(64) scientific_name "NOT NULL"
        TEXT description "NOT NULL"
        VARCHAR(255) image_url "NOT NULL"
        INTEGER price "NOT NULL, centimes"
        INTEGER category_id FK "ref categories(id)"
        INTEGER location_id FK "ref locations(id)"
        INTEGER height_max "NULL"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    stock {
        INTEGER id PK "auto-increment"
        INTEGER tree_id FK,UK "ref trees(id), CASCADE"
        INTEGER quantity "defaut 0"
        INTEGER low_stock_threshold "defaut 10"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    orders {
        INTEGER id PK "auto-increment"
        INTEGER user_id FK "ref users(id)"
        INTEGER total_amount "NOT NULL, centimes"
        VARCHAR(3) currency "defaut EUR"
        INTEGER total_excl_tax "defaut 0"
        INTEGER total_vat_amount "defaut 0"
        TEXT invoice_number UK "NULL"
        TIMESTAMP invoice_date "NULL"
        PaymentStatus payment_status "defaut PENDING"
        OrderStatus order_status "defaut PENDING"
        TEXT payment_intent_id UK "NULL"
        VARCHAR(255) contact_email "NULL"
        TIMESTAMP stock_reserved_at "NULL"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    order_items {
        INTEGER id PK "auto-increment"
        INTEGER order_id FK "ref orders(id)"
        INTEGER tree_id FK "ref trees(id)"
        INTEGER quantity "NOT NULL"
        INTEGER unit_price "NOT NULL, fige achat"
        INTEGER vat_rate "defaut 2000 (20%)"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    order_addresses {
        INTEGER id PK "auto-increment"
        INTEGER order_id FK "ref orders(id), CASCADE"
        AddressType type "NOT NULL"
        VARCHAR(64) first_name "NOT NULL"
        VARCHAR(64) last_name "NOT NULL"
        VARCHAR(255) street "NOT NULL"
        VARCHAR(255) street_line_2 "NULL"
        VARCHAR(20) postal_code "NOT NULL"
        VARCHAR(100) city "NOT NULL"
        VARCHAR(5) country_code "snapshot, sans FK"
        VARCHAR(20) phone "NULL"
        TIMESTAMP created_at "defaut now()"
        TIMESTAMP updated_at "auto"
    }

    users }o--|| roles : "role"
    locations }o--|| countries : "country"
    trees }o--|| categories : "category"
    trees }o--|| locations : "location"
    stock |o--|| trees : "tree"
    orders }o--|| users : "user"
    order_items }o--|| orders : "order"
    order_items }o--|| trees : "tree"
    order_addresses }o--|| orders : "order"
    orders |o--|| PaymentStatus : "payment_status"
    orders |o--|| OrderStatus : "order_status"
    order_addresses |o--|| AddressType : "type"
```

## Contraintes physiques complémentaires (non représentables dans le diagramme)

- **Clés primaires** : `id` en auto-increment (séquence PostgreSQL) sur les 10 tables.
- **Unicités composites** :
  - `trees (name, location_id)` — un même arbre unique par lieu (`unique_tree_per_location`)
  - `order_items (order_id, tree_id)` — une ligne par arbre et par commande
  - `order_addresses (order_id, type)` — au plus une adresse par type et par commande
- **Actions référentielles (`ON DELETE`)** :
  - `stock.tree_id` → `trees(id)` **CASCADE** (le stock disparaît avec l'arbre)
  - `order_addresses.order_id` → `orders(id)` **CASCADE**
  - autres FK : `RESTRICT` (défaut — on ne supprime pas un référentiel utilisé)
- **Types énumérés PostgreSQL** (`CREATE TYPE`) : `payment_status`, `order_status`, `address_type`.
- **Extension** : **PostGIS** requise pour `locations.coordinates` (`geometry`).
- **Index** : implicites sur chaque PK et chaque contrainte `UNIQUE` ; index sur FK selon les besoins de requêtage.
- **Défauts** : `currency = 'EUR'`, `total_excl_tax = 0`, `total_vat_amount = 0`, `quantity = 0`,
  `low_stock_threshold = 10`, `vat_rate = 2000` (20,00 %), booléens `= false`, statuts `= PENDING`.

> **Source de vérité** : les fichiers `prisma/migrations/*/migration.sql` (≈20 migrations) contiennent
> le DDL PostgreSQL réel (CREATE TABLE, types, contraintes, index). Cet ERD en est la synthèse visuelle.
