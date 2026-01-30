# Security checklist (CI gate)

Your CI should fail if these are violated (where feasible):

- [ ] Container runs as non-root (Dockerfile `USER` or Compose `user:`)
- [ ] Minimal base image (alpine/slim/distroless) where feasible
- [ ] No secrets baked into images (no keys in Dockerfile / layers)
- [ ] Image scanned (Trivy) and gates enforced
- [ ] Resource limits set (Compose deploy or runtime flags)
- [ ] Read-only filesystem where possible + tmpfs for writable paths
- [ ] Capabilities dropped (cap_drop: [ALL]) and add back only required
- [ ] No privileged containers
- [ ] no-new-privileges enabled
- [ ] Healthchecks configured
- [ ] Networks segmented (frontend/backend/db)
- [ ] Keep base images and dependencies updated (Dependabot/Renovate)
