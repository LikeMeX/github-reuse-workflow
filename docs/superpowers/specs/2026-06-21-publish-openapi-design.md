# Publish OpenAPI to GCS — Design Spec

**Date:** 2026-06-21  
**Status:** Approved

## Goal

Add a reusable GitHub Actions workflow that generates a NestJS OpenAPI spec in CI and uploads it to GCS at a fixed path per environment and service.

## Requirements

- **Trigger:** `workflow_call` only — invoked from consumer pipelines (typically after test/build).
- **Generation:** NestJS via `@nestjs/swagger`; caller provides `generate-command` (e.g. `pnpm openapi:generate`).
- **Destination:** `gs://<bucket>/<env>/<service>/openapi.json` (overwrite on each run).
- **Auth:** GCP Workload Identity (same pattern as `build-image.yaml`, `backup-to-gcp.yaml`).
- **Optional build:** Caller may pass `build-command`; skip when empty.

## Workflow Inputs

| Input | Required | Default | Purpose |
|-------|----------|---------|---------|
| `environment` | yes | — | GCS path segment; used for `env/.env.<environment>` if present |
| `service-name` | yes | — | GCS path segment (e.g. `user-api`) |
| `gcs-bucket` | yes | — | Bucket name without `gs://` |
| `generate-command` | yes | — | Shell command to produce the spec |
| `openapi-output-path` | no | `openapi.json` | Local file to upload |
| `build-command` | no | `''` | Run before generate when non-empty |
| `node-version` | no | `18` | Node.js version |
| `pnpm-version` | no | `8` | pnpm version |
| `workload_identity_provider` | yes | — | WIF provider |
| `service_account` | yes | — | GCP SA with GCS write |
| `project_id` | no | — | GCP project ID |

## Job Flow

1. Checkout repository
2. Setup pnpm + Node
3. `pnpm i --frozen-lockfile`
4. Copy `env/.env.<environment>` → `.env` when file exists
5. Run `build-command` when non-empty
6. Run `generate-command`
7. Fail if `openapi-output-path` missing
8. Authenticate to GCP
9. Upload to `<gcs-bucket>/<environment>/<service-name>/`

## Permissions

- `contents: read`
- `id-token: write`

## Consumer Contract

Each NestJS service must expose a generate script that bootstraps the app and writes JSON via `SwaggerModule.createDocument`, for example:

```typescript
const document = SwaggerModule.createDocument(app, config);
writeFileSync('openapi.json', JSON.stringify(document, null, 2));
```

## Out of Scope

- Upload-only or artifact-based variants
- Versioned paths by tag/SHA (fixed path overwrite only)
- Non-pnpm package managers

## Reference Workflows

- `.github/workflows/backup-to-gcp.yaml` — GCS upload pattern
- `.github/workflows/static-site-export.yaml` — pnpm + env file pattern
- `.github/workflows/test.yaml` — pnpm install pattern
