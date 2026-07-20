# Script oral - Soutenance CDA GreenRoots

Texte parlé, slide par slide, dans l'ordre du deck. Registre oral (phrases courtes), pensé pour être dit à voix haute et répété. Les `(transition)` relient les slides. Repères de durée indicatifs. Les notes à l'écran (vue présentateur, touche S) reprennent ces idées en version télégraphique.

---

## BLOC 1 - Introduction

### Cover
Bonjour à tous. Je m'appelle Tony Memponteil, et je vais vous présenter le projet de fin d'études sur lequel j'ai travaillé, GreenRoots, mené sur environ un mois.

### Intercalaire - Introduction
Dans cette première partie, je vais vous présenter rapidement qui je suis, le projet, son périmètre, et notre méthode de travail.

### Mon parcours
Un mot sur mon parcours, parce qu'il explique ma façon de travailler.
Je viens du secteur financier, où j'ai passé neuf ans - un métier de rigueur.
En 2023, j'ai fait le grand saut avec la formation DWWM : ça a été le déclic, la confirmation d'une vocation.
J'ai enchaîné sur une alternance CDA au Groupe LDLC, deux ans sur l'ERP interne, un outil transverse à tous les métiers de l'entreprise.
Le fil rouge de tout ça, c'est une envie simple : concevoir, puis construire, et voir tourner ce qu'on a imaginé.

### Le projet GreenRoots
Le projet GreenRoots n'est pas simplement un site e-commerce avec un catalogue d'arbres, c'est un peu plus.
C'est parti d'un constat simple : beaucoup de gens veulent agir pour le climat, mais ne savent pas vraiment comment, ni à qui faire confiance.
GreenRoots répond à ça en rendant la reforestation aussi simple qu'un achat en ligne : on choisit un arbre, on paie, il est planté.
Mais notre vrai parti pris, c'est la traçabilité : pour chaque arbre, on sait quelle essence et à quel endroit il est planté. On transforme une intention en une action concrète et vérifiable - c'est tout le sens de notre signature : transformer ses intentions en actions concrètes, un arbre à la fois.
La plateforme vise d'abord les particuliers - c'est le périmètre de notre MVP - puis, à terme, les entreprises engagées dans une démarche RSE.
Un mot pour cadrer, en toute transparence : GreenRoots est un cas d'école, réalisé dans le cadre de ma formation CDA. Mais on l'a mené avec l'ambition d'un projet production-ready - vrai paiement, vrai déploiement, vraies exigences de sécurité.

### Périmètre & acteurs
*(transition)* Alors concrètement, à qui s'adresse la plateforme, et jusqu'où va notre MVP ?
On a défini trois acteurs, organisés en hiérarchie d'accès.
Le visiteur, non connecté : il consulte le catalogue, les fiches arbres, et peut remplir son panier.
L'utilisateur connecté : il hérite de tout ça, et débloque le tunnel d'achat complet avec paiement Stripe, plus son espace personnel - profil et historique de commandes.
Et l'administrateur : il gère le back-office - le catalogue des arbres en CRUD, et le suivi des commandes.
Côté périmètre, un point important : le cœur du MVP, c'est le parcours d'achat de bout en bout, avec un vrai paiement - Stripe fait partie du MVP, ce n'est pas simulé.
Et tout ce qui est autour - carte interactive, suivi de la pousse, multilingue, parrainage - a été écarté volontairement : ce n'est pas oublié, c'est un choix de priorisation.

### Organisation & méthode
Pour mener ce projet, on était une équipe de cinq : Marie au Product Owner, Léo Scrum Master, Vincent lead développeur back, moi lead développeur front, et Naomi en full-stack.
On a travaillé en Agile, méthode Scrum, sur des sprints d'une semaine, avec les cérémonies habituelles - le daily, la revue, la rétro.
Et nos sprints racontent une histoire : un sprint 0 de conception, puis les fondations, puis le MVP complet, et enfin un sprint 3 dédié à la qualité et à la mise en production.
Côté outillage : Trello pour le backlog, GitHub pour le versioning - avec des pull requests systématiquement relues - Google Docs pour la documentation, et des conventions partagées : Conventional Commits, ESLint et Prettier.

---

## BLOC 2 - Conception

### Intercalaire - Conception
*(transition)* Voilà pour le cadre. Entrons maintenant dans la conception : comment on est passé du besoin à l'architecture.

### Du besoin aux user stories
Tout part du cahier des charges, rédigé au sprint 0.
On l'a traduit en vingt-huit user stories, avec le format classique : en tant que tel utilisateur, je veux telle action, afin d'obtenir tel bénéfice.
Ça force à raisonner en valeur pour l'utilisateur, pas en solution technique.
Ces user stories alimentent directement notre backlog Trello.
Et une fois regroupées par acteur, elles nous donnent le diagramme de cas d'utilisation.

### Diagramme de cas d'utilisation
Voici ce diagramme. On y retrouve nos trois acteurs en hiérarchie d'accès.
Deux relations sont importantes en UML. Le include, un cas toujours appelé : par exemple passer commande inclut forcément vérifier le stock et payer. Et le extend, un cas optionnel : par exemple rembourser, qui étend gérer les commandes.
Ce inclut vérifier le stock, on le retrouvera très concrètement dans le code, au moment du tunnel de vente.

### Du wireframe à la maquette
Côté interface, on a suivi une démarche progressive.
D'abord le wireframe, sous Balsamiq : la structure, le zoning, le parcours, sans style.
Puis la maquette haute-fidélité, sous Figma : la charte, les couleurs, les vrais composants.
Le responsive a été pensé dès la maquette, en desktop et en mobile.
Et la conformité entre la maquette et le produit final, vous pourrez la vérifier tout à l'heure, en direct, pendant la démo.

### Qualité de l'interface
Pour garantir la qualité de l'interface, plusieurs leviers concrets.
Lighthouse, d'abord : au sprint 3, on a mesuré et optimisé les performances, l'accessibilité, les bonnes pratiques et le référencement.
Le responsive, ensuite : avec Tailwind, mais surtout un hook, useMediaQuery, qui fait carrément changer de composant - un tableau de commandes sur ordinateur devient des cartes sur mobile, ce n'est pas juste du CSS.
Les composants sont organisés en Atomic Design, pour la réutilisabilité et la cohérence.
Et l'accessibilité est réelle : par exemple notre champ de quantité expose un rôle spinbutton, donc un lecteur d'écran annonce la valeur et les bornes à un utilisateur non-voyant.

### Justification de la stack
Un mot sur la stack, parce qu'aucun choix n'est au hasard.
Trois fils directeurs : la sûreté de typage, l'éco-conception, et l'accessibilité.
Il y a aussi une part de pragmatisme assumée : des technos qu'on maîtrisait déjà plus ou moins, dans un délai court.
Le choix emblématique, c'est shadcn/ui : il coche les trois cases - accessible grâce à Radix, éco parce qu'on n'importe que ce qu'on utilise, et entièrement typé.
Et Stripe, enfin, nous permet de ne stocker aucune donnée bancaire - tout est délégué.

### Une architecture 3 tiers découplée
Passons à l'architecture. Elle est en trois tiers, trois couches déployables séparément : la présentation, une application React ; la logique métier, une API NestJS ; et les données, PostgreSQL avec Redis.
Tout est découplé - une application front d'un côté, une API REST de l'autre, sans framework de rendu serveur, ce qui est adapté à une application derrière authentification.
Nginx sert de reverse proxy : le navigateur ne parle jamais directement au back ni à la base.
Et surtout, la sécurité est pensée dès la conception, selon la grille DICP - disponibilité, intégrité, confidentialité, preuve - conformément aux recommandations de l'ANSSI et de l'OWASP.

### Architecture applicative
Si on zoome à l'intérieur des briques.
Côté back, NestJS repose sur l'injection de dépendances et une organisation modulaire - une quinzaine de modules par domaine. Une requête suit toujours le même trajet : le contrôleur reçoit le HTTP, le service porte la logique métier, Prisma accède aux données.
En toute honnêteté, c'est une architecture en couches, pas une hexagonale stricte - le service dépend directement de Prisma, un choix pragmatique.
Côté front, Atomic Design et découpage par fonctionnalité. Et mon point fort : la séparation des trois natures d'état - chacune avec le bon outil, plutôt qu'un seul gros store fourre-tout.

### Modélisation MERISE : MCD vers MLD
Pour la base de données, on a suivi la méthode MERISE, en trois niveaux.
Le modèle conceptuel, le métier pur. Puis le modèle logique, obtenu par les règles de passage - les associations deviennent des tables et des clés étrangères. Par exemple, une commande contient des arbres devient une table OrderItem.
Et le troisième niveau, le modèle physique, c'est la slide suivante.

### MPD - le passage au physique
Le modèle physique, lui, dépend vraiment de PostgreSQL. Quelques décisions concrètes.
Les prix sont stockés en entiers, en centimes - le standard Stripe, pour éviter les erreurs d'arrondi.
Les statuts sont des enums, qui serviront de support aux machines à états.
J'ai séparé l'arbre de son stock en deux tables, parce que le stock change très souvent.
Le pont entre le code en camelCase et la base en snake_case se fait avec l'annotation @map.
Et tout est versionné : vingt-quatre migrations, pour une traçabilité complète du schéma.

---

## BLOC 3 - Tunnel de vente

### Intercalaire - Le tunnel de vente
*(transition)* On arrive au cœur du projet, mon fil rouge : le tunnel de vente.

### Démo - le parcours d'achat
Avant de décortiquer le code, je vous propose de voir le produit fini en action.
Je vais dérouler un parcours d'achat complet, sur l'application déployée en production : le catalogue, une fiche arbre, l'ajout au panier, le checkout avec une carte de test, et la confirmation du paiement.
Je suis déjà connecté, et Stripe est en mode test. Si le réseau nous joue des tours, j'ai des captures en secours.

*(Faire la démo, puis enchaîner.)*

### Vue d'ensemble du tunnel
Pourquoi ce parcours comme fil rouge ? Parce que c'est la fonctionnalité la plus complète : elle concentre les vraies décisions - la transaction, le paiement, la sécurité, la gestion d'état.
Je vais vous la présenter sous deux angles.
Côté utilisateur, le parcours : panier, checkout, paiement, confirmation.
Et côté technique, la même donnée qui descend couche par couche : de React à NestJS, puis Prisma, puis PostgreSQL.

### Le formulaire de checkout
Première étape côté front : le formulaire de checkout, l'adresse et le paiement.
Toute la validation repose sur Zod : un seul schéma définit à la fois les règles et le type TypeScript, donc les deux ne peuvent jamais diverger. Et c'est le cas pour tous les formulaires du site.
On le couple à React Hook Form, qui gère les champs et la soumission de façon performante.
Les champs viennent de shadcn/ui, avec le label et les messages d'erreur reliés en ARIA.
Et cette validation côté client est toujours redoublée côté serveur, par les DTO - c'est de la défense en profondeur.

### Trois natures d'état
Deuxième étape front : la gestion de l'état. Mon principe, c'est de ne pas tout mettre dans un état global, mais de séparer par nature.
L'état local, le panier, avec Zustand - persisté dans le navigateur, avec une expiration de vingt-quatre heures.
L'état serveur, le stock, avec TanStack Query - rafraîchi régulièrement.
Et l'état de formulaire, avec React Hook Form et Zod.
Un mécanisme important : syncWithStock, qui réconcilie le panier local avec le stock réel - si un article est épuisé, la quantité est ajustée et l'utilisateur est prévenu.

### Cycle de vie : deux dimensions d'état
Une commande a deux dimensions d'état, que j'ai modélisées explicitement.
L'avancement de la commande est une vraie machine à états : une table déclare les transitions autorisées, et un garde refuse tout le reste - donc jamais d'état incohérent.
Le statut de paiement, lui, est un enum piloté par le webhook Stripe.
C'est la charnière entre la conception - les enums du modèle - et le code qu'on va voir juste après.

### Réserver le stock : confirmCheckout
Côté back, première pièce maîtresse : la réservation du stock, dans la méthode confirmCheckout.
L'objectif : réserver le stock de façon atomique, sécurisée et idempotente, avant le paiement.
Tout se passe dans une transaction : c'est du tout-ou-rien.
On vérifie d'abord que la commande appartient bien à l'utilisateur, sinon on renvoie une erreur 403 - c'est du contrôle d'accès, le risque numéro un de l'OWASP.
Ensuite l'idempotence : si le stock est déjà réservé, par exemple sur un double-clic, on ne rejoue pas.
Et on valide tout le stock avant d'écrire : s'il manque un seul article, rien n'est décrémenté.

### Stripe : le webhook, source de vérité
Deuxième pièce back : le paiement, dont la source de vérité est le webhook Stripe.
Le résultat du paiement n'est jamais décidé par le navigateur, mais par Stripe, qui notifie mon serveur.
J'en vérifie la signature HMAC : seul Stripe peut la produire, une signature invalide est rejetée.
Le webhook est exclu du rate-limiting, parce que Stripe rejoue ses notifications tant qu'il n'a pas eu de réponse.
Je gère trois événements : paiement réussi, échoué - qui restaure le stock - et annulé. Avec deux protections : l'anti-usurpation et l'idempotence.
Et pour la résilience, un cron rattrape les webhooks qui se seraient perdus.

### De l'ORM au SQL : la transaction
Un point que le jury attend souvent : comprendre le SQL derrière l'ORM.
À gauche ma transaction en Prisma, à droite son équivalent SQL.
Le transaction, c'est un BEGIN suivi d'un COMMIT.
Les include imbriqués deviennent des jointures.
Le decrement devient un UPDATE atomique - et comme il y a une boucle, c'est un UPDATE par arbre de la commande.
Enfin, les requêtes sont paramétrées - donc l'injection SQL est tout simplement impossible.

### Redis : NoSQL clé-valeur, en mémoire
Dernière couche, les données NoSQL, avec Redis - une base clé-valeur en mémoire, de l'ordre de la fraction de milliseconde. Trois usages concrets.
Le cache, en cache-aside : les pays pendant vingt-quatre heures, les catégories une heure, avec invalidation - ça réduit les requêtes SQL redondantes, c'est aussi de l'éco-conception.
L'anti-force-brute : cinq échecs de connexion et le compte est bloqué quinze minutes.
Et la blacklist des jetons au logout.
Point important : si Redis tombe, l'application continue - elle se rabat sur la base. Le cache accélère, mais n'est pas critique.

---

## BLOC 4 - Sécurité, tests & conformité

### Intercalaire - Sécurité, tests & conformité
*(transition)* J'en arrive aux exigences transverses du titre : la sécurité, les tests, et la conformité.

### Authentification : défense en profondeur
La sécurité, d'abord par l'authentification, en défense en profondeur.
La décision clé : le jeton d'authentification est stocké dans un cookie HttpOnly, donc illisible en JavaScript - une faille XSS ne peut pas le voler.
Les mots de passe sont hachés avec bcrypt : salés, et volontairement lents.
Les jetons de réinitialisation sont hachés en SHA-256 en base.
On a aussi l'anti-force-brute et la blacklist, déjà vus avec Redis.
Et le tout repose sur Passport, avec deux stratégies : une pour le login, une pour protéger les routes.

### Injection SQL & XSS : défense en profondeur
Deux failles majeures, neutralisées à plusieurs niveaux.
L'injection SQL : Prisma génère des requêtes paramétrées, et on nettoie les entrées avec la ValidationPipe et les DTO.
Le XSS : React échappe le HTML par défaut - on n'utilise jamais dangerouslySetInnerHTML - Helmet ajoute une politique de sécurité de contenu, et surtout le cookie HttpOnly protège le jeton même en cas de XSS.
Le principe : si une couche cède, une autre protège.

### CSRF & contrôle d'accès
Ensuite, contrôler qui peut faire quoi.
Le CSRF est neutralisé par un CORS restrictif - une seule origine autorisée - et le cookie en SameSite Lax, qui empêche une requête forgée depuis un autre site.
Et le contrôle d'accès, le risque numéro un de l'OWASP, se joue à deux niveaux : par rôle, avec les guards, et par propriétaire - on vérifie que la ressource appartient bien à l'utilisateur.
À retenir : pour le CORS comme pour le cookie, c'est le serveur qui déclare la règle, et le navigateur qui l'applique.

### Tests : une stratégie ciblée
Pour les tests, une stratégie assumée : uniquement des tests unitaires - je ne vais pas afficher une fausse pyramide.
Deux cent onze tests au total : cent cinquante et un côté back avec Jest, soixante côté front avec Vitest.
Je les ai concentrés sur la logique critique - le checkout, le paiement, le panier - là où une régression coûte cher.
Ils tournent en intégration continue à chaque pull request, et bloquent la fusion.
À l'écran, un vrai test du panier, en arrange, act, assert.

### Jeu d'essai : le tunnel de vente
En complément des tests unitaires, un jeu d'essai fonctionnel sur le parcours critique.
Huit scénarios, du cas nominal aux cas d'erreur : le cumul de panier, l'ajustement au stock, l'expiration, l'accès interdit à la commande d'un autre, la double confirmation, le webhook réussi ou échoué, et la signature invalide.
Sur les huit, aucun écart entre le résultat attendu et le résultat obtenu.
Et ces scénarios sont rejoués en intégration continue.

### RGPD & cadre légal
Enfin la conformité RGPD.
L'arbitrage central : on ne peut pas simplement supprimer un compte, parce que les commandes servent de preuve comptable, à conserver plusieurs années.
La solution : anonymiser les données personnelles, en transaction, tout en gardant les commandes.
On implémente aussi le droit d'accès, avec un export des données en JSON.
Et un point simple : on n'a aucun traceur ni analytics, donc pas besoin de bannière cookies - le seul cookie, c'est l'authentification, strictement nécessaire.

---

## BLOC 5 - Déploiement & DevOps

### Intercalaire - Déploiement & DevOps
*(transition)* Passons de la machine locale à la production.

### Conteneurisation : un environnement reproductible
Tout l'environnement est conteneurisé avec Docker, ce qui règle le fameux « ça marche chez moi ».
En développement, sept services, avec rechargement à chaud.
En production, quatre services durcis : pas de ports exposés publiquement, des healthchecks, un redémarrage automatique.
Les images sont en multi-stage, sur Alpine, en utilisateur non-root - donc légères et avec une surface d'attaque réduite.
Les variables d'environnement sont validées au démarrage par Joi, et un Makefile d'une vingtaine de commandes simplifie tout ça.

### CI/CD : trois workflows GitHub Actions
La qualité est automatisée à chaque étape, avec trois workflows GitHub Actions calqués sur notre git flow.
Sur chaque pull request : lint, tests, et audit de sécurité des dépendances - c'est le garde-fou, il bloque la fusion.
Sur la branche d'intégration : une vraie base PostgreSQL, les migrations, le seed, et la couverture.
Et sur la branche principale : les tests, puis le déclenchement du déploiement, et une vérification de santé de l'application.
J'interprète évidemment les rapports générés.

### Mise en production : Hetzner + Coolify
Point dont je suis assez fier : j'ai réalisé le déploiement de bout en bout, moi-même.
Un VPS chez Hetzner, avec Coolify - une plateforme auto-hébergée, une sorte de Heroku maison, qui orchestre Docker.
Le flux : un push sur la branche principale déclenche les tests, puis un webhook vers Coolify qui reconstruit et déploie les conteneurs, le tout derrière Nginx en HTTPS.
J'ai aussi une procédure de sauvegarde de la base, avec pg_dump, validée en local.
En toute transparence, il me reste à automatiser ces sauvegardes.

---

## BLOC 6 - Conclusion

### Intercalaire - Conclusion
*(transition)* Prenons un peu de recul pour conclure.

### Perspectives & axes d'amélioration
Un MVP solide a été livré ; voici les axes que j'ai identifiés et priorisés pour la suite.
Côté tests : de l'intégration, du end-to-end, un seuil de couverture, et une authentification avec refresh token.
Côté fonctionnel : la carte interactive de plantation, un portail partenaires, le parrainage.
Et côté production : finaliser le webhook Stripe et ajouter du monitoring.
Le message, c'est qu'on sait ce qu'on ferait avec plus de temps - c'est une posture de priorisation.

### Bilan personnel
Sur le plan personnel, la boucle est bouclée : le fil rouge du départ, concevoir puis construire, aboutit à un projet mené de bout en bout.
Trois choses que je retiens : une vraie montée en compétence full-stack, de la conception à la production ; la rigueur et la méthode Agile ; et l'autonomie, avec ce déploiement réalisé seul.
Et pour tout dire, cette dynamique m'a mené au poste que j'occupe aujourd'hui : concepteur intégrateur IA au Groupe LDLC.

### Merci / clôture
Voilà, je vous remercie de votre attention, et je suis à votre disposition pour vos questions.
