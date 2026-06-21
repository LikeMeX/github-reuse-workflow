---
title: github-reuse-workflow Inputs & Secrets Reference
repo: github-reuse-workflow
layer: infra
tech: GH Actions
default_branch: main
updated: 2026-04-24
source_path: docs/env.md
---

## Inputs & Secrets Reference

### semantic-release.yaml

**Secrets**

| Secret | Required | Description |
| --- | --- | --- |
| `GH_PAT` | Yes | GitHub PAT with `repo` scope; used by semantic-release to push tags |

**Outputs**

| Output | Description |
| --- | --- |
| `tag` | New semver tag (e.g. `1.2.3`); empty if no release |
| `new_release_published` | `"true"` / `"false"` |
| `latest_tag` | Latest tag without leading `v` |

### semantic-release-mobile.yaml

Same as `semantic-release.yaml` plus:

**Inputs**

| Input | Required | Type | Description |
| --- | --- | --- | --- |
| `BUILD_NUMBER` | No | string | Appended to tag as `v${version}+${BUILD_NUMBER}` |

### build-image.yaml

**Inputs / Secrets** — see workflow file for the current parameter list (varies with registry target).

### deploy-to-cluster.yaml

**Inputs**

| Input | Required | Type | Default | Description |
| --- | --- | --- | --- | --- |
| `environment` | Yes | string | — | Target namespace / environment (`dev`, `production`) |
| `docker-image` | Yes | string | — | Full image path without tag |
| `docker-tag` | Yes | string | — | Image tag to deploy |
| `workload_identity_provider` | Yes | string | — | GCP Workload Identity Federation provider resource name |
| `service_account` | Yes | string | — | GCP service account to impersonate for cluster auth |
| `GKE_CLUSTER` | Yes | string | — | GKE cluster name |
| `GKE_ZONE` | Yes | string | — | GKE cluster zone |
| `RELEASE_NAME` | Yes | string | — | Helm release name |
| `map_env_to_config` | No | boolean | `true` | Whether to create a ConfigMap from `env/.env.<environment>` |

**Secrets**

| Secret | Required | Description |
| --- | --- | --- |
| `GH_PAT` | Yes | GitHub PAT for submodule checkout |

### deploy-static-cloudflare-page.yaml

**Inputs**

| Input | Required | Type | Description |
| --- | --- | --- | --- |
| `cloudflare-account-id` | Yes | string | Cloudflare account ID |
| `cloudflare-page-project-name` | Yes | string | Cloudflare Pages project name |
| `branch` | Yes | string | Branch to publish to (controls preview vs production URL) |

**Secrets**

| Secret | Required | Description |
| --- | --- | --- |
| `CLOUDFLARE_API_TOKEN` | Yes | Cloudflare API token with Pages edit permissions |

### static-site-export.yaml

**Inputs**

| Input | Required | Type | Description |
| --- | --- | --- | --- |
| `environment` | Yes | string | Target environment label |
| `tag-version` | Yes | string | Version tag being built |

### build-ios-app-staging.yaml / build-ios-app-prod.yaml

**Inputs**

| Input | Required | Type | Description |
| --- | --- | --- | --- |
| `LASTEST_VERSION_BUILD` | Yes | string | Build number (from `lastest-buildnumber-release.yaml`) |
| `LASTEST_VERSION_NAME` | Yes | string | Version name / semver tag |

**Secrets**

| Secret | Required | Description |
| --- | --- | --- |
| `FIREBASE_DISTRIBUTION_IOS_APP_ID` | Yes | Firebase App Distribution iOS App ID |
| `MATCH_KEYCHAIN_PASSWORD` | Yes | Fastlane Match keychain password |
| `ASC_KEY_ID` | Yes | App Store Connect API key ID |
| `ASC_ISSUER_ID` | Yes | App Store Connect issuer ID |
| `ASC_KEY_P8` | Yes | App Store Connect private key (P8 content) |
| `MATCH_GIT_BASIC_AUTHORIZATION` | Yes | Base64-encoded `user:PAT` for Match certificate repo |

### build-android-app-staging.yaml / build-android-app-prod.yaml

See workflow file for current inputs. Typically requires Firebase credentials and keystore secrets.

### backup-to-gcp.yaml

**Inputs / Secrets** — uses GCP Workload Identity; see workflow file for current parameter list.

### npm-publish.yaml

No extra secrets — uses OIDC Trusted Publisher (`npm publish --registry=https://registry.npmjs.org/`). Requires the npm package to have a configured Trusted Publisher in the npm registry UI.

**Secrets**

| Secret | Required | Description |
| --- | --- | --- |
| `GH_PAT` | Yes | For checkout and semantic-release tag push |

### code-scanning.yaml

No inputs or secrets beyond default `GITHUB_TOKEN` (injected automatically by GitHub Actions).

### discord-notification.yaml

Uses `workflow_call` with no explicit inputs — reads event context from the calling workflow. Requires a Discord webhook URL configured as a repository variable or secret (see workflow file).
