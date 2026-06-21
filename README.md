---
title: FutureSkill Reusable GitHub Actions Workflows
repo: github-reuse-workflow
layer: infra
tech: GH Actions
default_branch: main
updated: 2026-04-24
source_path: /
---

## github-reuse-workflow — Reusable CI/CD Workflows

Central repository of reusable GitHub Actions workflows (`workflow_call`) for all FutureSkill repos. Consumer repos call these workflows rather than duplicating CI/CD logic.

**Canonical overview:** [architecture.md](https://github.com/LikeMeX/fs-infrastructure/blob/main/docs/architecture.md)

## Workflow catalogue

| File | Name | Purpose |
| --- | --- | --- |
| `semantic-release.yaml` | Semantic release | Conventional-commit tagging via `cycjimmy/semantic-release-action@v4`; outputs `tag`, `new_release_published`, `latest_tag` |
| `semantic-release-mobile.yaml` | Semantic release (mobile) | Same as above with `BUILD_NUMBER` input for mobile version names |
| `build-image.yaml` | Build image | Builds and pushes Docker image to GCR |
| `deploy-to-cluster.yaml` | Deploy to cluster | Helm upgrade/install on GKE; injects ConfigMap and SOPS-decrypted secrets |
| `deploy-static-cloudflare-page.yaml` | Deploy static to Cloudflare Page | Publishes a static build artifact to Cloudflare Pages via Wrangler |
| `static-site-export.yaml` | Static site export | Next.js static export step |
| `build-ios-app-staging.yaml` | build-ios-app-staging | Flutter iOS build → Firebase Distribution / TestFlight (staging) |
| `build-ios-app-prod.yaml` | build-ios-app-prod | Flutter iOS build → TestFlight (production) |
| `build-android-app-staging.yaml` | build-android-app-staging | Flutter Android build (staging) |
| `build-android-app-prod.yaml` | build-android-app-prod | Flutter Android build (production) |
| `backup-to-gcp.yaml` | Reusable - Backup to GCS | Database / file backup to Google Cloud Storage |
| `npm-publish.yaml` | Publish | Publish npm package via OIDC Trusted Publisher |
| `code-scanning.yaml` | Code Scanning | Bearer SAST security scan |
| `discord-notification.yaml` | Webhook on Event | Discord webhook notification on release events |
| `get-ci-environment.yaml` | Get ci environment | Resolves target environment name from branch/tag |
| `get-tag.yaml` | Get tag | Reads the latest git tag |
| `lastest-buildnumber-release.yaml` | Get latest build number from platform | Fetches latest iOS/Android build number from App Store / Play Store |
| `test.yaml` | Test | Generic test runner |

## Prerequisites

Consumer repos need:

- `GH_PAT` secret — a GitHub Personal Access Token with `repo` scope (used by semantic-release and submodule checkout)
- Secrets specific to the workflows they call (see `docs/env.md`)

## Usage — call a workflow from a consumer repo

```yaml
# .github/workflows/release.yaml
jobs:
  release:
    uses: LikeMeX/github-reuse-workflow/.github/workflows/semantic-release.yaml@main
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}

  deploy:
    needs: release
    if: needs.release.outputs.new_release_published == 'true'
    uses: LikeMeX/github-reuse-workflow/.github/workflows/deploy-to-cluster.yaml@main
    with:
      environment: dev
      docker-image: gcr.io/futureskillstaging/my-service
      docker-tag: ${{ needs.release.outputs.tag }}
      workload_identity_provider: projects/1097293104076/locations/global/workloadIdentityPools/...
      service_account: deployer@futureskillstaging.iam.gserviceaccount.com
      GKE_CLUSTER: fs-cluster-staging
      GKE_ZONE: asia-southeast1-b
      RELEASE_NAME: my-service
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

See `docs/runbook.md` for more snippets and `docs/env.md` for the full inputs/secrets reference.

## Environment pointer

Full inputs and secrets reference: `docs/env.md`

## Ownership

Infrastructure team — FutureSkill Engineering.

## Related repos

| Repo | Purpose |
| --- | --- |
| [fs-app-chart](https://github.com/LikeMeX/fs-app-chart) | Helm chart used by `deploy-to-cluster.yaml` |
| [terragrunt](https://github.com/LikeMeX/terragrunt) | Provisions the GKE clusters targeted by `deploy-to-cluster.yaml` |
| [fs-infrastructure](https://github.com/LikeMeX/fs-infrastructure) | Canonical architecture overview |
