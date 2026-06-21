---
title: github-reuse-workflow Consumers
repo: github-reuse-workflow
layer: infra
tech: GH Actions
default_branch: main
updated: 2026-04-24
source_path: docs/consumers.md
---

## Consumers — repos that call these workflows

All FutureSkill repos that have CI/CD call workflows from this central repo via `workflow_call`.

### Known consumers

| Repo | Workflows used | Notes |
| --- | --- | --- |
| [LikeMeX/fs-backend](https://github.com/LikeMeX/fs-backend) | `semantic-release`, `build-image`, `deploy-to-cluster` | Main API — deploys to dev + production GKE |
| [LikeMeX/fs-frontend](https://github.com/LikeMeX/fs-frontend) | `semantic-release`, `static-site-export`, `deploy-static-cloudflare-page`, `deploy-to-cluster` | Next.js app — Cloudflare Pages + GKE |
| [LikeMeX/fs-mobile](https://github.com/LikeMeX/fs-mobile) | `semantic-release-mobile`, `build-ios-app-staging`, `build-ios-app-prod`, `build-android-app-staging`, `build-android-app-prod`, `lastest-buildnumber-release` | Flutter mobile app |
| [LikeMeX/fs-worker](https://github.com/LikeMeX/fs-worker) | `semantic-release`, `build-image`, `deploy-to-cluster` | Background worker service |

> If your repo calls these workflows and is not listed, open a PR to add it.

### Minimum required setup for a new consumer repo

1. Add repository secrets:

   | Secret | Where to get it |
   | --- | --- |
   | `GH_PAT` | GitHub → Settings → Developer settings → PAT (classic), `repo` scope |
   | Additional secrets | See `docs/env.md` for each workflow you call |

2. Create `.github/workflows/ci.yaml` in the consumer repo referencing the desired workflows (see `docs/runbook.md` for copy-paste snippets).

3. Ensure the consumer repo has `fs-app-chart` as a submodule if deploying to GKE:

```bash
git submodule add https://github.com/LikeMeX/fs-app-chart.git fs-app-chart
```

4. Provide per-environment values files at `deploy/values-<env>.yaml` for Helm overrides.

5. Provide env files at `env/.env.<env>` (plain, non-secret config) and encrypted secrets at `secrets/.env.enc.<env>` (SOPS-encrypted with the environment's KMS key).
