# Stack: Laravel + Vue 3

Laravel at the repo root, Vue 3 in `resources/js/` (npm, same `package.json`). Reference project: Summit-Stats.

## Baseline

| Item | Value |
|---|---|
| Runtimes | Python 3.14 (uv), Node 26, PHP 8.5 |
| Actions | pinned by commit SHA, version in a trailing comment, bumped by the Dependabot `actions` group |
| Required check on `main` | `CI status` (the `ci-status` aggregator job) |
| Coverage | 85% minimum per part (backend, frontend), Codecov project status |
| Secret scan | gitleaks on the staged changes in the pre-commit hook (system binary, the hook fails without it) |
| Changelog | git-cliff post-commit hook + `changelog.yml` on `main` |

## Shared components

| Purpose | Component |
|---|---|
| Deploy | `components/github-actions/build-push-deploy.yml` (jobs `Build & push` and `Deploy`) |
| E2E | `components/github-actions/e2e.yml` |
| Changelog | `components/github-actions/changelog.yml`, `components/changelog/` (`cliff.toml`, `changelog-post-commit.sh` into `scripts/`) |
| Dependabot | `components/dependabot/dependabot.yml` |
| Codecov | `components/codecov/codecov.base.yml` |
| Docker | `components/docker/docker-compose.prod.yml`, `components/docker/docker-compose.dev.yml`, `components/docker/traefik-labels.yml` |

Docker variables (`docker-compose.*.yml`): `${PROJECT_NAME}`, `${DOMAIN}`, `${API_DOMAIN}`, `${GHCR_OWNER}`, `${BACKEND_PORT}`, `${FRONTEND_PORT}`, `${DB_ENGINE}` (see the root `CLAUDE.md`). Prod config lives on the VPS in `shared/env/app.env` and `shared/env/secrets.env`, read through `env_file`.

## Stack components

| Purpose | Component |
|---|---|
| CI | `components/github-actions/ci-laravel-vue.yml` |
| Pre-commit | `components/husky/pre-commit` (tracked) + `components/husky/post-commit` (gitignored), lint-staged in `package.json` (eslint + prettier on `resources/js`, pint on PHP) |

## Variables

| File | Variable | Value |
|---|---|---|
| `build-push-deploy.yml` | `${BACKEND_DOCKERFILE}` / `${FRONTEND_DOCKERFILE}` | `Dockerfile` for both, with `target:` set per image |
| `build-push-deploy.yml` | `${MIGRATIONS_CMD}` | `php artisan migrate --force` |
| `dependabot.yml` | `${NPM_DIR}` / `${DOCKER_DIRS}` | `"/"` / `["/"]` (drop uv) |
| `husky/pre-commit` | `${LINT_STAGED_CMD}` | `npx lint-staged` then `composer phpstan` on the next line |
| `codecov.base.yml` | `${FRONTEND_SRC_PATH}` / `${BACKEND_SRC_PATH}` | `resources/js/` / `app/` |

## Notes

- Two images: `app` (PHP-FPM) and `nginx` (static assets baked in). Rename the `backend` / `frontend` images of `build-push-deploy.yml` accordingly.
- PHP 8.5: OPcache is built in, do not `docker-php-ext-install opcache` (the build fails).
- The queue worker reuses the `app` image with another `command:`; one `redis` service serves cache and queues.
- `components/pre-commit/php.yml` is only for a PHP project without Node.
