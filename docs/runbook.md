---
title: github-reuse-workflow Runbook
repo: github-reuse-workflow
layer: infra
tech: GH Actions
default_branch: main
updated: 2026-04-24
source_path: docs/runbook.md
---

## Runbook — github-reuse-workflow Operations

### Add a new reusable workflow

1. Create `.github/workflows/<name>.yaml` in this repo.
2. Set the trigger to `workflow_call` only:

```yaml
on:
  workflow_call:
    inputs:
      my_input:
        required: true
        type: string
    secrets:
      MY_SECRET:
        required: true
```

3. Document inputs and secrets in `docs/env.md`.
4. Document the new workflow in `docs/consumers.md` and `README.md`.
5. Commit, push, and tag a new version (see below).

### Version / tag a workflow release

This repo uses semantic-release. Commit messages must follow Conventional Commits:

```
feat: add npm-publish workflow          # → minor bump
fix: pin wrangler-action to v2.0.0      # → patch bump
feat!: rename deploy input 'env'        # → major bump (breaking change)
```

Push to `main` — `semantic-release.yaml` (if wired up as a self-referencing workflow) will publish the tag automatically. To tag manually:

```bash
git tag v1.2.3
git push origin v1.2.3
```

Consumer repos should pin to a tag, not `@main`, for stability:

```yaml
uses: LikeMeX/github-reuse-workflow/.github/workflows/deploy-to-cluster.yaml@v1.2.3
```

### Consumer usage — call semantic-release

```yaml
jobs:
  release:
    uses: LikeMeX/github-reuse-workflow/.github/workflows/semantic-release.yaml@main
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

### Consumer usage — call deploy-to-cluster

```yaml
jobs:
  deploy-dev:
    uses: LikeMeX/github-reuse-workflow/.github/workflows/deploy-to-cluster.yaml@main
    with:
      environment: dev
      docker-image: gcr.io/futureskillstaging/my-service
      docker-tag: ${{ needs.release.outputs.tag }}
      workload_identity_provider: projects/1097293104076/locations/global/workloadIdentityPools/gh-pool/providers/gh-provider
      service_account: deployer@futureskillstaging.iam.gserviceaccount.com
      GKE_CLUSTER: fs-cluster-staging
      GKE_ZONE: asia-southeast1-b
      RELEASE_NAME: my-service
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }}
```

### Consumer usage — Cloudflare Pages deploy

```yaml
jobs:
  deploy-pages:
    uses: LikeMeX/github-reuse-workflow/.github/workflows/deploy-static-cloudflare-page.yaml@main
    with:
      cloudflare-account-id: ${{ vars.CF_ACCOUNT_ID }}
      cloudflare-page-project-name: my-site
      branch: main
    secrets:
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

### Consumer usage — iOS staging build

```yaml
jobs:
  build-ios:
    uses: LikeMeX/github-reuse-workflow/.github/workflows/build-ios-app-staging.yaml@main
    with:
      LASTEST_VERSION_BUILD: ${{ needs.buildnumber.outputs.build_number }}
      LASTEST_VERSION_NAME: ${{ needs.release.outputs.tag }}
    secrets:
      FIREBASE_DISTRIBUTION_IOS_APP_ID: ${{ secrets.FIREBASE_DISTRIBUTION_IOS_APP_ID }}
      MATCH_KEYCHAIN_PASSWORD: ${{ secrets.MATCH_KEYCHAIN_PASSWORD }}
      ASC_KEY_ID: ${{ secrets.ASC_KEY_ID }}
      ASC_ISSUER_ID: ${{ secrets.ASC_ISSUER_ID }}
      ASC_KEY_P8: ${{ secrets.ASC_KEY_P8 }}
      MATCH_GIT_BASIC_AUTHORIZATION: ${{ secrets.MATCH_GIT_BASIC_AUTHORIZATION }}
```

### Debugging a failing workflow

1. Check the Actions tab of the consumer repo for the run log.
2. Confirm secrets are set in the consumer repo's Settings → Secrets.
3. To run the workflow locally for testing, use [act](https://github.com/nektos/act):

```bash
act workflow_call -W .github/workflows/deploy-to-cluster.yaml
```

4. To test a change to a reusable workflow before merging, reference a branch in the consumer:

```yaml
uses: LikeMeX/github-reuse-workflow/.github/workflows/deploy-to-cluster.yaml@feature/my-fix
```
