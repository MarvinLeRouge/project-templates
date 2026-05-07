# Stack: Fastify + Vue 3 (pnpm monorepo)

pnpm workspace monorepo: `apps/api/` (Node/Fastify + Prisma) + `apps/web/` (Vue 3) + `packages/shared/` (shared types).
Reference project: hivemind.

---

## Components to use

| Purpose    | Component                                    |
|------------|----------------------------------------------|
| CI         | `components/github-actions/ci-node-pnpm.yml` |
| CD         | `components/github-actions/cd.yml`           |
| E2E        | `components/github-actions/e2e.yml`          |
| Codecov    | `components/codecov/codecov.base.yml`        |
| Prod       | `components/docker/docker-compose.prod.yml`  |
| Dev        | `components/docker/docker-compose.dev.yml`   |
| Pre-commit | `components/pre-commit/node-vue.yml`         |

---

## Variable substitution

### ci-node-pnpm.yml

| Variable                   | Value                              |
|----------------------------|------------------------------------|
| `${NODE_VERSION}`          | `20`                               |
| `${BACKEND_APP_DIR}`       | `apps/api`                         |
| `${FRONTEND_APP_DIR}`      | `apps/web`                         |
| `${SHARED_PACKAGES_DIR}`   | `packages/shared`                  |
| `${BACKEND_LINT_CMD}`      | `pnpm --filter api lint`           |
| `${BACKEND_TEST_CMD}`      | `pnpm --filter api test:coverage`  |
| `${COVERAGE_FILE_BACKEND}` | `apps/api/coverage/lcov.info`      |
| `${FRONTEND_LINT_CMD}`     | `pnpm --filter web lint`           |
| `${FRONTEND_TEST_CMD}`     | `pnpm --filter web test:coverage`  |
| `${COVERAGE_FILE_FRONTEND}`| `apps/web/coverage/lcov.info`      |

**Prisma setup** — uncomment the Prisma steps in the `backend` job:
```yaml
- name: Generate Prisma client
  run: pnpm --filter api exec prisma generate
- name: Run migrations
  run: pnpm --filter api exec prisma migrate deploy
- name: Seed database
  run: pnpm --filter api exec prisma db seed
```
These require the `postgres` service block in the `backend` job (add under `services:`).

### cd.yml

| Variable                   | Value                            |
|----------------------------|----------------------------------|
| `${CI_WORKFLOW_NAME}`      | `CI`                             |
| `${GHCR_OWNER}`            | `marvinlerouge`                  |
| `${PROJECT_NAME}`          | `<your-project-name>`            |
| `${BACKEND_BUILD_CONTEXT}` | `.`                              |
| `${BACKEND_DOCKERFILE}`    | `apps/api/Dockerfile`            |
| `${FRONTEND_BUILD_CONTEXT}`| `.`                              |
| `${FRONTEND_DOCKERFILE}`   | `apps/web/Dockerfile`            |
| `${DEPLOY_PATH}`           | `/opt/<project>`                 |
| `${COMPOSE_FILE}`          | `docker-compose.prod.yml`        |

**Migration step** — uncomment and set in the deploy script:
```bash
docker compose -f docker-compose.prod.yml exec -T backend \
  sh -c "npx prisma migrate deploy"
```

### e2e.yml

| Variable              | Value                           |
|-----------------------|---------------------------------|
| `${HEALTH_CHECK_URL}` | `http://localhost:3000/health`  |
| `${E2E_CMD}`          | `pnpm test:e2e`                 |
| `${E2E_REPORT_PATH}`  | `playwright-report/`            |

Setup step to uncomment:
```bash
docker compose exec -T backend sh -c "npx prisma migrate deploy && npx prisma db seed"
```

### codecov.base.yml

| Variable               | Value           |
|------------------------|-----------------|
| `${FRONTEND_SRC_PATH}` | `apps/web/src/` |
| `${BACKEND_SRC_PATH}`  | `apps/api/src/` |

### docker-compose.prod.yml

| Variable            | Value                             |
|---------------------|-----------------------------------|
| `${PROJECT_NAME}`   | `<your-project-name>`             |
| `${DOMAIN}`         | `<project>.marvinlerouge.dev`     |
| `${API_DOMAIN}`     | `api-<project>.marvinlerouge.dev` |
| `${GHCR_OWNER}`     | `marvinlerouge`                   |
| `${BACKEND_PORT}`   | `3000`                            |
| `${FRONTEND_PORT}`  | `80`                              |
| `${DB_ENGINE}`      | `postgres:16-alpine`              |

### pre-commit/node-vue.yml

| Variable           | Value                            |
|--------------------|----------------------------------|
| `${FRONTEND_DIR}`  | `apps/web`                       |
| `${LINT_FIX_CMD}`  | `pnpm --filter web lint:fix`     |
| `${TYPECHECK_CMD}` | `pnpm --filter web typecheck`    |

For Prettier, make sure `node_modules/.bin/prettier` resolves at the repo root
(pnpm hoists binaries to `node_modules/.bin/` by default).

---

## Notes

- The `ci-node-pnpm.yml` template uses `dorny/paths-filter` to skip unchanged apps. Changes to `packages/shared/` or `pnpm-lock.yaml` trigger both jobs.
- Dockerfile build context is `.` (repo root) for both images — required to include workspace packages during build.
- `pnpm/action-setup@v4` reads the `packageManager` field from `package.json` to pin the pnpm version. Set it there rather than in the workflow.
- If the backend exposes a health endpoint, add a healthcheck to the `backend` service in the prod compose. Adjust the URL in `e2e.yml` accordingly.
