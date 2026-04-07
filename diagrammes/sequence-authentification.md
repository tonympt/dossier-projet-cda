# Diagramme de séquence — Authentification

```mermaid
sequenceDiagram
    actor U as Utilisateur
    participant F as Frontend
    participant B as Backend
    participant DB as PostgreSQL
    participant R as Redis

    rect rgb(240, 249, 235)
    note right of U: Login
    U->>F: Email + mot de passe
    F->>B: POST /auth/login
    B->>R: Vérifier brute force (5 tentatives / 15 min)
    B->>DB: Chercher utilisateur par email
    B->>B: bcrypt.compare(mdp)
    B->>R: Effacer compteur tentatives
    B->>B: Générer JWT (24h)
    B-->>F: Set-Cookie: access_token (HttpOnly, Secure, SameSite=Lax)
    end

    rect rgb(252, 228, 236)
    note right of U: Logout
    U->>F: Clic "Déconnexion"
    F->>B: POST /auth/logout (cookie envoyé auto)
    B->>R: Blacklister token (TTL = durée restante JWT)
    B-->>F: Clear-Cookie
    end
```

> **Points clés** : Redis joue un double rôle — protection brute force (compteur INCR avec TTL auto-expirant) et blacklist des tokens révoqués au logout. Le cookie HttpOnly protège contre le vol de token par XSS. Le rôle est vérifié en BDD à chaque requête authentifiée (pas depuis le token).
