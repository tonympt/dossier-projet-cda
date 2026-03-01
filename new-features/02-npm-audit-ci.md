# Feature 02 — npm audit + ajout CI

## Priorité : P1 | Estimation : ~30 min

## Contexte
Aucun audit de sécurité des dépendances n'était intégré dans le workflow CI. Les vulnérabilités connues n'étaient pas détectées automatiquement lors des PR.

## Modifications

### 1. Correction des vulnérabilités (`npm audit fix`)
- **Backend** : 28 → 25 vulnérabilités (3 corrigées)
- **Frontend** : 6 → 3 vulnérabilités (3 corrigées)
- Les vulnérabilités restantes sont dans des **dépendances transitives upstream** (NestJS, Prisma, ESLint) — non corrigeables sans breaking changes

### 2. Ajout de l'audit de sécurité dans la CI (`.github/workflows/pr-checks.yml`)
- Étape `npm audit --audit-level=critical` ajoutée dans les jobs **backend-checks** et **frontend-checks**
- **Bloquant** : toute PR introduisant une vulnérabilité de niveau `critical` sera rejetée
- Choix de `critical` plutôt que `high` : les vulnérabilités `high` restantes sont toutes dans des deps transitives non corrigeables — utiliser `high` ferait échouer systématiquement la CI

### Vulnérabilités restantes (upstream, non actionnables)
| Dépendance | Sévérité | Origine | Raison |
|---|---|---|---|
| multer | high | @nestjs/platform-express | Fix = downgrade NestJS |
| minimatch | high | @nestjs/cli, ESLint | Deps de dev, pas en prod |
| ajv | moderate | @nestjs/config, webpack | Pas de fix disponible |
| hono, lodash | moderate/high | Prisma CLI | Deps de dev |
| axios | high | frontend | Corrigé par audit fix |

## Branche
`feature/npm-audit-ci`

## Fichiers modifiés
- `.github/workflows/pr-checks.yml` (+8 lignes)
- `app/backend/package-lock.json` (audit fix)
- `app/frontend/package-lock.json` (audit fix)

## Section du dossier CDA concernée
- §10 (veille sécurité — audit automatisé des dépendances)
