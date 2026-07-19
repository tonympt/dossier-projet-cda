# Glossaire oral CDA - GreenRoots (Partie 2/2)

Suite de la partie 1 (gestion de projet, architecture, sécurité). Ici : base de données, frontend, paiement, tests, déploiement, RGPD, services. La dernière section explique comment écouter ces notes en podcast sur ton téléphone.

---

## 4. Base de données

### 4.1 Modélisation MERISE

**MERISE.** Une méthode française de conception de bases de données, en trois niveaux d'abstraction : le modèle conceptuel (MCD), le modèle logique (MLD), le modèle physique (MPD). On part du métier et on descend vers le technique. Si le jury creuse : « pourquoi trois niveaux ? » Réponse : pour séparer le quoi du comment ; le conceptuel décrit le métier sans se soucier de la technique, le physique dépend du moteur de base choisi ; ça rend la conception plus claire et plus portable.

**MCD (Modèle Conceptuel de Données).** La vue métier : les entités (par exemple Utilisateur, Commande, Arbre), leurs propriétés, et les associations entre elles avec des cardinalités. Indépendant de tout moteur de base. Si le jury creuse : « c'est quoi une cardinalité ? » Réponse : elle indique combien d'occurrences d'une entité participent à une association, par exemple « un utilisateur passe zéro à plusieurs commandes, une commande appartient à un et un seul utilisateur ». On la note souvent 0,n et 1,1.

**MLD (Modèle Logique de Données).** La traduction du MCD en tables relationnelles, en appliquant les règles de passage : chaque entité devient une table, les associations deviennent des clés étrangères ou des tables de liaison. Si le jury creuse : « comment passe-t-on une association plusieurs-à-plusieurs ? » Réponse : elle devient une table d'association portant les deux clés étrangères ; par exemple, une commande contient plusieurs arbres et un arbre est dans plusieurs commandes, ce qui donne la table OrderItem avec une contrainte d'unicité sur le couple commande-arbre.

**MPD (Modèle Physique de Données).** Le schéma réel implanté dans le moteur, ici PostgreSQL : les types exacts (entier, texte), les enums, les contraintes, les index. Chez moi il est matérialisé par le schéma Prisma. Piège à connaître : mon schéma n'a pas d'index explicites (@@index), seulement des contraintes d'unicité (@@unique). Si le jury creuse : « avez-vous des index pour la performance ? » Réponse honnête : les clés primaires et les contraintes uniques créent des index automatiquement ; je n'ai pas ajouté d'index supplémentaires, ce serait un axe d'optimisation si le volume augmentait. Ne prétends pas avoir des @@index, tu n'en as pas.

**Clé primaire, clé étrangère, contrainte d'unicité.** La clé primaire identifie de façon unique une ligne. La clé étrangère référence la clé primaire d'une autre table, garantissant l'intégrité référentielle. La contrainte d'unicité empêche les doublons sur une ou plusieurs colonnes. Si le jury creuse : « donnez un exemple d'unicité métier chez vous. » Réponse : sur OrderItem, l'unicité du couple commande-arbre évite d'avoir deux lignes pour le même arbre dans une commande (on incrémente la quantité à la place) ; sur OrderAddress, l'unicité du couple commande-type évite deux adresses de facturation sur une même commande.

**Enum.** Un type énuméré : une colonne ne peut prendre qu'une valeur parmi une liste fixe. Dans GreenRoots : le statut de commande (PENDING, CONFIRMED, PLANTED, CANCELLED), le statut de paiement (PENDING, PROCESSING, PAID, FAILED, REFUNDED), le type d'adresse (facturation, livraison). Si le jury creuse : « pourquoi un enum plutôt qu'un texte libre ? » Réponse : pour l'intégrité (impossible d'avoir un statut invalide) et la lisibilité ; ces enums sont le support des machines à états des commandes.

**Le prix stocké en centimes (type entier).** Les montants sont stockés en entiers, en centimes, pas en nombres à virgule. Si le jury creuse : « pourquoi pas un type décimal ? » Réponse : pour éviter les erreurs d'arrondi des flottants (0,1 + 0,2 ne fait pas exactement 0,3 en binaire) ; c'est aussi le format qu'attend Stripe, qui raisonne en plus petite unité monétaire.

**PostGIS.** Une extension de PostgreSQL qui ajoute des types et fonctions géographiques (coordonnées, distances, zones). Dans GreenRoots, la localisation des plantations utilise un type geometry. Si le jury creuse : « vous l'exploitez vraiment ? » Réponse honnête : la colonne géographique existe et prépare la future carte interactive de plantation ; l'exploitation cartographique complète est hors périmètre du MVP, mais le socle est posé.

**Séparation Tree et TreeStock.** L'arbre (son nom, son prix, sa description) et son stock sont dans deux tables séparées, reliées un-à-un. Si le jury creuse : « pourquoi séparer ? » Réponse : c'est une partition verticale ; le stock change très souvent (à chaque vente) alors que la fiche de l'arbre est stable ; les séparer isole les écritures fréquentes et clarifie les responsabilités.

**Migrations.** Des scripts versionnés qui font évoluer le schéma de la base de façon reproductible et traçable. GreenRoots en compte vingt-quatre. Si le jury creuse : « à quoi ça sert par rapport à modifier la base à la main ? » Réponse : chaque changement est enregistré, rejouable à l'identique sur n'importe quel environnement (dev, CI, prod), et réversible dans l'historique Git ; c'est ce qui rend le schéma reproductible.

### 4.2 Accès aux données et transactions

**ORM (Object-Relational Mapping).** Une couche qui fait correspondre les tables de la base à des objets du code, pour manipuler les données sans écrire de SQL à la main. Prisma est mon ORM. Si le jury creuse : « avantage et inconvénient d'un ORM ? » Réponse : avantage, la productivité et la sécurité (requêtes paramétrées, typage) ; inconvénient, on peut générer des requêtes non optimales sans s'en rendre compte, il faut savoir regarder le SQL produit.

**Prisma.** Mon ORM, en TypeScript, qui apporte un typage de bout en bout, des migrations et des requêtes paramétrées. Si le jury creuse : « Prisma génère quoi comme SQL pour un include imbriqué ? » Réponse : des jointures (JOIN) ; par exemple récupérer une commande avec ses articles, leurs arbres et le stock se traduit par des JOIN entre les tables commandes, articles, arbres et stock. Sache lire l'équivalence, c'est une attente explicite du jury.

**Requête paramétrée.** Une requête où les valeurs sont envoyées séparément de l'instruction SQL, via des marqueurs (comme $1, $2), et ne sont jamais interprétées comme du code. C'est la parade fondamentale contre l'injection SQL. Si le jury creuse : « la différence avec la concaténation ? » Réponse : concaténer une entrée utilisateur dans une chaîne SQL permet l'injection ; la paramétrer la traite comme une simple donnée, l'injection devient impossible.

**Transaction et atomicité.** Une transaction regroupe plusieurs opérations en un tout indivisible : soit tout réussit, soit tout est annulé (rollback), jamais un état intermédiaire. En SQL, cela s'écrit BEGIN au début et COMMIT à la fin (ou ROLLBACK en cas d'erreur) ; en Prisma, c'est la fonction transaction. Dans GreenRoots, la réservation de stock décrémente le stock et met à jour la commande dans une même transaction. Si le jury creuse : « que se passe-t-il si le serveur plante au milieu ? » Réponse : la transaction n'est pas validée, donc annulée automatiquement ; le stock n'est pas décrémenté à moitié, l'intégrité est préservée.

**ACID.** Les quatre garanties d'une transaction : Atomicité (tout ou rien), Cohérence (on passe d'un état valide à un autre), Isolation (les transactions concurrentes ne se marchent pas dessus), Durabilité (une fois validée, c'est écrit durablement). Si le jury creuse : « expliquez l'isolation dans votre cas. » Réponse : deux clients qui achètent le dernier arbre en même temps ne peuvent pas décrémenter le stock deux fois ; le décrément atomique en base et la transaction garantissent qu'un seul passe, l'autre voit le stock insuffisant.

**Idempotence.** Une opération est idempotente si l'exécuter une ou plusieurs fois produit le même résultat. Crucial pour le paiement. Dans GreenRoots : si l'utilisateur double-clique sur « Payer », un drapeau (stockReservedAt, plus le statut PROCESSING) évite de réserver le stock deux fois ; et le webhook Stripe, s'il est rejoué, ne repasse pas une commande déjà payée. Si le jury creuse : « donnez un contre-exemple non idempotent. » Réponse : un simple « décrémente le stock de 1 » rejoué deux fois retirerait 2 ; c'est pourquoi je garde une marque d'état pour ne rejouer que si nécessaire.

**Race condition (situation de compétition).** Un bug qui survient quand deux opérations concurrentes accèdent à la même ressource sans coordination, produisant un résultat incohérent (par exemple sur-vendre un stock). Parade : l'opération atomique en base et la transaction. Si le jury creuse : « comment évitez-vous la sur-vente ? » Réponse : le décrément se fait en une seule instruction SQL atomique (quantité égale quantité moins X), et la lecture-écriture est dans une transaction, donc deux commandes simultanées ne peuvent pas passer sous le stock réel.

### 4.3 NoSQL et Redis

**NoSQL.** Des bases qui ne suivent pas le modèle relationnel : clé-valeur, document, colonne, graphe. On les choisit pour la vitesse, la simplicité ou la mise à l'échelle sur certains usages. Redis est une base NoSQL clé-valeur. Si le jury creuse : « SQL ou NoSQL, comment choisir ? » Réponse : SQL pour des données structurées et des relations fortes avec intégrité (mes commandes) ; NoSQL clé-valeur pour du cache et des données éphémères à accès ultra-rapide (Redis) ; les deux cohabitent, chacun sur son usage.

**Redis.** Une base clé-valeur qui vit en mémoire vive, donc extrêmement rapide (de l'ordre de la fraction de milliseconde). Trois usages dans GreenRoots : le cache, l'anti-force-brute, et la liste noire des JWT. Si le jury creuse : « pourquoi en mémoire, ce n'est pas risqué ? » Réponse : la volatilité est acceptable ici parce que ces données sont soit reconstructibles (le cache se recharge depuis la base), soit temporaires (compteurs, jetons expirants) ; je n'y mets rien de critique et non reconstructible.

**Cache-aside (cache à côté).** Un patron de cache : on lit d'abord le cache ; en cas d'absence (miss), on lit la base, puis on remplit le cache pour la prochaine fois. Dans GreenRoots, les pays sont mis en cache vingt-quatre heures, les catégories une heure avec invalidation à l'écriture. Si le jury creuse : « qu'est-ce que l'invalidation ? » Réponse : quand une donnée change en base (une catégorie modifiée), on efface son entrée de cache pour ne pas servir une valeur périmée ; le cache se reconstruit à la lecture suivante.

**TTL (Time To Live, durée de vie).** La durée après laquelle une entrée de cache expire automatiquement. Si le jury creuse : « comment choisit-on un TTL ? » Réponse : selon la fraîcheur nécessaire ; les pays changent quasi jamais, donc vingt-quatre heures ; les catégories peuvent bouger, donc une heure, complétée par l'invalidation immédiate à l'écriture.

**Dégradation gracieuse et fail-open.** Si Redis tombe, l'application continue de fonctionner en se rabattant sur la base de données ; le cache accélère mais n'est pas critique. Fail-open signifie qu'en cas de panne du garde-fou, on laisse passer plutôt que de tout bloquer. Si le jury creuse : « fail-open, n'est-ce pas un risque de sécurité ? » Réponse : c'est un arbitrage disponibilité contre confidentialité ; pour l'anti-force-brute, je préfère que le login reste disponible si Redis tombe plutôt que de bloquer tous les utilisateurs ; c'est un choix conscient, pas un oubli.

**SPOF (Single Point Of Failure, point de défaillance unique).** Un composant dont la panne fait tomber tout le système. Justement, grâce à la dégradation gracieuse, Redis n'est pas un SPOF chez moi. Si le jury creuse : « quels SPOF reste-t-il ? » Réponse : la base PostgreSQL en est un (single instance) ; en production sérieuse on ajouterait de la réplication ; c'est un axe identifié.

**Machine à états (state machine).** Un modèle où un objet ne peut passer que par des transitions autorisées entre des états définis. Dans GreenRoots, le statut de commande est une vraie machine à états : une constante déclare les transitions permises (par exemple de PENDING vers CANCELLED, de CONFIRMED vers PLANTED), et un garde refuse toute transition non listée. Nuance à assumer : seul le statut de commande a cette table de transitions explicite ; le statut de paiement, lui, est un enum piloté par le webhook Stripe, sans table de transitions formelle. Si le jury creuse : « avez-vous deux machines à états ? » Réponse : une seule formelle, celle du statut de commande ; le paiement suit des transitions implicites dictées par Stripe. Ne survends pas.

---

## 5. Frontend

**React.** Une bibliothèque JavaScript pour construire des interfaces à base de composants réutilisables, avec un rendu déclaratif. Si le jury creuse : « déclaratif, ça veut dire quoi ? » Réponse : je décris à quoi l'interface doit ressembler pour un état donné, et React se charge de mettre à jour le DOM ; je ne manipule pas le DOM à la main.

**Vite.** Un outil de build et un serveur de développement très rapide pour les applications front modernes. Si le jury creuse : « pourquoi Vite et pas Webpack ? » Réponse : démarrage quasi instantané et rechargement à chaud rapide en développement, build optimisé en production ; c'est plus léger et moderne que Webpack, et ça sert l'éco-conception.

**Les trois natures d'état.** Un point fort à mettre en avant : je ne mets pas tout dans un état global, je sépare l'état selon sa nature. L'état local de l'interface (le panier) avec Zustand ; l'état serveur (le stock, source de vérité) avec TanStack Query ; l'état de formulaire avec React Hook Form et Zod. Si le jury creuse : « pourquoi cette séparation ? » Réponse : chaque type d'état a des besoins différents (persistance, synchronisation, validation) ; un seul store global mélangerait tout et deviendrait ingérable ; le bon outil pour chaque besoin.

**Zustand.** Une petite bibliothèque de gestion d'état global côté client. Le panier y est stocké, avec persistance dans le localStorage et une expiration de vingt-quatre heures. Nuance : ce n'est pas un TTL natif, c'est une expiration maison basée sur la date de création. Si le jury creuse : « pourquoi Zustand plutôt que Redux ? » Réponse : Redux est puissant mais verbeux ; Zustand offre l'essentiel avec beaucoup moins de code, adapté à la taille du projet.

**TanStack Query.** Une bibliothèque qui gère les données venant du serveur : cache, rafraîchissement, états de chargement et d'erreur. Le stock est rafraîchi par interrogation régulière (polling) toutes les cinq minutes et au retour sur l'onglet. Si le jury creuse : « c'est quoi staleTime ? » Réponse : la durée pendant laquelle une donnée est considérée comme fraîche et n'est pas refetchée ; au-delà, TanStack Query la rafraîchit ; les pays ont un staleTime d'une heure car ils changent peu.

**syncWithStock (réconciliation panier - stock).** Le panier est persisté localement, il peut donc devenir périmé ; à l'ouverture, on le réconcilie avec le stock serveur. Trois cas : produit introuvable (arbre supprimé côté serveur), l'article est conservé dans le panier mais masqué à l'affichage ; stock suffisant, inchangé ; stock insuffisant, la quantité est ajustée et une fenêtre prévient l'utilisateur. Si le jury creuse : « pourquoi ne pas juste supprimer l'article introuvable ? » Réponse : choix produit, on préfère le garder et informer plutôt que de vider silencieusement le panier de l'utilisateur.

**Polling.** Interroger le serveur à intervalle régulier pour récupérer des données à jour. On l'utilise pour le stock et pour attendre la confirmation de paiement. Si le jury creuse : « pourquoi pas du temps réel avec WebSocket ou SSE ? » Réponse : le polling est simple et suffisant pour ces besoins ; les Server-Sent Events seraient plus efficaces pour du temps réel, c'est un axe d'amélioration identifié dans les perspectives.

**Responsive et useMediaQuery.** Le responsive adapte l'interface à la taille de l'écran. Point fort : chez moi ce n'est pas que du CSS, le hook useMediaQuery fait carrément *changer de composant* - un tableau des commandes sur ordinateur devient des cartes sur mobile. Si le jury creuse : « différence avec du responsive CSS classique ? » Réponse : le CSS réarrange le même HTML ; ici, sous un certain seuil, je rends un composant différent, mieux adapté au tactile ; c'est un choix d'expérience utilisateur.

**RGAA, WCAG, ARIA.** Le RGAA est le référentiel français d'accessibilité, adossé au standard international WCAG. ARIA est un ensemble d'attributs HTML qui décrivent le rôle et l'état des éléments aux technologies d'assistance (lecteurs d'écran). Dans GreenRoots, les composants shadcn/ui apportent l'ARIA nativement, et mon champ de quantité expose un rôle spinbutton avec les valeurs min, max et courante. Si le jury creuse : « votre score d'accessibilité Lighthouse de 96 veut-il dire conforme RGAA ? » Réponse honnête : non ; Lighthouse teste automatiquement environ un tiers des critères ; une conformité RGAA complète demande un audit manuel ; 96 est un bon indicateur, pas une certification.

**shadcn/ui et Radix.** shadcn/ui est une collection de composants d'interface que l'on copie dans son projet, construits sur Radix, une bibliothèque de primitives accessibles et non stylées. Si le jury creuse : « pourquoi ce choix cochait toutes vos cases ? » Réponse : Radix apporte l'accessibilité, on importe seulement ce qu'on utilise (éco-conception), et c'est typé ; c'est le choix qui réunit mes trois fils directeurs.

**Tailwind CSS.** Un framework CSS utilitaire : on style directement dans le HTML avec des petites classes. Le build purge les classes inutilisées. Si le jury creuse : « ça ne rend pas le HTML illisible ? » Réponse : c'est le reproche courant ; en pratique, combiné à des composants, ça évite des feuilles de style énormes et ça garde le style au plus près du markup ; la purge réduit fortement le CSS final.

**Type-safety et z.infer.** La sûreté de typage garantit à la compilation qu'on manipule les bons types. Avec Zod, je définis un schéma de validation et j'en dérive automatiquement le type TypeScript avec z.infer, si bien que le schéma et le type ne peuvent jamais diverger. Si le jury creuse : « quel intérêt concret ? » Réponse : une seule source de vérité pour les règles et le type ; si je modifie le schéma, le type suit, et le compilateur m'alerte partout où c'est incohérent.

---

## 6. Paiement (Stripe)

**Stripe.** Une plateforme de paiement en ligne. Point clé de sécurité : aucune donnée bancaire ne transite ni n'est stockée chez moi, c'est Stripe qui les gère ; je ne manipule qu'un identifiant de paiement. Si le jury creuse : « et la conformité PCI-DSS ? » Réponse : en déléguant la saisie de la carte à Stripe, je sors mon serveur du périmètre PCI-DSS le plus lourd ; c'est justement l'intérêt d'externaliser le paiement.

**PaymentIntent.** L'objet Stripe qui représente une tentative de paiement et suit son cycle de vie. Chaque commande a son PaymentIntent. Si le jury creuse : « pourquoi un PaymentIntent et pas un simple paiement ? » Réponse : il gère les étapes (authentification forte 3D Secure, échec, nouvelle tentative) et l'état du paiement, ce qui colle aux exigences européennes DSP2.

**Le chemin du PaymentIntent, de bout en bout (front, back, Stripe).** À savoir raconter d'une traite. Un, le front poste sur la route d'initialisation du checkout. Deux, le back, dans une transaction, crée le PaymentIntent chez Stripe en lui envoyant le montant, la devise et, dans les métadonnées, l'identifiant de la commande ; il stocke l'identifiant du PaymentIntent sur la commande et récupère le client_secret. Trois, le back renvoie au front l'identifiant de commande et le client_secret. Quatre, le front injecte ce client_secret dans les composants Stripe (la bibliothèque Elements) ; au clic sur payer, il appelle d'abord la confirmation côté back qui réserve le stock de façon atomique, puis il lance le paiement dans le navigateur avec le client_secret (la carte est saisie et envoyée directement à Stripe, jamais à mon serveur). Cinq, Stripe encaisse puis notifie mon back par webhook (événement paiement réussi) ; c'est le webhook, et lui seul, qui fait passer la commande en payée et confirmée, génère la facture et envoie l'email. Point clé à marteler : le résultat du paiement n'est jamais décidé par le navigateur, le front attend la confirmation du serveur (issue du webhook) par interrogation régulière. Les quatre secrets Stripe à ne pas confondre : la clé publiable (front, publique, pour initialiser Stripe.js), la clé secrète (back, pour appeler l'API Stripe), le secret de webhook (back, pour vérifier la signature HMAC des notifications), et le client_secret (propre à chaque PaymentIntent, envoyé au front pour confirmer ce paiement précis).

**Webhook.** Une notification que Stripe envoie à mon serveur quand un événement se produit (paiement réussi, échoué, annulé). C'est la source de vérité du paiement. Si le jury creuse : « pourquoi la source de vérité est le webhook et pas la réponse du navigateur ? » Réponse : le navigateur est manipulable, on ne peut pas lui faire confiance pour valider un paiement ; le webhook vient directement de Stripe, serveur à serveur ; mon front attend d'ailleurs la confirmation serveur par polling.

**Signature de webhook (constructEvent).** Chaque webhook Stripe est signé en HMAC avec un secret partagé. Mon serveur vérifie cette signature avant tout traitement ; une signature invalide est rejetée avec une erreur 400. Il faut le corps brut de la requête (rawBody), non transformé, pour vérifier la signature. Si le jury creuse : « pourquoi le corps brut ? » Réponse : la signature est calculée sur les octets exacts envoyés ; si un middleware reformate le JSON, la signature ne correspond plus ; d'où la nécessité du corps brut.

**Anti-spoofing (anti-usurpation).** Vérifier qu'un webhook correspond bien à la bonne commande : je compare l'identifiant de paiement reçu à celui enregistré sur la commande. Si le jury creuse : « quel scénario ça bloque ? » Réponse : une tentative de faire confirmer une commande avec l'identifiant de paiement d'une autre ; s'ils ne correspondent pas, on ignore.

**Cron de réconciliation.** Une tâche planifiée (toutes les heures) qui rattrape les webhooks éventuellement perdus : elle réinterroge Stripe pour les commandes bloquées en cours de traitement et les finalise si le paiement a réussi. Si le jury creuse : « pourquoi en avoir besoin si le webhook marche ? » Réponse : un webhook peut se perdre (panne réseau, redémarrage) ; le cron est un filet de sécurité pour la disponibilité et l'intégrité, il évite qu'une commande payée reste bloquée. C'est le « D » de DICP.

**Mode test Stripe et carte 4242.** Stripe a un mode test qui simule des paiements sans argent réel, avec des numéros de carte de test comme 4242 4242 4242 4242. C'est ce que j'utilise pour la démo. Si le jury creuse : « la démo est un vrai paiement ? » Réponse : c'est un vrai flux technique de bout en bout, mais en mode test Stripe ; le mécanisme est identique en production, seule la clé change.

---

## 7. Tests

**Test unitaire.** Un test qui vérifie une petite unité de code isolée (une fonction, une méthode). GreenRoots en compte deux cent onze : cent cinquante et un côté back avec Jest, soixante côté front avec Vitest et Testing Library. Si le jury creuse : « pourquoi surtout de l'unitaire ? » Réponse : j'ai ciblé la logique critique (panier, checkout, paiement) où une régression coûte cher ; c'est un choix assumé, les tests d'intégration et de bout en bout sont dans mes perspectives.

**Jest, Vitest, Testing Library.** Jest est le lanceur de tests côté back ; Vitest son équivalent rapide côté front (aligné avec Vite) ; Testing Library teste les composants du point de vue de l'utilisateur (ce qu'il voit et fait), pas les détails internes. Si le jury creuse : « pourquoi tester du point de vue utilisateur ? » Réponse : ça rend les tests robustes au refactoring ; tant que le comportement visible ne change pas, le test passe, même si j'ai réécrit l'intérieur.

**Arrange, Act, Assert.** La structure d'un bon test : préparer le contexte, exécuter l'action, vérifier le résultat. Mon exemple : j'ajoute deux fois le même arbre au panier (arrange et act), puis je vérifie qu'il n'y a qu'une ligne avec la quantité cumulée (assert). Si le jury creuse : « c'est quoi un bon test ? » Réponse : rapide, isolé, déterministe, et qui teste un seul comportement clair.

**Mock.** Un faux objet qui remplace une dépendance réelle dans un test, pour isoler l'unité testée. Par exemple, un faux Prisma pour tester un service sans vraie base. Si le jury creuse : « mock ou vraie base ? » Réponse : mock en test unitaire pour la rapidité et l'isolation ; vraie base en test d'intégration (ma CI en lance une), pour vérifier les vraies requêtes.

**Couverture (coverage).** Le pourcentage de code exécuté par les tests. La CI la mesure sur la branche d'intégration. Si le jury creuse : « visez-vous 100 % ? » Réponse : non, viser 100 % est contre-productif ; je privilégie la couverture de la logique critique ; un seuil minimal en CI est un axe d'amélioration.

**Jeu d'essai.** Un ensemble de scénarios fonctionnels avec résultat attendu et résultat obtenu, pour valider un parcours de bout en bout. Mon jeu d'essai couvre le tunnel de vente : huit cas du nominal (ajout, paiement réussi) aux cas d'erreur (stock insuffisant, accès interdit, double confirmation, signature invalide). Si le jury creuse : « différence avec les tests unitaires ? » Réponse : le test unitaire vérifie une brique de code ; le jeu d'essai valide un comportement fonctionnel complet, tel que l'utilisateur ou le métier l'attend ; les deux sont demandés par le référentiel.

**Tests d'intégration et de bout en bout (E2E).** L'intégration teste plusieurs composants ensemble (par exemple l'API avec une vraie base) ; le bout en bout simule un utilisateur réel dans un navigateur, avec des outils comme Cypress ou Playwright. Si le jury creuse : « pourquoi ne pas les avoir faits ? » Réponse : arbitrage de temps sur un projet de formation ; je les ai identifiés et priorisés dans les perspectives, en connaissant leur valeur.

---

## 8. Déploiement et DevOps

**Docker et conteneur.** Docker empaquette une application et tout ce dont elle a besoin dans un conteneur : une unité isolée, légère et reproductible, qui tourne à l'identique partout. Si le jury creuse : « conteneur ou machine virtuelle ? » Réponse : une VM embarque un système d'exploitation complet, c'est lourd ; un conteneur partage le noyau de l'hôte et n'embarque que l'application, c'est bien plus léger et rapide à démarrer.

**Image Docker.** Le modèle figé à partir duquel on lance des conteneurs. Une image se construit à partir d'un Dockerfile. Si le jury creuse : « différence image et conteneur ? » Réponse : l'image est le gabarit (la classe), le conteneur est une instance en cours d'exécution (l'objet) ; d'une image on lance plusieurs conteneurs.

**Docker Compose.** Un outil pour décrire et lancer plusieurs conteneurs ensemble via un fichier. GreenRoots a un environnement de développement à sept services (front, back, PostGIS, Redis, Nginx, l'outil Stripe en ligne de commande, Prisma Studio) et un environnement de production durci à quatre services. Si le jury creuse : « pourquoi moins de services en production ? » Réponse : les outils de développement (Prisma Studio, le tunnel Stripe) et le hot reload n'ont rien à faire en production ; on réduit la surface d'attaque et le poids.

**Multi-stage build, Alpine, non-root.** Le multi-stage sépare la construction de l'image finale : on compile dans une première étape, et l'image finale ne garde que le nécessaire, donc légère. Alpine est une distribution Linux minimale. Non-root signifie que le conteneur ne tourne pas en administrateur. Si le jury creuse : « pourquoi non-root ? » Réponse : si un attaquant compromet le conteneur, il n'a pas les pleins pouvoirs ; c'est le principe du moindre privilège appliqué aux conteneurs.

**Healthcheck.** Une sonde qui vérifie régulièrement que le service répond, via une route de santé (chez moi /api/health). Si le jury creuse : « à quoi ça sert ? » Réponse : l'orchestrateur sait si le conteneur est réellement opérationnel, pas juste démarré ; il peut le redémarrer s'il ne répond plus, ce qui sert la disponibilité.

**Nginx gzip et cache.** En production, Nginx compresse les fichiers (gzip) pour réduire leur poids et met en cache les fichiers statiques longtemps (un an), avec un nom de fichier qui change à chaque build. Si le jury creuse : « comment invalidez-vous un cache d'un an ? » Réponse : par le hash dans le nom de fichier ; à chaque build le nom change, donc le navigateur télécharge la nouvelle version tout en gardant l'ancienne en cache tant qu'elle ne change pas.

**Joi (validation des variables d'environnement).** Au démarrage, Joi valide que toutes les variables d'environnement nécessaires sont présentes et correctes (par exemple, le secret JWT doit faire au moins trente-deux caractères), sinon l'application refuse de démarrer. Si le jury creuse : « pourquoi valider au démarrage ? » Réponse : principe du fail-fast ; mieux vaut planter tout de suite avec un message clair qu'au milieu d'une requête en production à cause d'une variable oubliée.

**CI/CD (intégration et déploiement continus).** L'intégration continue teste automatiquement chaque changement ; le déploiement continu automatise la mise en production. GreenRoots a trois workflows GitHub Actions calqués sur le git flow. Si le jury creuse : « différence CI et CD ? » Réponse : la CI, c'est vérifier automatiquement que le code s'intègre sans casse (lint, tests) ; le CD, c'est livrer automatiquement en production une fois la CI verte.

**GitHub Actions, workflow, job.** GitHub Actions est le moteur d'automatisation intégré à GitHub. Un workflow est un pipeline déclenché par un événement (une pull request, un push) ; il contient des jobs (étapes). Mes trois workflows : sur chaque pull request, lint plus tests plus audit de sécurité, c'est le garde-fou qui bloque la fusion ; sur la branche d'intégration, une suite étendue avec une vraie base PostgreSQL, migrations, seed et couverture ; sur la branche principale, les tests puis le déclenchement du déploiement et la vérification de santé. Si le jury creuse : « que se passe-t-il si un test échoue en pull request ? » Réponse : la fusion est bloquée ; c'est le garde-fou qualité.

**Lint, ESLint, Prettier.** Le lint analyse le code pour détecter erreurs et mauvaises pratiques ; ESLint est le linter JavaScript/TypeScript, Prettier formate automatiquement le code. Si le jury creuse : « lint et formatage, c'est pareil ? » Réponse : non ; ESLint traque les problèmes de qualité et de bugs potentiels ; Prettier ne s'occupe que de la mise en forme (indentation, guillemets) ; les deux tournent en CI.

**npm audit.** Une commande qui vérifie si les dépendances du projet ont des vulnérabilités connues. En CI, je bloque sur les failles critiques (niveau critical). Si le jury creuse : « pourquoi seulement les critiques ? » Réponse : choix assumé pour ne pas bloquer sur du bruit de faible gravité ; les vulnérabilités critiques, elles, arrêtent la chaîne ; on interprète le rapport et on traite au cas par cas.

**Coolify, PaaS, VPS, Hetzner.** Un VPS est un serveur privé virtuel loué (le mien est chez Hetzner). Coolify est une plateforme (un PaaS auto-hébergé, une sorte de Heroku maison) qui orchestre Docker et gère les déploiements. Point fort à valoriser : j'ai réalisé ce déploiement de bout en bout moi-même. Si le jury creuse : « pourquoi auto-héberger plutôt qu'un service managé ? » Réponse : maîtrise complète de la chaîne et coût, dans une démarche d'apprentissage ; en contrepartie j'assume l'exploitation, d'où le monitoring en perspective.

**HTTPS et sslip.io.** HTTPS chiffre les échanges entre le navigateur et le serveur. sslip.io est un service qui fournit un nom de domaine à partir d'une adresse IP, pratique pour obtenir un certificat HTTPS sans acheter de domaine. Si le jury creuse : « pourquoi sslip.io ? » Réponse : c'est un projet de formation sans nom de domaine acheté ; sslip.io permet un HTTPS valide rapidement ; en vrai projet on prendrait un domaine dédié.

**pg_dump et pg_restore (sauvegarde et restauration).** pg_dump exporte une base PostgreSQL dans un fichier ; pg_restore la réimporte. J'ai une procédure documentée, validée en local. Nuance honnête : l'automatisation planifiée de ces sauvegardes reste à finaliser. Si le jury creuse : « votre sauvegarde est-elle automatisée en production ? » Réponse : la procédure est écrite et testée en local ; l'automatisation par tâche planifiée est le prochain pas, je l'assume dans les perspectives.

**Makefile.** Un fichier qui regroupe des commandes sous des noms courts (par exemple « make dev » pour tout lancer). GreenRoots en a une vingtaine. Si le jury creuse : « pourquoi un Makefile ? » Réponse : simplifier et documenter les commandes courantes ; un nouvel arrivant lance le projet en une commande, c'est de l'onboarding et de la reproductibilité.

---

## 9. RGPD et cadre légal

**RGPD.** Le Règlement Général sur la Protection des Données, le cadre européen sur les données personnelles. Il donne des droits aux personnes et des obligations aux traitements. Si le jury creuse : « quels droits avez-vous implémentés ? » Réponse : le droit d'accès (export des données), le droit à l'effacement (suppression de compte par anonymisation), et la minimisation (on ne renvoie jamais les champs sensibles).

**Droit d'accès (article 15).** L'utilisateur peut obtenir une copie de ses données. Chez moi, une route d'export renvoie un fichier JSON structuré : profil, commandes, adresses (pas le panier). Si le jury creuse : « accès ou portabilité ? » Réponse : la route s'appelle export au titre de l'accès (article 15) ; comme le format JSON est structuré et réutilisable, une lecture au titre de la portabilité (article 20) s'appliquerait aussi.

**Droit à l'effacement (article 17) et anonymisation.** L'utilisateur peut demander la suppression de ses données. Mais on ne peut pas tout supprimer, car les commandes servent de preuve comptable. La solution : anonymiser les données personnelles (le nom devient « Utilisateur supprimé », l'email un email neutre, l'adresse est effacée) puis supprimer le compte, le tout dans une transaction atomique ; les commandes, elles, sont conservées mais rattachées à un utilisateur anonyme. Si le jury creuse : « anonymisation ou pseudonymisation ? » Réponse : anonymisation, car le lien vers la personne réelle est irréversiblement rompu ; la pseudonymisation garderait une table de correspondance permettant de ré-identifier, ce n'est pas mon cas.

**Obligation de conservation.** Certaines données (factures, commandes) doivent être conservées un certain temps pour des raisons comptables et fiscales, généralement six ans. C'est ce qui justifie l'anonymisation plutôt que la suppression pure. Piège juridique à éviter : je dis « obligation légale de conservation, six ans » sans citer un article précis qui pourrait être faux ; ne cite pas un numéro d'article dont tu n'es pas sûr. Si le jury creuse : « le droit à l'effacement ne prime-t-il pas ? » Réponse : non, le RGPD prévoit des exceptions quand une obligation légale impose la conservation ; on concilie les deux en anonymisant les données personnelles tout en gardant les documents comptables.

**Minimisation et safeSelect.** Le principe de minimisation dit de ne traiter que les données nécessaires. Chez moi, une fonction safeSelect exclut par défaut le mot de passe et les jetons hachés de toutes les réponses de l'API. Si le jury creuse : « c'est de la minimisation ou de la sécurité ? » Réponse : les deux se rejoignent ; ne jamais renvoyer un champ sensible protège la confidentialité et respecte la minimisation en sortie.

**Cookies et ePrivacy.** La directive ePrivacy impose une bannière de consentement pour les cookies non essentiels (traceurs, analytics). Point fort : GreenRoots n'a aucun traceur ni analytics, le seul cookie est celui d'authentification, strictement nécessaire, donc exempté de bannière. Si le jury creuse : « pourquoi pas de bannière cookies ? » Réponse : parce que le seul cookie est fonctionnel et strictement nécessaire (l'authentification) ; la loi n'exige de consentement que pour les cookies non essentiels, que je n'ai pas.

**Facture et numérotation séquentielle.** Chaque paiement réussi génère une facture avec un numéro séquentiel unique, au format FA, année, numéro sur six chiffres, généré dans une transaction pour garantir l'unicité et l'absence de trou. Si le jury creuse : « pourquoi une numérotation séquentielle sans trou ? » Réponse : c'est une exigence légale de facturation ; la générer en transaction évite les doublons et les sauts, même en cas d'accès concurrents ; c'est le « P » de DICP, la preuve.

---

## 10. Services externes et divers

**Cloudinary.** Un service de gestion et d'optimisation d'images dans le cloud. Les images des arbres y sont hébergées et servies optimisées. Si le jury creuse : « pourquoi externaliser les images ? » Réponse : optimisation automatique (formats, tailles, compression), distribution rapide, et on n'encombre pas notre serveur ; ça sert la performance et l'éco-conception.

**Resend.** Un service d'envoi d'emails transactionnels (confirmation de commande, réinitialisation de mot de passe). Si le jury creuse : « pourquoi un service tiers pour les emails ? » Réponse : la délivrabilité ; envoyer des emails depuis son propre serveur finit souvent en spam ; un service spécialisé gère la réputation d'envoi.

**Cron (tâche planifiée).** Une tâche qui s'exécute automatiquement à intervalle régulier. Chez moi, le cron de réconciliation des paiements tourne toutes les heures. Si le jury creuse : « où tourne ce cron ? » Réponse : dans le back NestJS, via son planificateur intégré ; il n'a pas besoin d'un service externe.

**Swagger / OpenAPI.** Une spécification standard pour documenter une API REST, avec une interface web pour explorer et tester les routes. Si le jury creuse : « votre API est-elle documentée ? » Réponse : oui, via OpenAPI/Swagger, ce qui sert de contrat entre le front et le back et facilite les tests manuels.

---

## Comment écouter ce glossaire en podcast sur ton téléphone

Ces deux fichiers sont du texte : n'importe quel outil de synthèse vocale (text-to-speech) peut les lire à voix haute. Le plus simple sur ton Pixel, avec de vraies voix naturelles :

**Option 1 - Microsoft Edge (gratuit, voix françaises très naturelles, recommandé).**
1. Installe l'application Edge sur le Pixel.
2. Transfère-toi le fichier en PDF ou ouvre-le : le plus simple est que je te génère une version PDF (demande-la-moi), tu l'ouvres dans Edge, ou tu colles le texte dans une page.
3. Menu (les trois points) puis « Lecture à voix haute ». Tu choisis la voix (prends une voix française « Naturel »), la vitesse, et ça se met en lecture, y compris écran éteint.

**Option 2 - @Voice Aloud Reader (application Android dédiée).**
1. Installe « @Voice Aloud Reader » depuis le Play Store.
2. Importe le fichier texte ou PDF (l'appli lit TXT, PDF, HTML, ePub).
3. Elle lit avec la synthèse vocale du téléphone, en tâche de fond, avec une file de lecture, comme un vrai podcast.

**Option 3 - Google Play Livres.**
1. Envoie-toi le fichier en PDF ou ePub (via Google Drive ou email).
2. Importe-le dans Play Livres (« Importer des fichiers »).
3. Ouvre le livre, menu, « Lire à voix haute » (lecture automatique). Pratique pour la lecture en fond.

**Option 4 - Speechify ou NaturalReader.** Applications spécialisées, voix très naturelles, gratuites avec limites. Tu colles le texte ou importes le fichier, et tu écoutes comme un podcast.

**Conseil de format.** Les symboles Markdown (les dièses des titres, les tirets) peuvent être lus bizarrement par certaines voix. Si tu veux une écoute vraiment fluide, je peux te générer une **version « script audio »** : le même contenu en texte continu, sans symboles, découpé en chapitres, prêt à être lu d'une traite. Dis-le-moi et je te la prépare (et je peux aussi te sortir un PDF propre des deux parties).
