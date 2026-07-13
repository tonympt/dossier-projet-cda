# MPD — Modèle Physique de Données (GreenRoots)

> Niveau **physique** (PostgreSQL). Représentation de ce que définit `schema.prisma`.
> **Colonne clé** : `PK` (primaire), `FK` (étrangère), `UK` (unicité). Commentaires : type
> nullable / valeur par défaut. Types = types PostgreSQL réels (VARCHAR(n), TEXT, INTEGER,
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
        VARCHAR(100) name UK
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    users {
        INTEGER id PK "auto-increment"
        VARCHAR(255) email UK
        VARCHAR(255) password
        VARCHAR(64) first_name
        VARCHAR(64) last_name
        VARCHAR(500) avatar_url "nullable"
        INTEGER role_id FK
        BOOLEAN is_email_verified "default false"
        VARCHAR(64) email_verification_token_hash UK "nullable"
        TIMESTAMP email_verification_token_expiry "nullable"
        VARCHAR(64) password_reset_token_hash UK "nullable"
        TIMESTAMP password_reset_token_expiry "nullable"
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    locations {
        INTEGER id PK "auto-increment"
        VARCHAR(64) name
        VARCHAR(64) region "nullable"
        geometry coordinates "PostGIS"
        TEXT description
        VARCHAR(255) image_url "nullable"
        INTEGER country_id FK
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    categories {
        INTEGER id PK "auto-increment"
        VARCHAR(64) name UK
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    trees {
        INTEGER id PK "auto-increment"
        VARCHAR(64) name
        VARCHAR(64) scientific_name
        TEXT description
        VARCHAR(255) image_url
        INTEGER price
        INTEGER category_id FK
        INTEGER location_id FK
        INTEGER height_max "nullable"
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    stock {
        INTEGER id PK "auto-increment"
        INTEGER tree_id FK,UK
        INTEGER quantity "default 0"
        INTEGER low_stock_threshold "default 10"
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    orders {
        INTEGER id PK "auto-increment"
        INTEGER user_id FK
        INTEGER total_amount
        VARCHAR(3) currency "default EUR"
        INTEGER total_excl_tax "default 0"
        INTEGER total_vat_amount "default 0"
        TEXT invoice_number UK "nullable"
        TIMESTAMP invoice_date "nullable"
        PaymentStatus payment_status "default PENDING"
        OrderStatus order_status "default PENDING"
        TEXT payment_intent_id UK "nullable"
        VARCHAR(255) contact_email "nullable"
        TIMESTAMP stock_reserved_at "nullable"
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    order_items {
        INTEGER id PK "auto-increment"
        INTEGER order_id FK
        INTEGER tree_id FK
        INTEGER quantity
        INTEGER unit_price
        INTEGER vat_rate "default 2000"
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    countries {
        INTEGER id PK "auto-increment"
        VARCHAR(5) code UK
        VARCHAR(100) name
        VARCHAR(3) currency "default EUR"
        BOOLEAN is_billing_available "default false"
        BOOLEAN is_planting_available "default false"
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    order_addresses {
        INTEGER id PK "auto-increment"
        INTEGER order_id FK
        AddressType type
        VARCHAR(64) first_name
        VARCHAR(64) last_name
        VARCHAR(255) street
        VARCHAR(255) street_line_2 "nullable"
        VARCHAR(20) postal_code
        VARCHAR(100) city
        VARCHAR(5) country_code
        VARCHAR(20) phone "nullable"
        TIMESTAMP created_at
        TIMESTAMP updated_at
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
