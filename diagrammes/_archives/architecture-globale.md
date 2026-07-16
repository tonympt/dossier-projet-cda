# Architecture globale - GreenRoots

```mermaid
graph LR
    subgraph Presentation["Couche Presentation"]
        direction TB
        Nginx["Nginx<br/>(reverse proxy +<br/>fichiers statiques)"]
        SPA["SPA React 19<br/>TanStack Router/Query<br/>Zustand<br/><i>executee dans le navigateur</i>"]
        Nginx -->|"sert"| SPA
    end

    subgraph Metier["Couche Metier"]
        direction TB
        Controllers["Controllers"]
        Guards["Guards<br/>(auth + roles)"]
        DTOs["DTOs<br/>(validation)"]
        Services["Services<br/>(logique metier)"]
        Controllers --> Guards
        Guards --> DTOs
        DTOs --> Services
    end

    subgraph Donnees["Couche Donnees"]
        direction TB
        Prisma["Prisma ORM"]
        PostgreSQL["PostgreSQL<br/>+ PostGIS"]
        Redis["Redis<br/>(cache + sessions)"]
        Prisma --> PostgreSQL
    end

    subgraph Externes["Services externes"]
        direction TB
        Stripe["Stripe<br/>(paiement)"]
        Cloudinary["Cloudinary<br/>(images)"]
        Resend["Resend<br/>(emails)"]
    end

    SPA -->|"HTTP + cookies<br/>(/api)"| Controllers
    Services --> Prisma
    Services --> Redis
    Services --> Stripe
    Services --> Cloudinary
    Services --> Resend
```

> **Architecture 3 tiers** : la couche présentation (Nginx + SPA React) communique avec la couche métier (API REST NestJS) via des requêtes HTTP. La couche métier accède à la couche données (PostgreSQL, Redis) et aux services externes (Stripe, Cloudinary, Resend). Nginx sert les fichiers statiques de la SPA et redirige les appels `/api` vers NestJS.
