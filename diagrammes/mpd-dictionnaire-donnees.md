# MPD — Dictionnaire de données (GreenRoots)

> **Modèle Physique de Données** présenté en **dictionnaire de données** : une table par relation,
> colonnes **Champ / Type / Contraintes**. Vérifié contre le DDL réel des **24 migrations**
> (`prisma/migrations/*/migration.sql`), source de vérité.
> Complémentaire de l'ERD (`mpd-erd.md`), qui donne la vue d'ensemble des relations.
>
> **Conventions**
> - Types = types PostgreSQL réels. Les dates sont des `TIMESTAMP(3)` (précision milliseconde).
> - `PK` = clé primaire (`SERIAL` : `INTEGER` auto-incrémenté par séquence) · `FK →` = clé étrangère ·
>   `UNIQUE` = unicité · `NULL` = facultatif · défaut indiqué le cas échéant. Sans `NULL`, le champ est obligatoire.
> - **Colonnes d'audit communes** (non répétées) : chaque table possède `created_at`
>   (`TIMESTAMP(3)`, défaut `CURRENT_TIMESTAMP`) et `updated_at` (`TIMESTAMP(3)`, mise à jour auto).
> - **Toutes les clés étrangères sont `ON UPDATE CASCADE`** ; le comportement `ON DELETE` est précisé par FK.

---

## Types énumérés (`CREATE TYPE`)

| Type | Valeurs |
|---|---|
| `PaymentStatus` | `PENDING`, `PROCESSING`, `PAID`, `FAILED`, `REFUNDED` |
| `OrderStatus` | `PENDING`, `CONFIRMED`, `PLANTED`, `CANCELLED` |
| `AddressType` | `BILLING`, `DELIVERY` |

---

## Domaine Utilisateur

### `roles`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| name | VARCHAR(100) | UNIQUE |

### `users`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| email | VARCHAR(255) | UNIQUE |
| password | VARCHAR(255) | hash bcrypt |
| first_name | VARCHAR(64) | |
| last_name | VARCHAR(64) | |
| avatar_url | VARCHAR(500) | NULL |
| role_id | INTEGER | FK → roles(id), ON DELETE RESTRICT |
| is_email_verified | BOOLEAN | défaut `false` |
| email_verification_token_hash | VARCHAR(64) | UNIQUE, NULL (hash SHA-256) |
| email_verification_token_expiry | TIMESTAMP(3) | NULL |
| password_reset_token_hash | VARCHAR(64) | UNIQUE, NULL (hash SHA-256) |
| password_reset_token_expiry | TIMESTAMP(3) | NULL |

---

## Domaine Catalogue

### `countries`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| code | VARCHAR(5) | UNIQUE (ISO 3166-1 alpha-2) |
| name | VARCHAR(100) | |
| currency | VARCHAR(3) | défaut `EUR` (ISO 4217) |
| is_billing_available | BOOLEAN | défaut `false` |
| is_planting_available | BOOLEAN | défaut `false` |

### `locations`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| name | VARCHAR(64) | |
| region | VARCHAR(64) | NULL |
| coordinates | geometry | PostGIS |
| description | TEXT | |
| image_url | VARCHAR(255) | NULL |
| country_id | INTEGER | FK → countries(id), ON DELETE RESTRICT |

### `categories`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| name | VARCHAR(64) | UNIQUE |

### `trees`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| name | VARCHAR(64) | |
| scientific_name | VARCHAR(64) | |
| description | TEXT | |
| image_url | VARCHAR(255) | |
| price | INTEGER | centimes |
| category_id | INTEGER | FK → categories(id), ON DELETE RESTRICT |
| location_id | INTEGER | FK → locations(id), ON DELETE RESTRICT |
| height_max | INTEGER | NULL |

> **Contrainte de table** : `UNIQUE (name, location_id)` — un même arbre unique par lieu (`unique_tree_per_location`).

### `stock`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| tree_id | INTEGER | FK → trees(id), **ON DELETE CASCADE**, UNIQUE |
| quantity | INTEGER | défaut `0` |
| low_stock_threshold | INTEGER | défaut `10` |

---

## Domaine Commande

### `orders`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| user_id | INTEGER | FK → users(id), ON DELETE RESTRICT |
| total_amount | INTEGER | centimes |
| currency | VARCHAR(3) | défaut `EUR` |
| total_excl_tax | INTEGER | défaut `0` |
| total_vat_amount | INTEGER | défaut `0` |
| invoice_number | TEXT | UNIQUE, NULL |
| invoice_date | TIMESTAMP(3) | NULL |
| payment_status | PaymentStatus | défaut `PENDING` |
| order_status | OrderStatus | défaut `PENDING` |
| payment_intent_id | TEXT | UNIQUE, NULL |
| contact_email | VARCHAR(255) | NULL |
| stock_reserved_at | TIMESTAMP(3) | NULL |

### `order_items`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| order_id | INTEGER | FK → orders(id), ON DELETE RESTRICT |
| tree_id | INTEGER | FK → trees(id), ON DELETE RESTRICT |
| quantity | INTEGER | |
| unit_price | INTEGER | centimes, figé à l'achat |
| vat_rate | INTEGER | défaut `2000` (= 20,00 %) |

> **Contrainte de table** : `UNIQUE (order_id, tree_id)` — une ligne par arbre et par commande.

### `order_addresses`

| Champ | Type | Contraintes |
|---|---|---|
| id | INTEGER | PK, auto-increment |
| order_id | INTEGER | FK → orders(id), **ON DELETE CASCADE** |
| type | AddressType | |
| first_name | VARCHAR(64) | |
| last_name | VARCHAR(64) | |
| street | VARCHAR(255) | |
| street_line_2 | VARCHAR(255) | NULL |
| postal_code | VARCHAR(20) | |
| city | VARCHAR(100) | |
| country_code | VARCHAR(5) | **sans FK** (snapshot historique) |
| phone | VARCHAR(20) | NULL |

> **Contrainte de table** : `UNIQUE (order_id, type)` — au plus une adresse par type et par commande.

---

## Récapitulatif des contraintes transversales

- **Clés primaires** : `id` de type `SERIAL` (INTEGER + séquence auto-increment) sur les 10 tables.
- **Actions référentielles** : toutes les FK sont `ON UPDATE CASCADE`. Pour `ON DELETE` :
  **CASCADE** sur `stock.tree_id` et `order_addresses.order_id` (données filles supprimées avec le parent) ;
  **RESTRICT** (défaut) sur les autres — on ne supprime pas un référentiel utilisé.
- **Unicités composites** : `trees (name, location_id)`, `order_items (order_id, tree_id)`, `order_addresses (order_id, type)`.
- **Index** : implicites sur chaque PK et chaque contrainte `UNIQUE` ; index sur FK selon les besoins de requêtage.
- **Extension** : **PostGIS** requise pour `locations.coordinates` (`geometry`).
- **Choix notables** : montants en **centimes** (`INTEGER`, standard Stripe, zéro erreur d'arrondi) ;
  `country_code` **dénormalisé** dans `order_addresses` (adresse figée) alors que `locations.country_id`
  est une vraie FK (référentiel vivant).

> **Source de vérité** : le DDL des 24 fichiers `prisma/migrations/*/migration.sql`. Ce dictionnaire en est la synthèse.
