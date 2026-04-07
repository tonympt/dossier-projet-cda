---
name: maj-docs
description: Mettre à jour les documents vivants du dossier CDA après validation d'une section. Met à jour AVANCEMENT, PLAN-DOSSIER, COMPETENCES et autres docs de référence.
argument-hint: "[numéro-section] [résumé optionnel]"
---

# Skill : Mettre à jour les documents vivants

## Objectif
Après validation de la section **$ARGUMENTS**, mettre à jour tous les documents de référence du dossier CDA.

## Documents à vérifier et mettre à jour

### 1. `docs/AVANCEMENT.md` (toujours)
- Marquer la section comme **rédigée/validée**
- Ajouter un résumé des décisions prises lors de la rédaction
- Mettre à jour la date

### 2. `docs/PLAN-DOSSIER.md` (si nécessaire)
- Enrichir avec le contenu détaillé validé si le plan initial était générique
- Ajouter des précisions sur le contenu effectivement rédigé

### 3. `docs/COMPETENCES.md` (si nécessaire)
- Ajouter des précisions sur la couverture d'une CP si la rédaction a apporté de nouvelles démonstrations
- Mettre à jour le statut de couverture

### 4. `docs/CONSIGNES-REDACTION.md` (si nécessaire)
- Ajouter de nouvelles décisions transversales prises pendant la rédaction
- Documenter les choix de rédaction qui s'appliquent à d'autres sections

### 5. `docs/PROJET-GREENROOTS.md` (si nécessaire)
- Ajouter de nouvelles informations techniques ou fonctionnelles découvertes pendant la rédaction

### 6. `MEMORY.md` (mémoire persistante, si nécessaire)
- Mettre à jour la mémoire projet dans `/home/tonympt/.claude/projects/-home-tonympt-workspace-github-tonympt-dossier-projet-cda/memory/MEMORY.md`
- Ajouter les décisions structurantes ou informations qui doivent persister entre sessions
- Mettre à jour les infos existantes si elles ont changé

## Règles
- **Lire chaque fichier avant de le modifier** — ne jamais modifier à l'aveugle
- **Ne modifier que ce qui est pertinent** — pas de modification inutile ou cosmétique
- **Ne pas modifier un fichier s'il n'y a rien à changer** pour cette section

## Livrable
Lister les fichiers modifiés et les changements effectués :
```
## Mise à jour des docs — Section [X]

### Fichiers modifiés
- `docs/AVANCEMENT.md` : [description des changements]
- `docs/COMPETENCES.md` : [description des changements]
- ...

### Fichiers non modifiés (rien à changer)
- `docs/CONSIGNES-REDACTION.md`
- ...
```
