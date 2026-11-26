# project-templates

[English](README.md)

Configurations réutilisables de CI/CD, Docker, Codecov, Dependabot et hooks Git pour les projets du portfolio hébergés sur marvinlerouge.dev. Les nouveaux projets partent de ces fichiers au lieu de réécrire leur outillage, pour que chaque dépôt garde les mêmes garde-fous qualité.

## Structure

| Dossier | Contenu |
|---|---|
| `components/github-actions/` | `ci-fastapi-vue.yml`, `ci-fastify-vue.yml`, `ci-laravel-vue.yml` (une CI complète par stack), `build-push-deploy.yml`, `e2e.yml`, `changelog.yml` |
| `components/pre-commit/` | `base.yml` (gitleaks, changelog), `python.yml`, `node-vue.yml`, `php.yml` pour les projets pre-commit.com |
| `components/husky/` | `pre-commit` et `post-commit` pour les stacks Node et Laravel |
| `components/changelog/` | `cliff.toml` et `changelog-post-commit.sh` (git-cliff) |
| `components/dependabot/` | `dependabot.yml` avec les règles de mise à jour du portfolio |
| `components/codecov/` | `codecov.base.yml` (flags frontend et backend) |
| `components/docker/` | `docker-compose.dev.yml`, `docker-compose.prod.yml`, `traefik-labels.yml` |
| `stacks/` | un README par stack : quels composants copier et quelles valeurs substituer |

Stacks prises en charge : [FastAPI + Vue](stacks/fastapi-vue/README.md), [Fastify + Vue (monorepo pnpm)](stacks/fastify-vue/README.md), [Laravel + Vue](stacks/laravel-vue/README.md). Les README de stack sont en anglais (documentation technique).

## Socle (2026-11)

| Élément | Valeur |
|---|---|
| Runtimes | Python 3.14 (uv), Node 26, PHP 8.5 |
| GitHub Actions | épinglées par SHA de commit, version en commentaire, un groupe Dependabot `actions` hebdomadaire |
| CI | un seul `ci.yml` par dépôt : filtre de chemins, jobs lint / test / sécurité par partie, agrégateur `ci-status` |
| Check requis sur `main` | `CI status` (les PR de doc seule passent, les jobs ignorés comptent comme réussis) |
| Couverture | 85 % minimum par partie (backend, frontend) |
| Détection de secrets | gitleaks sur les modifications indexées dans le hook pre-commit |
| Changelog | git-cliff, hook post-commit plus `changelog.yml` sur `main` |
| Déploiement | `build-push-deploy.yml` : `Build & push` vers GHCR, puis `Deploy` en SSH ; config de prod dans `shared/env` sur le VPS |

## Utilisation

1. Choisir le README de stack dans `stacks/`.
2. Copier les composants qu'il liste dans le nouveau projet (workflows dans `.github/workflows/`, `dependabot.yml` dans `.github/`, `changelog-post-commit.sh` dans `scripts/`).
3. Remplacer chaque `${VARIABLE}` par la valeur du projet. Les variables communes sont `${PROJECT_NAME}`, `${DOMAIN}`, `${API_DOMAIN}`, `${GHCR_OWNER}`, `${BACKEND_PORT}`, `${FRONTEND_PORT}` et `${DB_ENGINE}` ; chaque fichier liste les siennes dans son en-tête.
4. Supprimer les blocs inutiles (par exemple les écosystèmes Dependabot que le projet n'utilise pas).
5. Protéger `main` avec le ruleset standard, `CI status` en check requis.

## Maintenance

Une modification de composant s'applique aux nouveaux projets. Les projets existants la reçoivent par une branche et une PR par dépôt, jamais en silence.
