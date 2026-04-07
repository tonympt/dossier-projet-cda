# Audit de Sécurité Backend - OWASP Top 10:2025

**Date:** 2026-01-29
**Version:** 1.0
**Auditeur:** Claude Code
**Scope:** Application NestJS/TypeScript (app/backend)
**Référentiel:** OWASP Top 10:2025

---

## Résumé Exécutif

| Catégorie OWASP | Score | État |
|-----------------|-------|------|
| A01 - Broken Access Control | 6/10 | 🟡 |
| A02 - Security Misconfiguration | 5/10 | 🔴 |
| A03 - Software Supply Chain | 7/10 | 🟡 |
| A04 - Cryptographic Failures | 6/10 | 🔴 |
| A05 - Injection | 8/10 | 🟡 |
| A06 - Insecure Design | 6/10 | 🔴 |
| A07 - Authentication Failures | 6/10 | 🔴 |
| A08 - Software/Data Integrity | 5/10 | 🔴 |
| A09 - Logging & Alerting | 4/10 | 🔴 |
| A10 - Error Handling | 7/10 | 🟡 |

**Total Failles:** 15 critiques, 8 moyennes, 12 améliorations recommandées

---

## A01:2025 - Broken Access Control

### ✅ Points Conformes

**JWT Authentication Guard**
- Fichier: `app/backend/src/auth/guards/jwt-auth.guard.ts`
- Extraction Bearer token correcte
- Expiration vérifiée (`ignoreExpiration: false`)

**RBAC implémenté**
- Fichier: `app/backend/src/auth/guards/roles.guard.ts`
- Décorateurs `@Roles()` appliqués sur routes sensibles
- SUPER_ADMIN a accès complet

**Vérification de propriété des commandes**
- Fichier: `app/backend/src/orders/orders.controller.ts`
```typescript
@Get(":id")
findOne(@CurrentUser() user, @Param("id") id: number) {
  if (user.role === "ADMIN" || user.role === "SUPERADMIN") {
    return this.ordersService.findOneAdmin(id);
  }
  return this.ordersService.findOne(id, user.userId); // ✅ Vérifie propriété
}
```

**Protection Checkout**
- Fichier: `app/backend/src/checkout/checkout.service.ts:141-145`
```typescript
if (order.userId !== userId) {
  throw new ForbiddenException("Vous n'êtes pas autorisé...");
}
```

---

### 🔴 Failles Critiques

**1. CORS ouvert sans restriction**
- Fichier: `app/backend/src/main.ts:18`
```typescript
app.enableCors(); // ❌ Accepte TOUTES les origines
```

**Recommandation:**
```typescript
app.enableCors({
  origin: process.env.FRONTEND_URL,
  credentials: true,
  methods: ['GET', 'POST', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
});
```

OK, fait.

**2. Pas de Helmet (headers de sécurité)**
- ❌ Aucun X-Frame-Options, X-Content-Type-Options, HSTS
- Helmet non installé dans package.json

**Recommandation:**
```bash
npm install helmet
```
```typescript
import helmet from 'helmet';
app.use(helmet());
```
Ok, fait.

 L'intérêt de Helmet en une phrase : il dit au navigateur du client comment se protéger contre les attaques les plus classiques (XSS, clickjacking…) en ajoutant des headers de sécurité dans chaque réponse HTTP, sans que vous ayez à   
  coder quoi que ce soit de plus.

**3. Swagger exposé en production**
- Fichier: `app/backend/src/main.ts:37-57`
```typescript
SwaggerModule.setup('docs', app, document); // ❌ Toujours actif
```

**Recommandation:**
```typescript
if (process.env.NODE_ENV !== 'production') {
  SwaggerModule.setup('docs', app, document);
}
```
OK, fait.

**4. JWT Secret sans validation de force**
- Fichier: `app/backend/src/app.module.ts`
```typescript
JWT_SECRET: Joi.string().required(), // ❌ Pas de minLength
```

**Recommandation:**
```typescript
JWT_SECRET: Joi.string().min(32).required(),
```
Une clé JWT courte peut être trouvée par brute-force : un attaquant intercepte un token, puis teste des clés courtes pour en trouver une qui produit la même signature. 32 caractères garantit une entropie minimale qui rend cette      
  attaque computationnellement infaisable.

---

### 🟡 Améliorations Recommandées

- Ajouter audit trail pour actions admin
- Implémenter refresh tokens
- Ajouter blacklist JWT après logout

---

## A02:2025 - Security Misconfiguration

### ✅ Points Conformes

**Validation des variables d'environnement**
- Fichier: `app/backend/src/app.module.ts`
```typescript
ConfigModule.forRoot({
  validationSchema: Joi.object({
    DATABASE_URL: Joi.string().required(),
    JWT_SECRET: Joi.string().required(),
    // ...
  }),
}),
```

**Rate Limiting global**
- Fichier: `app/backend/src/app.module.ts:40-48`
```typescript
ThrottlerModule.forRoot({
  throttlers: [{ ttl: 60000, limit: 60 }], // 60 req/min
}),
```

**Rate Limiting renforcé sur Auth**
- Fichier: `app/backend/src/auth/auth.controller.ts`
```typescript
@Throttle({ default: { limit: 5, ttl: 60000 } }) // 5 req/min sur login
async login() { ... }
```

---

### 🔴 Failles Critiques

**1. Logs console.log en production**
- Fichier: `app/backend/src/database/prisma.service.ts`
```typescript
console.log("✅ Database connected"); // ❌ Visible en prod
```
- Fichier: `app/backend/src/email/email.service.ts:37-43`
```typescript
console.error('Erreur Resend:', error); // ❌ Stack traces exposés
```

**Recommandation:** Remplacer par `Logger` NestJS

OK, fait.


**2. Raw body activé globalement**
- Fichier: `app/backend/src/main.ts:14`
```typescript
rawBody: true, // ❌ Actif pour tous les endpoints
```

**Recommandation:** Middleware ciblé pour `/webhook` uniquement

---

## A03:2025 - Software Supply Chain Failures

### ✅ Points Conformes

- `package-lock.json` présent
- Dépendances récentes et maintenues (@nestjs 11, bcrypt 6, prisma 7)
- Pas de scripts npm suspects

---

### 🔴 Failles Critiques

**1. Pas d'audit de sécurité automatisé**
```json
// Ajouter dans package.json
"scripts": {
  "audit:security": "npm audit --audit-level=moderate"
}
```

**2. Versions non épinglées**
```json
"stripe": "^20.0.0" // ❌ Accepte minor changes
```

---

## A04:2025 - Cryptographic Failures

### ✅ Points Conformes

**Bcrypt pour hashage**
- Fichier: `app/backend/src/auth/auth.service.ts`
```typescript
const hashpassword = await bcrypt.hash(userData.password, 10); // ✅ 10 rounds
const match = await bcrypt.compare(pass, user.password); // ✅ Timing-safe
```

**Tokens cryptographiquement sécurisés**
- Fichier: `app/backend/src/auth/auth.service.ts:78`
```typescript
const verificationToken = crypto.randomBytes(32).toString("hex"); // ✅
```

**Reset tokens avec expiration**
- Fichier: `app/backend/src/users/users.service.ts:298-299`
```typescript
expiryDate.setHours(expiryDate.getHours() + 1); // ✅ Expire en 1h
```

**Webhook Stripe signé**
- Fichier: `app/backend/src/payment/payment.service.ts:27-36`
```typescript
event = this.stripe.webhooks.constructEvent(rawBody, signature, webhookSecret);
```

---

### 🔴 Failles Critiques

**1. HTTPS non forcé**
- ❌ Aucun redirect HTTP → HTTPS
- ❌ Pas de header HSTS

**Recommandation:**
```typescript
if (process.env.NODE_ENV === 'production') {
  app.use((req, res, next) => {
    if (req.header('x-forwarded-proto') !== 'https') {
      res.redirect(301, `https://${req.header('host')}${req.url}`);
    } else next();
  });
}
```

Géré par Helmet, mais peux aussi être géré par ngninx : 
 C'est juste un doublon inutile. Si on met le HSTS dans nginx en production, on peut supprimer celui de helmet pour garder une seule source de vérité. Mais rien d'urgent, c'est un détail de propreté. Pas de problème fonctionnel. Le navigateur reçoit deux headers Strict-Transport-Security identiques, il prend simplement le max-age le plus élevé entre les deux. Rien ne casse.  

**2. Tokens stockés en plain text**
- Fichier: `app/backend/prisma/schema.prisma`
```prisma
passwordResetToken       String?  // ❌ Plain text
emailVerificationToken   String?  // ❌ Plain text
```

**Recommandation:** Stocker `hash(token)` et comparer

**3. Tokens dans URLs d'emails**
- Fichier: `app/backend/src/email/email.service.ts:59`
```typescript
const verificationUrl = `${frontendUrl}/verify-email?token=${verificationToken}`;
// ❌ Token visible dans logs serveurs mail
```
OK fait.

---

## A05:2025 - Injection

### ✅ Points Conformes

**ORM Prisma (prévient SQL Injection)**
- Utilisation cohérente de Prisma pour toutes requêtes
- Aucune requête SQL brute
- Paramètres échappés par défaut

**Validation avec class-validator**
- Fichier: `app/backend/src/users/dtos/register.dto.ts`
```typescript
@IsEmail()
@MaxLength(255)
@Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
password: string;
```

**Global ValidationPipe**
- Fichier: `app/backend/src/main.ts:26-35`
```typescript
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,           // ✅ Rejette champs non définis
  forbidNonWhitelisted: true, // ✅ Erreur sur extras
  transform: true,
}));
```

**Pas de Command Injection**
- Aucun `exec()`, `spawn()`, `child_process`

---

### 🟡 Améliorations Recommandées

**Search sans maxLength**
- Fichier: `app/backend/src/trees/dtos/get-trees.dto.ts`
```typescript
search?: string; // ❌ Pas de @MaxLength
```

**Recommandation:**
```typescript
@IsOptional()
@MaxLength(50)
search?: string;
```

---

## A06:2025 - Insecure Design

### ✅ Points Conformes

**Transactions atomiques**
- Fichier: `app/backend/src/checkout/checkout.service.ts:55`
```typescript
return this.prisma.$transaction(async (tx) => {
  // ✅ Tout ou rien
});
```

**Réservation stock AVANT paiement**
```typescript
// 1. Vérifier stock disponible
// 2. Décrémenter atomiquement
// 3. Puis appeler Stripe ✅
```

**State machine pour statuts commandes**
- Fichier: `app/backend/src/orders/orders.constants.ts`
```typescript
export const ORDER_STATUS_TRANSITIONS = {
  PENDING: [OrderStatus.CANCELLED],
  CONFIRMED: [OrderStatus.PLANTED, OrderStatus.CANCELLED],
  PLANTED: [],    // ✅ État final
  CANCELLED: [],  // ✅ État final
};
```

---

### 🔴 Failles Critiques

**1. Race condition sur le stock**
```
User A: POST /checkout/init (stock=105, qty=100)
User B: POST /checkout/init (stock=105, qty=100) // Même stock vu
User A: POST /checkout/confirm → OK
User B: POST /checkout/confirm → ÉCHEC après navigation
```

  1. Race Condition sur le Stock                                                                                                                                                                                                                                                                       
  Verdict : Risque RÉEL mais faible pour ton projet                                                                                                                                                                                                                                                                                                                                                                                                 
  Le code actuel utilise une transaction Prisma ($transaction) mais ce n'est pas suffisant pour éviter totalement la race condition. Voici pourquoi :                                                                                      
                                                            
  // checkout.service.ts:121-223                                                                                                                                                                                                           
  // Transaction standard = lecture puis écriture                                                                                                                                                                                          
  // Pas de verrou sur la ligne pendant la lecture                                                                                                                                                                                                                         
  Scénario problématique :                                                                                                                                                                                                             
  User A: lit stock = 10, demande 8 → OK                                                                                                                                                                                                   
  User B: lit stock = 10 (même instant), demande 8 → OK                                                                                                                                                                                    
  User A: décrémente → stock = 2                                                                                                                                                                                                           
  User B: décrémente → stock = -6 ❌                                                                                                                                                                                                                            
  Cependant, la vérification frontend avant le confirm réduit significativement le risque. Pour un projet avec peu de trafic concurrent, c'est acceptable. 

**Recommandation:** Pessimistic locking
```typescript
const tree = await tx.tree.findUniqueOrThrow({
  where: { id: item.treeId },
  // SELECT ... FOR UPDATE
});
```

**2. Pas de validation stock à l'init**
- Fichier: `app/backend/src/checkout/checkout.service.ts:306-321`
- L'utilisateur peut ajouter 999999 arbres, erreur seulement au confirm

**3. Remboursement non implémenté**
- Fichier: `app/backend/src/orders/orders.service.ts:201`
```typescript
// TODO: Évolution future - Gérer le remboursement via Stripe
```
- Commandes PAID annulées = pas de remboursement automatique

  1. Limitation des quantités au checkout                                                                                                                                                                                                                  
  Fichier modifié : app/backend/src/checkout/dtos/init-checkout.dto.ts                                                                                                                                                                                                                                 
  Problème corrigé : Un utilisateur pouvait envoyer une quantité illimitée d'arbres (ex: 999999), provoquant des calculs inutiles et une mauvaise UX (erreur tardive au moment du confirm).                                                
                                                            
  Corrections apportées :                                                                                                                                                                                                                    
  // CheckoutItemDto                                                                                                                                                                                                                       
  @Max(100, { message: 'La quantité maximum est de 100 par type d\'arbre' })                                                                                                                                                               
  quantity: number;                                                                                                                                                                                                                          
  // InitCheckoutDto                                                                                                                                                                                                                       
  @ArrayMaxSize(50, { message: 'Le panier ne peut pas contenir plus de 50 types d\'arbres différents' })                                                                                                                                   
  items: CheckoutItemDto[];                                                                                                                                                                                                                                
  Limites :                                                                                                                                                                                                                          
  - Max 100 arbres par type                                                                                                                                                                                                                
  - Max 50 types d'arbres différents par commande                                                                                                                                                                                          
  - Soit un maximum théorique de 5000 arbres par commande                                                                                                                                                                                                                          
  ---                                                                                                                                                                                                                                      
  2. Remboursement automatique Stripe                                                                                                                                                                                                                           
  Fichiers modifiés :                                                                                                                                                                                                                      
  - app/backend/prisma/schema.prisma - Ajout du statut REFUNDED                                                                                                                                                                            
  - app/backend/src/orders/orders.service.ts - Logique de remboursement                                                                                                                                                                                                                     
  Problème corrigé : Les commandes payées puis annulées n'étaient pas remboursées automatiquement.                                                                                                                                         
                                                          
  Comportement :                                                                                                                                                                                                                      
  - Quand une commande PAID est annulée → remboursement Stripe automatique                                                                                                                                                                 
  - Le paymentStatus passe à REFUNDED                                                                                                                                                                                                      
  - Le stock est restauré (si réservé)                                                                                                                                                                                                     
  - En cas d'échec Stripe → erreur 400 avec message explicite                                                                                                                                                                                                                       
  // Remboursement via Stripe                                                                                                                                                                                                              
  await this.stripe.refunds.create({                                                                                                                                                                                                       
    payment_intent: order.paymentIntentId,                                                                                                                                                                                                 
  });                                                                                                                                                                                                                               
  Nouveau statut Prisma :                                                                                                                                                                                                                  
  enum PaymentStatus {                                                                                                                                                                                                                     
    PENDING                                                                                                                                                                                                                                
    PROCESSING                                                                                                                                                                                                                             
    PAID                                                                                                                                                                                                                                   
    FAILED                                                                                                                                                                                                                                 
    REFUNDED  // ← Nouveau                                                                                                                                                                                                                 
  }                                                                                                                                                                    
---

### 🟡 Améliorations Recommandées

- Limiter max items par commande (100)
- Ajouter timeout sur commandes PENDING
- Idempotency keys pour requêtes Stripe

---

## A07:2025 - Authentication Failures

### ✅ Points Conformes

**Vérification email obligatoire**
- Fichier: `app/backend/src/auth/strategies/jwt.strategy.ts:58-60`
```typescript
if (!user.isEmailVerified) {
  throw new UnauthorizedException('Email non vérifié');
}
```

**Rôle rechargé depuis DB à chaque requête**
```typescript
async validate(payload: JwtPayload) {
  const user = await this.prisma.user.findUnique({ where: { id: payload.sub } });
  return { userId: user.id, role: user.role.name }; // ✅ Rôle actuel
}
```

**Vérification mot de passe actuel pour changement**
- Fichier: `app/backend/src/auth/auth.service.ts:186-194`
```typescript
const isPasswordValid = await bcrypt.compare(currentPassword, user.password);
if (!isPasswordValid) {
  throw new UnauthorizedException('Le mot de passe actuel est incorrect');
}
```

**Token verification one-time use**
```typescript
data: {
  isEmailVerified: true,
  emailVerificationToken: null, // ✅ Supprimé après use
}
```

---

### 🔴 Failles Critiques

**1. Pas de brute force protection par username**
- Throttle 5 req/min par IP globale
- Mais aucune limite par email/username

**Recommandation:**
```typescript
await this.loginAttemptsService.recordFailure(email);
const attempts = await this.loginAttemptsService.getAttempts(email);
if (attempts > 5) {
  throw new TooManyRequestsException('Compte temporairement bloqué');
}
```

**2. Pas de logout avec révocation**
- Aucun endpoint `/auth/logout`
- JWT reste valide jusqu'à expiration

**Recommandation:** Blacklist Redis
```typescript
@Post('logout')
async logout(@Request() req) {
  const token = req.headers.authorization.split(' ')[1];
  await this.redis.setex(`blacklist:${token}`, 86400, '1');
}
```
                                                                                                           
  ## Protection brute force par email                                                                                                                                                                                                              
  ### Principe                                                                                                                                                                                                                           
  Chaque tentative de connexion échouée est comptabilisée par email dans Redis.                                                                                                                                                            
  Après 5 échecs, le compte est verrouillé pendant 15 minutes, quelle que soit l'IP.                                                                                                                                                       
                                                             
  ### Flux                                                                                                                                                                                                                              
  1. **Requête login** → `POST /auth/login` avec email + password                                                                                                                                                                          
  2. **Vérification Redis** → `GET login_attempts:{email}`                                                                                                                                                                                 
  3. **Si >= 5 échecs** → Erreur 429 "Compte temporairement bloqué"                                                                                                                                                                        
  4. **Sinon** → Vérification du mot de passe                                                                                                                                                                                              
     - **Échec** → `INCR login_attempts:{email}` (TTL 15 min)                                                                                                                                                                              
     - **Succès** → `DEL login_attempts:{email}` (reset compteur)                                                                                                                                                                                                                                                                                                                                                                                            
  ### Stockage Redis                                                                                                                                                                                                                           
  | Clé | Valeur | TTL |                                                                                                                                                                                                                   
  |-----|--------|-----|                                                                                                                                                                                                                   
  | `login_attempts:user@example.com` | `3` | 14:32 |                                                                                                                                                                                      
  | `login_attempts:hacker@test.com` | `5` | 12:45 (bloqué) |                                                                                                                                                                                                                         
  ### Comportement                                                                                                                                                                                                                                                                             
  - Premier échec → création clé avec TTL 15 min                                                                                                                                                                                           
  - Échecs suivants → incrémentation du compteur                                                                                                                                                                                           
  - Connexion réussie → suppression de la clé                                                                                                                                                                                              
  - Après 15 min sans échec → clé expire automatiquement                                                                                                                                                                                                                      
  ### Dégradation gracieuse                                                                                                                                                                                                                                                                              
  Si Redis est indisponible, la protection est désactivée (fail-open).                                                                                                                                                                     
  L'application continue de fonctionner avec uniquement le rate-limit par IP.                                     

## Vue d'ensemble                                                                                                                                                                                                                        
                                                                                                                                                                                                                                           
  Deux mécanismes de sécurité basés sur Redis ont été implémentés :                                                                                                                                                                                                                              
  1. **Protection brute force par email** - Bloque un compte après 5 tentatives échouées                                                                                                                                                   
  2. **Révocation JWT au logout** - Invalide immédiatement le token lors de la déconnexion                                                                                                                                                 
                                                              
  ## 1. Protection brute force par email                                                                                                                                                                                                                                
  ### Problème résolu                                                                                                                                                                                                                                
  Le rate-limiting par IP (`@Throttle`) ne protège pas contre les attaques distribuées.                                                                                                                                                    
  Un attaquant avec 100 IPs peut tester 500 mots de passe/minute sur un même compte.                                                                                                                                                       
                                                             
  ### Solution                                                                                                                                                                                                                            
  Compteur de tentatives échouées par email stocké dans Redis.                                                                                                                                                                                                                       
  ### Comportement                                                                                                                                                                                                                    
  | Tentatives échouées | Action |                                                                                                                                                                                                         
  |---------------------|--------|                                                                                                                                                                                                         
  | 1-4 | Erreur 401 normale |                                                                                                                                                                                                             
  | 5+ | Erreur 429 "Compte temporairement bloqué" |                                                                                                                                                                                       
  | Après 15 min | Reset automatique |                                                                                                                                                                                                     
  | Login réussi | Reset immédiat |                                                                                                                                                                                                        
                                                           
  ### Stockage Redis                                                                                                                                                                                                                       
                                                            
  Clé: login_attempts:{email}                                                                                                                                                                                                              
  Valeur: nombre d'échecs (1-5)                                                                                                                                                                                                            
  TTL: 15 minutes                                                                                                                                                                                                                           
  ### Dégradation gracieuse                                                                                                                                                                                                                       
  Si Redis est indisponible, la protection est désactivée (fail-open).                                                                                                                                                                     
  L'application continue de fonctionner avec uniquement le rate-limit par IP.                                                                                                                                                              
                                                           
  ---                                                                                                                                                                                                                             
  ## 2. Révocation JWT au logout                                                                                                                                                                                                                           
  ### Problème résolu                                                                                                                                                                                                                            
  Sans révocation, un JWT reste valide jusqu'à son expiration (1h).                                                                                                                                                                 
  Un token volé peut être utilisé même après que l'utilisateur se soit déconnecté.                                                                                                                                                         
                                                              
  ### Solution                                                                                                                                                                                                                          
  Blacklist des tokens dans Redis lors du logout.                                                                                                                                                                                                                          
  ### Flux                                                                                                                                                                                                                            
  POST /auth/logout                                                                                                                                                                                                                        
       ↓                                                                                                                                                                                                                                   
  Token ajouté à Redis (blacklist:{hash})                                                                                                                                                                                                  
  TTL = temps restant du JWT                                                                                                                                                                                                               
       ↓                                                                                                                                                                                                                                   
  Requêtes suivantes avec ce token → 401 "Token révoqué"                                                                                                                                                                                                                    
  ### Stockage Redis                                                                                                                                                                                                                            
  Clé: blacklist:{token_fingerprint}                                                                                                                                                                                                       
  Valeur: "1"                                                                                                                                                                                                                              
  TTL: temps restant avant expiration du JWT                                                                                                                                                                                                                            
  ### Implémentation frontend                                                                                                                                                                                                                          
  ```typescript                                                                                                                                                                                                                            
  // Hook useLogoutMutation.ts                                                                                                                                                                                                             
  const { mutate: logout } = useLogoutMutation();                                                                                                                                                                                                                               
  // Appel                                                                                                                                                                                                                                 
  logout(); // → POST /api/auth/logout puis clear local state                                                                                                                                                                                                                             
  Dégradation gracieuse                                                                                                                                                                                                                        
  Si Redis est indisponible :                                                                                                                                                                                                            
  - Le logout côté client fonctionne (suppression token local)                                                                                                                                                                             
  - Le token expire naturellement après 1h                                               


**3. Email enumeration possible**
- Si email existe → erreur DB spécifique
- Révèle l'existence d'utilisateurs

**Recommandation:**
```typescript
const existing = await this.usersService.findOneByEmail(email);
if (existing) {
  throw new BadRequestException('Cet email est déjà utilisé');
}
```
                                                                                                                                                                             
 En pratique, l'énumération à l'inscription est souvent considérée comme acceptable car :                                                                                                                                            
  - L'utilisateur doit savoir s'il a déjà un compte                                                                                                                                                               
  - L'alternative (message générique) est confuse UX            


**4. Pas de notification après reset password**
- Attaquant réinitialise le mot de passe, utilisateur non notifié

---

### 🟡 Améliorations Recommandées

- Implémenter refresh tokens
- Ajouter 2FA/TOTP
- Expulser sessions après changement de mot de passe

---

## A08:2025 - Software or Data Integrity Failures

### ✅ Points Conformes

**Webhook Stripe signé et vérifié**
```typescript
event = this.stripe.webhooks.constructEvent(rawBody, signature, webhookSecret);
```

**Vérification PaymentIntent**
- Fichier: `app/backend/src/payment/payment.service.ts:95-101`
```typescript
if (order.paymentIntentId !== paymentIntent.id) {
  this.logger.error(`SÉCURITÉ : PaymentIntent ne correspond pas`);
  return;
}
```

---

### 🔴 Failles Critiques

**1. Cloudinary URL validation bypassable**
- Fichier: `app/backend/src/cloudinary/cloudinary.service.ts:59-65`
```typescript
const pattern = new RegExp(`^https://res\\.cloudinary\\.com/${this.cloudName}/`);
// ❌ Bypassable: https://res.cloudinary.com/../evil.com/
```

**Recommandation:** Parser en URL et vérifier hostname

**2. Pas de virus scan sur uploads**
- Avatars uploadés sans vérification
- Risque de malware via CDN

**3. Pas d'audit trail**
- Qui a modifié quoi? Quand?
- Aucune table d'historique

---

## A09:2025 - Security Logging and Alerting Failures

### ✅ Points Conformes

**Logger NestJS utilisé**
```typescript
private readonly logger = new Logger(AuthService.name);
this.logger.log(`Connexion réussie pour l'utilisateur ID: ${user.id}`);
```

---

### 🔴 Failles Critiques

**1. Logs console en production**
```typescript
console.log("✅ Database connected"); // ❌
console.error('Erreur Resend:', error); // ❌
```

3 raisons principales:                                                                                                                                                                                                                   
  1. Sécurité en production - console.log affiche tout (y compris les stack traces sensibles). Logger NestJS gère les niveaux et peut masquer les détails en prod                                                                          
  2. Gestion centralisée - Un seul endroit pour filtrer/configurer tous les logs. Facile de désactiver le debug en production                                                                                                              
  3. Professionnel - Structured logging avec timestamps, contexte, niveaux (debug/log/warn/error) plutôt que du texte brut 

**2. Pas de détection d'attaques**
- Pas de détection brute force par username
- Pas d'alerte sur modifications admin

**3. Pas de centralisation des logs**
- Logs sur stdout uniquement
- Impossible d'auditer post-incident

**Recommandation:** Intégrer ELK, Datadog ou Splunk

**4. Logs sensibles non redactés**
- Possible logging accidentel de tokens/passwords

---

## A10:2025 - Mishandling of Exceptional Conditions

### ✅ Points Conformes

**Exceptions NestJS standardisées**
```typescript
throw new NotFoundException('Commande introuvable');
throw new ForbiddenException('Accès refusé');
```

**Messages génériques appropriés**
```typescript
throw new BadRequestException("Signature du webhook invalide");
// ✅ Ne révèle pas la raison exacte
```

---

### 🔴 Failles Critiques

**1. Pas de timeout sur transactions**
```typescript
return this.prisma.$transaction(async (tx) => {
  // ❌ Aucun timeout configuré
});
```

**Recommandation:**
```typescript
return this.prisma.$transaction(async (tx) => { ... }, { timeout: 30000 });
```

**2. Pas de circuit breaker pour Stripe**
- Si Stripe rate-limit, chaque requête échoue
- Pas de graceful degradation

---

### 🟡 Améliorations Recommandées

- Ajouter retry logic pour appels externes
- Implémenter health checks (`@nestjs/terminus`)
- Queue emails si Resend down

---

## Plan de Remédiation Prioritaire

### Phase 1 - Urgent (1-2 jours)

| Action | Fichier | Effort |
|--------|---------|--------|
| Ajouter Helmet | `main.ts` | 15 min |
| Configurer CORS strict | `main.ts` | 10 min |
| Désactiver Swagger en prod | `main.ts` | 5 min |
| Remplacer console.log par Logger | Multiple | 30 min |
| Valider JWT_SECRET > 32 chars | `app.module.ts` | 10 min |

### Phase 2 - Important (1-2 semaines)

| Action | Fichier | Effort |
|--------|---------|--------|
| Brute force protection par email | `auth.service.ts` | 4h |
| Logout avec JWT blacklist | `auth.controller.ts` | 6h |
| Hash reset tokens en DB | `users.service.ts` | 2h |
| Pessimistic locking stock | `checkout.service.ts` | 2h |
| Implémenter Stripe refund | `orders.service.ts` | 8h |

### Phase 3 - Amélioration (1-2 mois)

| Action | Effort |
|--------|--------|
| Centraliser logs (ELK/Datadog) | 2-3 jours |
| Ajouter 2FA/TOTP | 1 semaine |
| Audit logs complets | 1 semaine |
| Circuit breaker (Stripe/Cloudinary) | 2-3 jours |
| Health checks endpoint | 2h |

---

## Checklist de Vérification

```
Phase 1:
[X] Helmet installé et configuré
[X] CORS restreint à FRONTEND_URL
[X] Swagger conditionné à NODE_ENV
[X] console.log remplacés par Logger
[X] JWT_SECRET validé > 32 caractères

Phase 2:
[X] Brute force protection par email
[X] JWT blacklist après logout
[X] Reset tokens hashés
[X] Stock avec pessimistic locking
[X] Refund Stripe implémenté

Phase 3:
[ ] Logs centralisés
[ ] 2FA disponible
[ ] Audit trail complet
[ ] Circuit breakers actifs
[ ] Health checks fonctionnels
```

---
Voici les points identifiés comme encore vulnérables ou nécessitant une intervention pour finaliser la sécurisation.

### 🔴 Failles Critiques (Priorité Haute)
* **A02 - Raw Body Global :** Le paramètre `rawBody: true` est encore actif sur toute l'application. Il doit être restreint au seul endpoint des Webhooks Stripe pour limiter les risques de DoS.
* **A03 - Supply Chain :** Les dépendances utilisent encore des versions flottantes (ex: `^20.0.0`). Il est nécessaire d'épingler les versions exactes et d'ajouter un `npm audit` au pipeline CI.
* **A08 - Validation Cloudinary :** La regex de validation des URLs d'images est contournable. Un attaquant peut encore injecter des liens vers des domaines tiers.
* **A10 - Timeouts DB :** Les transactions Prisma n'ont pas de timeout configuré. Une transaction longue peut bloquer les ressources de la base de données.

### 🟡 Améliorations & Phase 3 (Maturité)
* **Audit Trail :** Absence d'historique des modifications (qui a modifié quel arbre/commande et quand).
* **Centralisation des logs :** Les logs sont limités à la console serveur (stdout). Une centralisation (ELK/Datadog) est nécessaire pour l'audit post-incident.
* **Résilience :** Absence de *Circuit Breakers* pour gérer les pannes des services externes (Stripe/Resend).

## Ressources

- [OWASP Top 10:2025](https://owasp.org/Top10/2025/)
- [NestJS Security](https://docs.nestjs.com/security/authentication)
- [Helmet.js](https://helmetjs.github.io/)
- [Prisma Security](https://www.prisma.io/docs/concepts/components/prisma-client/raw-database-access)

---

*Rapport généré automatiquement par Claude Code*
