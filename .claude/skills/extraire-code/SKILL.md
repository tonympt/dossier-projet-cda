---
name: extraire-code
description: Extraire et formater des extraits de code du projet GreenRoots pour le dossier CDA. Recherche dans le code source et propose des extraits annotés.
argument-hint: "[thème ou fonctionnalité]"
---

# Skill : Extraire et formater des extraits de code

## Objectif
Rechercher et extraire des extraits de code pertinents sur le thème **$ARGUMENTS** dans le projet GreenRoots, formatés pour insertion dans le dossier CDA.

## Étapes

1. **Rechercher les fichiers pertinents** dans `projet-greenroots/app/` (backend et frontend) en lien avec le thème demandé. Utiliser la recherche par nom de fichier et par contenu.

2. **Identifier les portions significatives** : ne pas extraire des fichiers entiers, mais les parties qui illustrent le mieux le propos technique (pattern d'architecture, logique métier, sécurité, etc.).

3. **Formater chaque extrait** :
   - Bloc de code avec indication du langage (```typescript, ```tsx, etc.)
   - Chemin source du fichier (relatif à `projet-greenroots/`)
   - Commentaire explicatif au-dessus de l'extrait
   - Annotations sur les points techniques remarquables

4. **Catégoriser les extraits** :
   - **Corps du dossier** : extraits courts (10-15 lignes max), illustrant un point précis
   - **Annexe** : extraits plus longs ou complets, pour référence détaillée

5. **Annoter les points techniques** : pour chaque extrait, expliquer ce qui est remarquable et en quoi cela démontre une compétence du référentiel CDA.

## Format de sortie

Pour chaque extrait :

```
### [Titre descriptif]

**Fichier** : `app/[backend|frontend]/src/...`
**Destination suggérée** : Corps §X / Annexe Y
**CP démontrée** : CPX — [nom de la compétence]

[Annotation : explication du point technique remarquable]

\`\`\`typescript
// extrait de code
\`\`\`
```

6. **Maintenir le fichier récap `docs/EXTRAITS-CODE.md`** :
   - Après chaque extraction, ajouter ou mettre à jour les extraits dans ce fichier
   - Organiser par section du dossier (§7.1, §7.2, etc.)
   - Ce fichier est la référence unique de tous les extraits validés
   - Le mettre à jour au fil de la discussion (ajouts, suppressions, modifications)

## Rappels
- Privilégier la qualité à la quantité : quelques extraits bien choisis valent mieux que beaucoup d'extraits moyens
- Les extraits doivent être compréhensibles hors contexte pour le jury
- Vérifier que le code extrait est bien le code actuel (pas une version obsolète)
- **Le code doit être le VRAI code** — jamais réécrit ni simplifié. Seule la troncature avec `// ...` est autorisée pour raccourcir
