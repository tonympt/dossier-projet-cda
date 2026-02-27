# État d'avancement - Construction du plan du dossier

## Où on en est

On est en train de définir le plan du dossier **section par section**, en discutant du contenu et des grandes lignes avant de rédiger. Les décisions sont validées ensemble au fil de la conversation.

## Sections validées

### ✅ Section 1 — Liste des compétences mises en œuvre
- Forme : tableau simple (CP / intitulé / couverte)
- Une demi-page max, sobre

### ✅ Section 2 — Cahier des charges / Expression des besoins
- Ce que le projet *demandait* (côté produit, pas technique)
- Contenu : présentation du projet, objectifs, cible utilisateurs, acteurs (3 rôles), périmètre MVP vs évolutions futures, user stories principales
- **Hors de cette section** : contraintes et livrables → section 5 / risques → section 4

### ✅ Section 3 — Contexte du projet (~0,5 page)
- Adaptation de "présentation de l'entreprise" pour un projet de formation
- Contenu : organisme de formation, GreenRoots projet fictif, rôle du candidat (Lead Dev Front + contribution étendue)
- **Hors de cette section** : organisation de l'équipe et pilotage → section 4
- Note : Tony a touché à tout malgré le rôle Lead Dev Front — à valoriser dans la rédaction

### ✅ Section 4 — Gestion de projet (~3-4 pages)
- Méthodologie Agile/Scrum, 4 sprints d'1 semaine (Sprint 0 à Sprint 3)
- Planning/suivi : Trello Kanban avec captures disponibles
- Cérémonies : daily meetings, rétrospectives, définition des objectifs
- Qualité : conventions communes documentées, guidelines frontend (Atomic Design), PRs + code review obligatoire, ESLint + Prettier
- Git flow : stratégie de branches + PRs focalisées, pas de merge sans review
- Risques : définis dans le CDC (section 2), simple référence ici — pas de suivi formel pendant le projet

## Section en cours

### 🔄 Section 5 — Spécifications fonctionnelles
On vient de commencer la discussion sur la **sous-section 5.1** (contraintes du projet et livrables attendus).

**Question ouverte :** y avait-il d'autres contraintes notables (techniques ou fonctionnelles) au-delà de :
- Tunnel de paiement simulé (pas Stripe réel au MVP)
- Compatibilité navigateurs (2 dernières versions majeures)
- Délai 4 semaines, équipe de 5
- Livrables : application déployée, documentation, dossier de projet

**Sous-sections restantes à discuter :**
- 5.2 Architecture logicielle (CP6)
- 5.3 Maquettes et enchaînement (CP5)
- 5.4 Modèle de données (CP7)
- 5.5 Diagramme de cas d'utilisation (CP5)
- 5.6 Diagrammes de séquence (CP5, CP6)

## Sections non encore abordées
- Section 6 — Spécifications techniques (CP1, CP6)
- Section 7 — Réalisations / extraits de code (CP2, CP3, CP8)
- Section 8 — Éléments de sécurité
- Section 9 — Plan de tests (CP9)
- Section 10 — Veille sécurité
- Annexes

## Décisions structurantes prises
- On suit le **plan "projet en entreprise"** du référentiel (adapté au contexte formation)
- **Section 2 ne chevauche pas la section 5** : le CDC = les besoins exprimés, les spécifications = la réponse technique
- Les **risques** sont mentionnés en section 4 par référence au CDC, pas développés
- Les **contraintes** sont en section 5, pas en section 2
- **Pas de DoD formel** — le code review obligatoire avant merge joue ce rôle en pratique

## Ressources disponibles
- Code source complet dans `projet-greenroots/`
- Schéma BDD : `projet-greenroots/app/backend/prisma/schema.prisma`
- Guidelines frontend : `guidelines-frontend.md`
- Conventions communes : `conventions-communes.docx`
- CDC initial : `Cahier des charges GreenRoots.docx`
- Référentiel officiel : `Référentiel Activités Compétences.pdf`
- Captures Trello : disponibles (à intégrer en section 4)
