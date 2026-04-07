---
name: verifier
description: Vérifier une section rédigée du dossier CDA. Contrôle la couverture des CPs, le ton, la longueur, la cohérence et les fils rouges.
argument-hint: "[numéro-section ou fichier]"
---

# Skill : Vérifier une section rédigée du dossier CDA

## Objectif
Vérifier la section **$ARGUMENTS** selon une grille de critères qualité, et produire un rapport structuré.

## Grille de vérification

### 1. Couverture des CPs
- Lire `docs/PLAN-DOSSIER.md` pour identifier les CPs associées à cette section
- Lire `docs/COMPETENCES.md` pour comprendre les attendus de chaque CP
- Vérifier que chaque CP listée est effectivement démontrée dans le texte rédigé
- Verdict : OK / À corriger (lister les CPs manquantes ou insuffisamment couvertes)

### 2. Ton et style
- Professionnel, technique mais accessible au jury
- Pas trop académique (pas de cours magistral), pas trop familier
- Ancré dans le concret du projet GreenRoots (pas de généralités théoriques)
- Lire `docs/CONSIGNES-REDACTION.md` pour les contraintes spécifiques
- Verdict : OK / À corriger (exemples de passages à revoir)

### 3. Longueur
- Estimer le nombre de pages (environ 400 mots/page)
- Comparer à la longueur cible indiquée dans `docs/PLAN-DOSSIER.md`
- Verdict : OK / Trop court / Trop long (avec estimation)

### 4. Cohérence avec les autres sections
- Lire `docs/AVANCEMENT.md` pour les décisions prises et les frontières entre sections
- Vérifier qu'il n'y a pas de chevauchement avec d'autres sections
- Vérifier qu'il n'y a pas de contradiction avec les décisions déjà prises
- Verdict : OK / À corriger (lister les chevauchements ou contradictions)

### 5. Fils rouges
- Sécurité : mentionnée si pertinent pour la section ?
- Éco-conception : mentionnée si pertinent (cohérence mission reforestation + démarche technique) ?
- RGPD : mentionné si pertinent ?
- Verdict : OK / À corriger (fils rouges manquants)

### 6. Extraits de code
- Si des extraits de code sont présents, vérifier qu'ils correspondent au code réel dans `projet-greenroots/`
- Vérifier que les extraits dans le corps font 10-15 lignes max
- Vérifier que les chemins source sont indiqués
- Verdict : OK / À corriger / N/A

### 7. Diagrammes
- Si des diagrammes Mermaid sont présents, vérifier la syntaxe
- Verdict : OK / À corriger / N/A

### 8. Format général
- Pas de redite interne à la section
- Pas de hors-sujet par rapport au plan
- Structure logique et lisible
- Verdict : OK / À corriger

## Livrable
Produire un rapport structuré :
```
## Rapport de vérification — Section [X]

| Critère | Verdict | Détail |
|---------|---------|--------|
| CPs couvertes | ... | ... |
| Ton et style | ... | ... |
| Longueur | ... | ... |
| Cohérence | ... | ... |
| Fils rouges | ... | ... |
| Extraits de code | ... | ... |
| Diagrammes | ... | ... |
| Format | ... | ... |

### Corrections requises
- ...

### Suggestions d'amélioration
- ...
```
