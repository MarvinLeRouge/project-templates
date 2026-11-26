# Stack: Fastify + Vue 3 (pnpm monorepo)

pnpm workspace: `apps/api/` (Fastify + Prisma 7) + `apps/web/` (Vue 3) + `packages/shared/` (shared types). Reference project: HiveMind.

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
| CI | `components/github-actions/ci-fastify-vue.yml` |
| Pre-commit | `components/husky/pre-commit` (tracked) + `components/husky/post-commit` (gitignored), lint-staged in `package.json` |

## Variables

| File | Variable | Value |
|---|---|---|
| `ci-fastify-vue.yml` | `${PROJECT_NAME}` | CI database name and user |
| `build-push-deploy.yml` | `${BACKEND_DOCKERFILE}` / `${FRONTEND_DOCKERFILE}` | `apps/api/Dockerfile` / `apps/web/Dockerfile` |
| `build-push-deploy.yml` | `${MIGRATIONS_CMD}` | `node_modules/.bin/prisma migrate deploy` |
| `dependabot.yml` | `${NPM_DIR}` / `${DOCKER_DIRS}` | `"/"` (one entry for the workspace) / `["/apps/api", "/apps/web"]` (drop uv and composer) |
| `husky/pre-commit` | `${LINT_STAGED_CMD}` | `pnpm lint-staged` |
| `codecov.base.yml` | `${FRONTEND_SRC_PATH}` / `${BACKEND_SRC_PATH}` | `apps/web/src/` / `apps/api/src/` |

## Notes

- Prisma 7: `prisma.config.ts` reads `DATABASE_URL` from `process.env` (not `env()`, which throws when the variable is missing during `prisma generate`), `prisma-client` generator, `@prisma/adapter-pg`. CI runs `prisma generate` before lint and typecheck.
- The Docker image must copy `prisma.config.ts` for `migrate deploy`.
