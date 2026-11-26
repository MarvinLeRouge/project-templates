# Stack: FastAPI + Vue 3

uv project at the repo root (`pyproject.toml`, `uv.lock`, backend code in `backend/`) + `frontend/` (Vue 3, npm). Reference project: Stamped.

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
| CI | `components/github-actions/ci-fastapi-vue.yml` |
| Pre-commit | `components/pre-commit/base.yml` + `python.yml` + `node-vue.yml`, merged into one `.pre-commit-config.yaml` |

## Variables

| File | Variable | Value |
|---|---|---|
| `ci-fastapi-vue.yml` | `${PYTHON_PACKAGE}` | backend package, e.g. `stamped` |
| `build-push-deploy.yml` | `${BACKEND_DOCKERFILE}` / `${FRONTEND_DOCKERFILE}` | e.g. `backend/Dockerfile` / `frontend/Dockerfile` (Stamped is not deployed: no reference) |
| `build-push-deploy.yml` | `${MIGRATIONS_CMD}` | `alembic upgrade head` |
| `dependabot.yml` | `${NPM_DIR}` / `${DOCKER_DIRS}` | `"/frontend"` / `["/backend", "/frontend"]` (drop the composer block) |
| `pre-commit/python.yml` | `${RUFF_REV}` / `${BACKEND_DIR}` | ruff version from `uv.lock` / `backend` |
| `pre-commit/node-vue.yml` | `${FRONTEND_DIR}` / `${LINT_FIX_CMD}` / `${TYPECHECK_CMD}` | `frontend` / `cd frontend && npm run lint` / `cd frontend && npm run type-check` |
| `codecov.base.yml` | `${FRONTEND_SRC_PATH}` / `${BACKEND_SRC_PATH}` | `frontend/src/` / `backend/` |

## Notes

- CI runs `uv sync --locked`, then ruff, mypy, pytest and pip-audit through `uv run` / `uvx`; pip-audit audits `uv export --no-emit-project`.
- Without E2E, set the deploy gate to `workflows: ["CI"]`.
