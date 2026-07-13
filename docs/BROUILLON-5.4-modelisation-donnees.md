# BROUILLON — Refonte §5.4 « Modélisation des données »

> Brouillon de travail à recopier dans `Dossier Projet GreenRoots.docx` (demain, autre device).
> Contient : (A) le texte de la 5.4, (B) les scripts sauvegarde/restauration, (C) la checklist
> de mise en forme du .docx, (D) diagramme → outil, (E) les TODO ultérieurs.
> ⚠️ La mémoire Claude est locale à la machine d'hier : ce fichier est le vrai vecteur de transfert.

---

## (A) Texte de la section 5.4 (à coller)

### 5.4 Modélisation des données

La base de données PostgreSQL a été conçue selon la **méthode Merise**, en trois niveaux d'abstraction successifs dès le sprint 0 : modèle conceptuel (MCD), logique (MLD) puis physique (MPD). Elle s'organise autour de **10 entités** réparties en trois domaines fonctionnels : catalogue, commande et utilisateur.

**• Modèle conceptuel des données (MCD)**
*[Insérer le diagramme MCD — version agrandie en Annexe X]*
Le MCD décrit le métier indépendamment de toute technologie :
- **Catalogue** : un `Arbre` appartient à une `Catégorie` (0,n–1,1) et est planté dans un `Lieu` (0,n–1,1) rattaché à un `Pays` (0,n–1,1).
- **Commande** : un `Utilisateur` passe des `Commandes` (0,n–1,1) ; une commande *contient* des arbres via une **association porteuse** (quantité, prix unitaire, TVA) et *possède* une à deux adresses typées (facturation, livraison).
- **Utilisateur** : un `Utilisateur` détient un `Rôle` (0,n–1,1). Le panier est géré côté client (Zustand) ; sa persistance serveur est une évolution envisagée.

**• Modèle logique des données (MLD)**
Le passage du MCD au modèle relationnel applique trois règles systématiques : (1) chaque entité devient une table dont l'identifiant est la clé primaire ; (2) chaque association *un-à-plusieurs* fait « descendre » une clé étrangère du côté « plusieurs » ; (3) chaque association *plusieurs-à-plusieurs* porteuse de propriétés devient une table de liaison. Appliquées à GreenRoots, elles produisent les **9 relations** suivantes *(clé primaire en gras, `#` = clé étrangère)* :

> **Role** (**id**, name)
> **User** (**id**, email, password, firstName, lastName, avatarUrl, isEmailVerified, #roleId)
> **Country** (**id**, code, name, currency, isBillingAvailable, isPlantingAvailable)
> **Location** (**id**, name, region, coordinates, description, imageUrl, #countryId)
> **Category** (**id**, name)
> **Tree** (**id**, name, scientificName, description, imageUrl, price, heightMax, stockQuantity, lowStockThreshold, #categoryId, #locationId)
> **Order** (**id**, totalAmount, totalExclTax, totalVatAmount, currency, contactEmail, invoiceNumber, invoiceDate, paymentStatus, orderStatus, #userId)
> **OrderItem** (**id**, #orderId, #treeId, quantity, unitPrice, vatRate) — *UNIQUE(orderId, treeId)*
> **OrderAddress** (**id**, #orderId, type, firstName, lastName, street, streetLine2, postalCode, city, countryCode, phone) — *UNIQUE(orderId, type)*

L'association porteuse *contenir* devient ainsi la table **`OrderItem`** (règle 3), qui conserve `unitPrice` et `vatRate` — figés à l'achat. Deux choix de conception se lisent directement dans ce modèle :
- **Snapshot** : `OrderAddress.countryCode` n'est **volontairement pas** une clé étrangère vers `Country` ; la commande devient un instantané historique, insensible aux évolutions ultérieures du référentiel.
- **Dénormalisation assumée** : les montants agrégés de `Order` (`totalAmount`, `totalExclTax`, `totalVatAmount`), bien que recalculables depuis les lignes, sont stockés pour l'intégrité de la facture (document légal figé) et la performance d'affichage.

**• Modèle physique des données (MPD)**
*[Insérer une vue d'ensemble de l'ERD — schéma complet en Annexe X]*
Le MPD est matérialisé par le **schéma Prisma** et ses **24 migrations** versionnées (historique complet depuis le sprint 0). Il ajoute la dimension physique PostgreSQL : types réels (`VARCHAR(n)`, `TEXT`, `INTEGER`, `TIMESTAMP`, `geometry`), types énumérés, et une **partition verticale** — le stock est isolé dans la table `stock` (relation 1:1 avec `trees`), séparant l'identité de l'arbre (lue souvent) de sa disponibilité commerciale (écrite souvent, soumise à concurrence).

**• Choix de modélisation notables**
- **Prix en centimes (INT)** : montants stockés en entiers plutôt qu'en décimal — élimine les erreurs d'arrondi flottant et s'aligne sur le standard Stripe. Conversion à l'affichage via un décorateur `@PriceToDecimal()`.
- **Types énumérés** : `PaymentStatus`, `OrderStatus`, `AddressType` contraignent les valeurs en base ; les transitions autorisées sont pilotées par une state machine applicative (cf. §7).
- **PostGIS** : le champ `coordinates` (`geometry`) de `Location` stocke des coordonnées natives, en vue de la carte interactive prévue au CDC.
- **Nommage** : modèles Prisma en PascalCase, tables et colonnes en snake_case via `@map`/`@@map`.

**• Contraintes d'intégrité**
- **Clés étrangères** : toutes les relations sont contraintes, avec `ON DELETE CASCADE` sur `stock` et `order_addresses` (suppression liée à l'entité parente), `RESTRICT` ailleurs.
- **Unicité** : email utilisateur, couple arbre/lieu (`unique_tree_per_location`), couple commande/arbre dans les lignes, couple commande/type d'adresse.
- **Défauts et obligations** : valeurs par défaut (devise `EUR`, TVA 2000 = 20 %, seuil de stock 10), champs obligatoires vs optionnels explicités.

**• Sécurité et confidentialité des données**
Au niveau du modèle : les mots de passe sont stockés **hashés (bcrypt)**, les tokens de vérification/réinitialisation sous forme de **hash SHA-256**, et un helper `safeSelect` exclut systématiquement les champs sensibles des réponses API. Le détail des mesures de sécurité et de la conformité RGPD est présenté en **§8**.

**• Gestion des droits d'accès**
Les droits sont gérés au **niveau applicatif** via les guards NestJS (JWT + rôles USER/ADMIN/SUPERADMIN) plutôt que par des rôles PostgreSQL. Ce choix découle de l'architecture : Prisma utilise une connexion unique à la base, et une gestion des permissions à la couche API est plus fine, plus testable, et constitue le pattern standard des API REST modernes.

**• Jeu d'essai, sauvegarde et migrations**
Un **seed Prisma** fournit un jeu de données de test couvrant les scénarios clés (utilisateurs aux rôles variés, arbres à stocks différents, commandes dans chaque statut). La **sauvegarde/restauration** repose sur une procédure `pg_dump`/`pg_restore` (scripts `backup.sh`/`restore.sh`), **testée et validée en local** : restauration complète d'une sauvegarde dans une base vierge, sans erreur, avec l'intégralité des tables et des données. Son **automatisation planifiée** (sauvegardes managées Coolify vers stockage externe) est prévue lors de la finalisation du déploiement.

---

## (B) Scripts sauvegarde / restauration

> À placer dans `scripts/` du sous-module `projet-greenroots` (via branche → PR `develop`).
> Valeurs réelles confirmées au test : user `root`, base `greenroot` (surchargées par `.env`).
> `pg_dump` embarque le PostGIS sans souci. **Procédure testée le 13/07/2026** : restore complet
> dans une base vierge, 14 tables + données restaurées, exit 0, zéro erreur.

`scripts/backup.sh`
```bash
#!/usr/bin/env bash
# Sauvegarde PostgreSQL GreenRoots (format compressé pg_dump)
set -euo pipefail
SERVICE="${DB_SERVICE:-database}"; DB_USER="${POSTGRES_USER:-root}"; DB_NAME="${POSTGRES_DB:-greenroot}"
OUT="${BACKUP_DIR:-./backups}"; mkdir -p "$OUT"
FILE="$OUT/greenroots_$(date +%Y%m%d_%H%M%S).dump"
docker compose exec -T "$SERVICE" pg_dump -U "$DB_USER" -Fc "$DB_NAME" > "$FILE"
echo "✔ Sauvegarde : $FILE"
```

`scripts/restore.sh`
```bash
#!/usr/bin/env bash
# Restauration PostgreSQL GreenRoots
set -euo pipefail
SERVICE="${DB_SERVICE:-database}"; DB_USER="${POSTGRES_USER:-root}"; DB_NAME="${POSTGRES_DB:-greenroot}"
FILE="${1:?Usage: ./restore.sh <fichier.dump>}"
docker compose exec -T "$SERVICE" pg_restore -U "$DB_USER" -d "$DB_NAME" --clean --if-exists < "$FILE"
echo "✔ Restauration depuis : $FILE"
```

**Automatisation (évolution)** : préférer les **backups planifiés natifs de Coolify** (rétention + envoi S3 + restore intégrés ; même `pg_dump` sous le capot) plutôt qu'un cron maison — une fois le déploiement finalisé.

---

## (C) Checklist mise en forme du .docx (demain)

1. [ ] Remplacer le contenu de **§5.4** par le texte (A) ; renommer le titre « Schéma entités-relations » → « **Modélisation des données** ».
2. [ ] Insérer les diagrammes : **MCD** (corps) · **MLD** = bloc texte des 9 relations (corps) · **MPD** = vue d'ensemble (corps) + **ERD complet en Annexe**.
3. [ ] Corriger **~ligne 275** : « Application MVP fonctionnelle et déployée » → « déployable, déploiement à finaliser » (aligner sur la formulation honnête de la ~l.827 « quasi production-ready »).
4. [ ] Corriger les chiffres : **10 entités** (pas 12/13), **24 migrations** (pas 23).
5. [ ] **Retirer toute mention Cart/CartItem** (supprimés du schéma, PR #4).
6. [ ] **RGPD** : reste en **§8.5** — ne PAS dupliquer en 5.4 (5.4 garde seulement la phrase confidentialité niveau-donnée + renvoi §8).

---

## (D) Diagramme → outil de rendu

| Fichier (dans `diagrammes/`) | Niveau | Outil de rendu |
|---|---|---|
| `mcd-mocodo.mcd` | MCD | coller dans [mocodo.net](https://mocodo.net) |
| `mld-source-mocodo.mcd` | MLD (diagramme, optionnel) | Mocodo `--transform mld` |
| `mld-modele-logique.md` | MLD (texte + annotations) | tel quel |
| `mpd-erd.md` | MPD | [mermaid.live](https://mermaid.live) / preview Mermaid |
| `mcd-evolution-rbac-mocodo.mcd` | MCD évolution RBAC (annexe optionnelle) | mocodo.net |
| `mocodo-out/` | sorties générées (svg, mld.md, ddl.sql) | déjà rendues |

---

## (E) TODO ultérieurs

- [x] ~~Tester `restore.sh` sur Docker local~~ **FAIT le 13/07/2026** (restore validé, 14 tables, exit 0) → « testée et validée » désormais autorisé dans le dossier.
- [ ] Ajouter `scripts/backup.sh` + `restore.sh` au sous-module `projet-greenroots` (branche → PR `develop`) — défauts corrigés : user `root`, base `greenroot`.
- [ ] Lancer `/maj-docs` : le `docs/PLAN-DOSSIER.md` est périmé (parle de « 13 entités », Cart/CartItem, et n'inclut pas le MLD).
- [ ] Balayer la cohérence « déployée » dans les autres sections (§6 infra, §9…) — l'app n'est pas déployée, CI prod cassée.
- [ ] Décider du sort du 2ᵉ MCD (évolution RBAC) : annexe optionnelle ou support oral uniquement.
