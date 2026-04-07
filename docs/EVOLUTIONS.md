# Evolutions potentielles identifiées

> Améliorations repérées pendant l'entraînement oral ou la relecture du code. Pas bloquantes, mais utiles à mentionner au jury comme preuve de recul critique.

## Code backend

### `@IsEnum(OrderStatus)` dans UpdateOrderDto
- **Actuellement** : `@IsIn(['PENDING', 'CONFIRMED', 'PLANTED', 'CANCELLED'])` avec les valeurs en dur
- **Problème** : duplication avec `ORDER_STATUS_TRANSITIONS` — risque de désynchronisation si on ajoute un statut
- **Evolution** : utiliser `@IsEnum(OrderStatus)` qui se base directement sur l'enum Prisma

## Infrastructure

### Ajout de Redis en production
- **Actuellement** : Redis absent de `compose.prod.yml` — le backend tourne sans (degradation gracieuse)
- **Consequence** : pas de cache API (categories/countries vont directement en PostgreSQL), pas de protection brute force login (fail-open), pas de blacklist JWT au logout
- **Evolution** : ajouter Redis en prod. Concrètement :
  1. Ajouter un service `redis` dans `compose.prod.yml` (image `redis:alpine`, `expose: "6379"`, healthcheck `redis-cli ping`)
  2. Ajouter `REDIS_URL=redis://redis:6379` dans les variables d'environnement du backend
  3. Le code backend est deja pret — `RedisService` detecte automatiquement la connexion via le flag `isAvailable()`
- **Priorite** : haute — la blacklist JWT et le brute force sont des features de securite qui devraient etre actives en production

## Code frontend

### Separation des responsabilites — AdminOrderDetailPage
- **Actuellement** : le composant gere a la fois l'affichage, la logique de mutation, les dialogs de confirmation, le changement de statut
- **Evolution** : extraire la logique metier dans des hooks dedies (useOrderStatusUpdate, useOrderCancel) et les sous-composants (OrderStatusDialog, OrderActions)
