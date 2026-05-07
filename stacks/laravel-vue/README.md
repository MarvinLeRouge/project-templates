# Stack: Laravel + Vue 3 (npm)

Standard Laravel layout at root with a Vue 3 frontend bundled by Vite.
Reference project: summit-stats.

---

## Components to use

| Purpose      | Component                                    |
|--------------|----------------------------------------------|
| CI backend   | `components/github-actions/ci-php.yml`       |
| CI frontend  | `components/github-actions/ci-node-npm.yml`  |
| CD           | `components/github-actions/cd.yml`           |
| E2E          | `components/github-actions/e2e.yml`          |
| Codecov      | `components/codecov/codecov.base.yml`        |
| Prod         | `components/docker/docker-compose.prod.yml`  |
| Dev          | `components/docker/docker-compose.dev.yml`   |
| Pre-commit   | `components/pre-commit/php.yml` + `components/pre-commit/node-vue.yml` |

---

## Variable substitution

### ci-php.yml

| Variable           | Value          |
|--------------------|----------------|
| `${PHP_VERSION}`   | `8.4`          |
| `${COVERAGE_FILE}` | `coverage.xml` |

The test job includes Node + npm steps for building frontend assets before running PHP tests. Adjust the Node version and npm commands if needed.

### ci-node-npm.yml

| Variable             | Value                               |
|----------------------|-------------------------------------|
| `${NODE_VERSION}`    | `20`                                |
| `${LINT_CMD}`        | `npm run lint`                      |
| `${TYPECHECK_CMD}`   | `npm run typecheck`                 |
| `${TEST_CMD}`        | `npm run test:coverage`             |
| `${COVERAGE_FILE}`   | `coverage-frontend/lcov.info`       |

### cd.yml

This stack builds **two images**: `app` (PHP-FPM) and `nginx` (static assets baked in).
Add a second `Build and push` step for the nginx image:

```yaml
- name: Build and push nginx
  uses: docker/build-push-action@v6
  with:
    context: .
    file: Dockerfile
    target: nginx-prod
    push: true
    tags: |
      ghcr.io/${GHCR_OWNER}/${PROJECT_NAME}/nginx:${{ steps.meta.outputs.tag }}
      ghcr.io/${GHCR_OWNER}/${PROJECT_NAME}/nginx:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

| Variable                   | Value                              |
|----------------------------|------------------------------------|
| `${CI_WORKFLOW_NAME}`      | `CI`                               |
| `${GHCR_OWNER}`            | `marvinlerouge`                    |
| `${PROJECT_NAME}`          | `<your-project-name>`              |
| `${BACKEND_BUILD_CONTEXT}` | `.`                                |
| `${BACKEND_DOCKERFILE}`    | `Dockerfile`                       |
| `${BACKEND_BUILD_TARGET}`  | `production` (uncomment `target:`) |
| `${FRONTEND_BUILD_CONTEXT}`| `.`                                |
| `${FRONTEND_DOCKERFILE}`   | `Dockerfile`                       |
| `${FRONTEND_BUILD_TARGET}` | `nginx-prod` (uncomment `target:`) |
| `${DEPLOY_PATH}`           | `/home/mlr/marvinlerouge.dev/<project>/compose` |
| `${COMPOSE_FILE}`          | `docker-compose.prod.yml`          |

**Migration step** — uncomment and set in the deploy script:
```bash
docker compose -f docker-compose.prod.yml run --rm app php artisan migrate --force
```

### e2e.yml

| Variable              | Value                            |
|-----------------------|----------------------------------|
| `${HEALTH_CHECK_URL}` | `http://localhost:8081/login`    |
| `${E2E_CMD}`          | `npm run test:e2e`               |
| `${E2E_REPORT_PATH}`  | `playwright-report/`             |

Setup step to uncomment:
```bash
docker compose exec -T app php artisan migrate --force
docker compose exec -T app php artisan db:seed
```

### codecov.base.yml

| Variable               | Value                     |
|------------------------|---------------------------|
| `${FRONTEND_SRC_PATH}` | `resources/js/`           |
| `${BACKEND_SRC_PATH}`  | `app/`                    |

### docker-compose.prod.yml

Laravel uses a separate nginx service (baked with static assets).
Rename `frontend` → `nginx` and `backend` → `app`. Add a `queue` worker service:

```yaml
  queue:
    image: ghcr.io/${GHCR_OWNER}/${PROJECT_NAME}/app:${IMAGE_TAG:-latest}
    restart: unless-stopped
    command: php artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
    volumes:
      - storage_data:/app/storage
    depends_on:
      app:
        condition: service_started
      redis:
        condition: service_healthy
    env_file:
      - .env.prod
    networks:
      - internal
```

Also add a `redis` service on the `internal` network.

| Variable            | Value                             |
|---------------------|-----------------------------------|
| `${PROJECT_NAME}`   | `<your-project-name>`             |
| `${DOMAIN}`         | `<project>.marvinlerouge.dev`     |
| `${API_DOMAIN}`     | *(not applicable — single domain)*|
| `${GHCR_OWNER}`     | `marvinlerouge`                   |
| `${BACKEND_PORT}`   | *(not exposed directly — nginx in front)* |
| `${FRONTEND_PORT}`  | `80`                              |
| `${DB_ENGINE}`      | `postgres:16-alpine`              |

The nginx service is the only one on `traefik-public`. The `app` (PHP-FPM) service stays on `internal`.

### pre-commit/php.yml

| Variable           | Value |
|--------------------|-------|
| `${PHPSTAN_LEVEL}` | `5`   |

### pre-commit/node-vue.yml

| Variable           | Value                       |
|--------------------|-----------------------------|
| `${FRONTEND_DIR}`  | `resources/js`              |
| `${LINT_FIX_CMD}`  | `npm run lint:fix`          |
| `${TYPECHECK_CMD}` | `npm run typecheck`         |

---

## Notes

- Unlike other stacks, Laravel's PHP and JS code live in the same repo root. The `ci-php.yml` test job already includes Node steps to build assets — no need to chain the two CI workflows.
- The Codecov `backend` flag covers `app/` (Laravel application logic). The `frontend` flag covers `resources/js/`.
- The queue worker uses the same `app` image as PHP-FPM; only the `command:` differs.
- If using Redis for caching in addition to queues, declare a single `redis` service and reference it from both `app` and `queue`.
