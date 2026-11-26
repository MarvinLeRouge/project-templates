# project-templates

[Français](README.fr.md)

Reusable CI/CD, Docker, Codecov, Dependabot and Git hook configurations for the portfolio projects hosted on marvinlerouge.dev. New projects start from these files instead of writing their tooling from scratch, so every repository keeps the same quality gates.

## Structure

| Folder | Content |
|---|---|
| `components/github-actions/` | `ci-fastapi-vue.yml`, `ci-fastify-vue.yml`, `ci-laravel-vue.yml` (one full CI per stack), `build-push-deploy.yml`, `e2e.yml`, `changelog.yml` |
| `components/pre-commit/` | `base.yml` (gitleaks, changelog), `python.yml`, `node-vue.yml`, `php.yml` for pre-commit.com projects |
| `components/husky/` | `pre-commit` and `post-commit` for the Node and Laravel stacks |
| `components/changelog/` | `cliff.toml` and `changelog-post-commit.sh` (git-cliff) |
| `components/dependabot/` | `dependabot.yml` with the portfolio update rules |
| `components/codecov/` | `codecov.base.yml` (frontend and backend flags) |
| `components/docker/` | `docker-compose.dev.yml`, `docker-compose.prod.yml`, `traefik-labels.yml` |
| `stacks/` | one README per stack: which components to copy and which values to substitute |

Supported stacks: [FastAPI + Vue](stacks/fastapi-vue/README.md), [Fastify + Vue (pnpm monorepo)](stacks/fastify-vue/README.md), [Laravel + Vue](stacks/laravel-vue/README.md).

## Baseline (2026-11)

| Item | Value |
|---|---|
| Runtimes | Python 3.14 (uv), Node 26, PHP 8.5 |
| GitHub Actions | pinned by commit SHA, version in a trailing comment, one weekly Dependabot `actions` group |
| CI | one `ci.yml` per repo: path filter, lint / test / security jobs per part, `ci-status` aggregator |
| Required check on `main` | `CI status` (docs-only PRs pass, skipped jobs count as success) |
| Coverage | 85% minimum per part (backend, frontend) |
| Secret scan | gitleaks on staged changes in the pre-commit hook |
| Changelog | git-cliff, post-commit hook plus `changelog.yml` on `main` |
| Deploy | `build-push-deploy.yml`: `Build & push` to GHCR, then `Deploy` over SSH; prod config in `shared/env` on the VPS |

## Usage

1. Pick the stack README in `stacks/`.
2. Copy the components it lists into the new project (workflows into `.github/workflows/`, `dependabot.yml` into `.github/`, `changelog-post-commit.sh` into `scripts/`).
3. Replace every `${VARIABLE}` with the project value. The shared ones are `${PROJECT_NAME}`, `${DOMAIN}`, `${API_DOMAIN}`, `${GHCR_OWNER}`, `${BACKEND_PORT}`, `${FRONTEND_PORT}` and `${DB_ENGINE}`; each file lists its own in its header.
4. Delete the blocks that do not apply (for example the Dependabot ecosystems the project does not use).
5. Protect `main` with the standard ruleset, `CI status` as required check.

## Maintenance

A change to a component applies to new projects. Existing projects get it through one branch and one PR per repository, never silently.
