# Stack: FastAPI + Vue 3 (npm)

Monorepo layout: `backend/` (Python/FastAPI) + `frontend/` (Vue 3, npm).
Reference project: gc-tracker.

---

## Components to use

| Purpose    | Component                                    |
|------------|----------------------------------------------|
| CI backend | `components/github-actions/ci-python.yml`    |
| CI frontend| `components/github-actions/ci-node-npm.yml`  |
| CD         | `components/github-actions/cd.yml`           |
| E2E        | `components/github-actions/e2e.yml`          |
| Codecov    | `components/codecov/codecov.base.yml`        |
| Prod       | `components/docker/docker-compose.prod.yml`  |
| Dev        | `components/docker/docker-compose.dev.yml`   |
| Pre-commit | `components/pre-commit/python.yml` + `components/pre-commit/node-vue.yml` |

---

## Variable substitution

### ci-python.yml

| Variable               | Value                            |
|------------------------|----------------------------------|
| `${PYTHON_VERSION}`    | `3.12`                           |
| `${REQUIREMENTS_FILE}` | `backend/requirements.txt`       |
| `${REQUIREMENTS_DEV_FILE}` | `backend/requirements-dev.txt` |
| `${BACKEND_DIR}`       | `backend`                        |
| `${TEST_PATH}`         | `tests/unit/`                    |
| `${COVERAGE_FILE}`     | `backend/coverage.xml`           |

### ci-node-npm.yml

| Variable             | Value                              |
|----------------------|------------------------------------|
| `${NODE_VERSION}`    | `20`                               |
| `${LINT_CMD}`        | `npm run lint`                     |
| `${TYPECHECK_CMD}`   | `npm run typecheck`                |
| `${TEST_CMD}`        | `npm run test:unit`                |
| `${COVERAGE_FILE}`   | `frontend/coverage/lcov.info`      |

### cd.yml

| Variable                   | Value                          |
|----------------------------|--------------------------------|
| `${CI_WORKFLOW_NAME}`      | `CI`                           |
| `${GHCR_OWNER}`            | `marvinlerouge`                |
| `${PROJECT_NAME}`          | `<your-project-name>`          |
| `${BACKEND_BUILD_CONTEXT}` | `./backend`                    |
| `${BACKEND_DOCKERFILE}`    | `./backend/Dockerfile`         |
| `${FRONTEND_BUILD_CONTEXT}`| `.`                            |
| `${FRONTEND_DOCKERFILE}`   | `./frontend/Dockerfile`        |
| `${DEPLOY_PATH}`           | `/home/mlr/marvinlerouge.dev/<project>/compose` |
| `${COMPOSE_FILE}`          | `ops/deploy/docker-compose.prod.yml` |

**Migration step** — uncomment and set in the deploy script:
```bash
docker compose -f docker-compose.prod.yml exec -T backend \
  sh -c "alembic upgrade head"
```

### e2e.yml

| Variable              | Value                             |
|-----------------------|-----------------------------------|
| `${HEALTH_CHECK_URL}` | `http://localhost:8000/health`    |
| `${E2E_CMD}`          | `npm run test:e2e`                |
| `${E2E_REPORT_PATH}`  | `playwright-report/`              |

Setup step to uncomment:
```bash
docker compose exec -T backend sh -c "alembic upgrade head"
```

### codecov.base.yml

| Variable               | Value          |
|------------------------|----------------|
| `${FRONTEND_SRC_PATH}` | `frontend/src/`|
| `${BACKEND_SRC_PATH}`  | `backend/`     |

### docker-compose.prod.yml

| Variable            | Value                                  |
|---------------------|----------------------------------------|
| `${PROJECT_NAME}`   | `<your-project-name>`                  |
| `${DOMAIN}`         | `<project>.marvinlerouge.dev`          |
| `${API_DOMAIN}`     | `api-<project>.marvinlerouge.dev`      |
| `${GHCR_OWNER}`     | `marvinlerouge`                        |
| `${BACKEND_PORT}`   | `8000`                                 |
| `${FRONTEND_PORT}`  | `80`                                   |
| `${DB_ENGINE}`      | `postgres:16-alpine`                   |

The backend service uses a FastAPI healthcheck endpoint (`/health`).
The default compose file path for the curl fetch in cd.yml is `ops/deploy/docker-compose.prod.yml`.

### pre-commit/python.yml

| Variable          | Value                       |
|-------------------|-----------------------------|
| `${RUFF_REV}`     | see latest ruff-pre-commit release |
| `${BACKEND_DIR}`  | `backend`                   |
| `${MYPY_CONFIG}`  | `backend/pyproject.toml`    |

### pre-commit/node-vue.yml

| Variable           | Value                   |
|--------------------|-------------------------|
| `${FRONTEND_DIR}`  | `frontend`              |
| `${LINT_FIX_CMD}`  | `npm run lint:fix`      |
| `${TYPECHECK_CMD}` | `npm run typecheck`     |

---

## Notes

- The `backend/` directory contains a Python virtual environment at `.venv/`. Mypy runs via `.venv/bin/mypy` in the pre-commit hook.
- If the project uses database services (PostgreSQL, etc.), add them to both CI test job (`services:`) and the e2e Compose file.
- The `cd.yml` fetches `ops/deploy/docker-compose.prod.yml` by default. Adjust `${COMPOSE_FILE}` if your prod compose lives elsewhere.
- CI uses two separate workflows (ci-python.yml + ci-node-npm.yml). Name both `CI` and list them both in the `cd.yml` trigger, or gate on one and let the other be informational.
