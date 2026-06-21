# Publish OpenAPI to GCS Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add reusable workflow `publish-openapi.yaml` that generates NestJS OpenAPI JSON and uploads to `gs://<bucket>/<env>/<service>/openapi.json`.

**Architecture:** Single `workflow_call` job mirroring `backup-to-gcp.yaml` upload auth and `static-site-export.yaml` Node setup. Caller owns NestJS bootstrap via `generate-command`.

**Tech Stack:** GitHub Actions, pnpm, Node 18, `google-github-actions/auth@v3`, `google-github-actions/upload-cloud-storage@v2`

## Global Constraints

- GCS path: `<gcs-bucket>/<environment>/<service-name>/openapi.json`
- Trigger: `workflow_call` only
- Default `openapi-output-path`: `openapi.json`
- Default `pnpm-version`: `8`, `node-version`: `18`
- Permissions: `contents: read`, `id-token: write`

---

### Task 1: Add publish-openapi workflow

**Files:**
- Create: `.github/workflows/publish-openapi.yaml`

**Interfaces:**
- Produces: reusable workflow callable as `org/github-reuse-workflow/.github/workflows/publish-openapi.yaml@<ref>`

- [ ] **Step 1:** Create workflow with all inputs from spec
- [ ] **Step 2:** Implement pnpm install, optional env copy, optional build, generate, verify file, GCP auth, GCS upload
- [ ] **Step 3:** Validate YAML structure (`actionlint` if available, or manual review)
