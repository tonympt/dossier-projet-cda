---
name: rediger
description: Rédiger une section du dossier CDA GreenRoots en mode collaboratif. Utiliser quand on veut rédiger ou écrire une section du dossier de projet.
argument-hint: "[numéro-section]"
---

# Skill : Rédiger une section du dossier CDA (mode collaboratif)

## Objectif
Rédiger la section **$ARGUMENTS** du dossier de projet CDA GreenRoots en mode collaboratif avec Tony, le candidat.

## Workflow collaboratif

Le candidat est **acteur de la rédaction**. Le processus est interactif, pas automatique.

### Phase 1 — Préparation (Claude)
1. Lire `docs/PLAN-DOSSIER.md` pour identifier le contenu attendu de la section $ARGUMENTS, les CPs associées et la longueur cible. **C'est la base de travail.**
2. Lire `docs/COMPETENCES.md` pour les CPs concernées.
3. Lire `docs/CONSIGNES-REDACTION.md` pour les contraintes de format et ton.
4. Lire `docs/AVANCEMENT.md` pour les décisions déjà prises.
5. Explorer le code source dans `projet-greenroots/app/` si nécessaire.

### Phase 2 — Exploration (Claude questionne, Tony répond)
L'objectif n'est PAS de collecter du contenu pour tout mettre dans le texte. C'est de **comprendre** :
- Ce que Tony a fait concrètement vs ce qu'il n'a pas fait
- Son niveau de maîtrise sur chaque aspect (peut-il le défendre à l'oral ?)
- Ses choix, justifications, problèmes rencontrés
- Ce qu'il referait différemment

Poser les questions **par petits groupes** (3-5 max), pas tout d'un coup.
Si Tony ne maîtrise pas un concept, l'expliquer et déclencher `/notes-oral`.

**⚠️ OBLIGATOIRE : La phase 2 doit être exécutée pour CHAQUE sous-section, même au sein d'une même section.** Ne jamais sauter directement au tri (phase 3). La phase 2 est indispensable pour que Tony s'approprie le sujet et soit en rédaction active. C'est un échange, pas un simple questionnaire.

### Phase 3 — Tri (décision conjointe)
Présenter à Tony une **liste des points possibles** pour la section, basée sur :
- Le contenu prévu dans `PLAN-DOSSIER.md`
- Les réponses de la phase 2
- Les CPs à couvrir

**Pour chaque point, décider ensemble : on garde, on reformule, ou on enlève.**
Critères de décision :
- ✅ Pertinent pour les CPs évaluées ?
- ✅ Tony peut le défendre à l'oral ? (sinon : reformuler ou enlever)
- ✅ Ne chevauche pas une autre section ?
- ✅ Apporte de la valeur au jury ? (pas de remplissage)

Le tri produit un **plan de rédaction validé** : liste des points retenus avec leur angle.

### Phase 4 — Rédaction (Claude rédige sur la base du tri)
Rédiger la section en respectant :
- Le **plan de rédaction validé** au tri — ne rien ajouter qui n'a pas été décidé
- Le **ton** : professionnel, technique mais accessible au jury
- La **longueur cible** indiquée dans le plan
- La **justification** de chaque choix en lien avec les CPs
- Les **fils rouges** : sécurité, éco-conception, RGPD — quand pertinent
- La **cohérence** avec les autres sections (pas de redites)
- Le texte doit refléter **la voix et l'expérience de Tony**, pas un ton générique

### Phase 5 — Validation (Tony relit, on ajuste)
- Tony relit le texte proposé
- Ajustements jusqu'à satisfaction
- Déclencher `/maj-docs` et `/notes-oral` si pertinent

## Rappels importants
- `docs/PLAN-DOSSIER.md` est la **référence principale** — toujours partir de ce qui y est prévu
- Ne pas chevaucher avec d'autres sections (§2 besoins produit ≠ §5 spécifications techniques, §8 sécurité implémentations ≠ §10 veille/processus)
- Toujours ancrer les propos dans le concret du projet GreenRoots, pas de généralités théoriques
- Tony a participé à toutes les étapes du projet (front, back, infra) — pas de rôle attitré, il a touché à tout
- Le texte final doit être défendable à l'oral : Tony doit pouvoir expliquer chaque point car il en est l'auteur
- Ne PAS mettre systématiquement toutes les réponses de Tony dans le texte — filtrer via le tri
