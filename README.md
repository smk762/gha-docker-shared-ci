# Shared GitHub Actions CI/CD for Docker + Docker Compose (DockerHub)

This repo is intended to be a **central CI/CD “library” repo** that application repos can call via **reusable workflows** (`workflow_call`).

It implements a practical security pipeline aligned with the hardening checklist:
- Lint Dockerfile
- Validate compose files
- Build (Buildx)
- SBOM (optional)
- Vulnerability scan (Trivy) with fail thresholds
- Push to DockerHub on release
- Optional **keyless Cosign signing** (recommended over legacy Docker Content Trust)

## Why this repo exists
Instead of copy/pasting YAML across many repos, each app repo keeps a tiny wrapper workflow that calls the shared workflows here. Updates are made once and rolled out by bumping the ref (`@v1`, `@v1.1.0`, etc.).

---

## What you get

### Reusable workflows
- `.github/workflows/docker-ci.yml`
  - For PRs / non-release builds (no push)
  - Lint + compose validate + build + Trivy scan
- `.github/workflows/docker-release.yml`
  - For main/tag releases (push to DockerHub)
  - Lint + compose validate + build+push + Trivy scan
  - Optional Cosign signing (keyless)

### Composite actions
- `.github/actions/compose-validate`
- `.github/actions/dockerfile-lint`

---

## Secrets required in calling repos

In each application repo (the repo that calls these workflows), add:
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN` (DockerHub access token)

> DockerHub tokens are preferred over passwords.

If you enable Cosign keyless signing, you **do not need** a cosign private key. GitHub OIDC is used.

---

## How app repos use this

Example **PR CI** wrapper in an app repo:

```yaml
name: ci
on:
  pull_request:

jobs:
  docker_ci:
    uses: YOURUSER/gha-dockerhub-shared-ci/.github/workflows/docker-ci.yml@v1
    with:
      image: yourdockerhubuser/yourapp
      context: .
      dockerfile: Dockerfile
      compose_files: "compose.yml"
      trivy_fail_severity: "CRITICAL,HIGH"
```

Example **release** wrapper (push to DockerHub) in an app repo:

```yaml
name: release
on:
  push:
    branches: [main]
    tags:
      - "v*"

jobs:
  docker_release:
    uses: YOURUSER/gha-dockerhub-shared-ci/.github/workflows/docker-release.yml@v1
    with:
      image: yourdockerhubuser/yourapp
      context: .
      dockerfile: Dockerfile
      platforms: linux/amd64
      push_on_main: true
      push_on_tag: true
      sign_with_cosign: true
      trivy_fail_severity: "CRITICAL"
    secrets: inherit
```

---

## Tagging strategy
The release workflow can publish:
- `:sha-<shortsha>` (always)
- `:latest` (optional, usually main branch)
- `:<git tag>` (when building on a tag like `v1.2.3`)

---

## Notes on Docker Content Trust (DCT)
The original thread mentions Docker Content Trust (Notary v1). Docker has announced retirement of DCT/Notary v1 in the past; modern best practice is **Cosign/Sigstore** signing.

---

## Versioning / publishing this repo
When you commit this repo, create a tag like `v1` or `v1.0.0`. App repos should reference a tag, not `main`.

