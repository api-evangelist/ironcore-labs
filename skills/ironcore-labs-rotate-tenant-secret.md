---
name: Rotate a tenant secret
description: >-
  Rotate a SaaS Shield tenant's secret safely using the Vendor API Bridge
  begin/commit rotation flow, then confirm the tenant's assignments.
api: openapi/ironcore-labs-vendor-bridge-openapi.yml
operations:
  - get-tenants-tenantId
  - post-1-tenants-tenantId-secrets-rotate-begin
  - post-1-tenants-tenantId-secrets-rotate-commit
  - get-tenants-tenantId-assignments
---

# Rotate a tenant secret

Secret rotation is a two-phase begin/commit operation so the old secret stays
valid until the new one is committed. Auth header: `Authorization: vab:1:<API_KEY>`.

## Steps

1. **Confirm the tenant** — `get-tenants-tenantId` (`GET /1/tenants/{tenantId}`).
2. **Begin rotation** — `post-1-tenants-tenantId-secrets-rotate-begin`
   (`POST /1/tenants/{tenantId}/secrets/rotate/begin`). This provisions the new
   secret in a pending state (`TenantSecretRotationStatus`).
3. **Commit rotation** — `post-1-tenants-tenantId-secrets-rotate-commit`
   (`POST /1/tenants/{tenantId}/secrets/rotate/commit`) once re-encryption is done.
4. **Verify assignments** — `get-tenants-tenantId-assignments`
   (`GET /1/tenants/{tenantId}/assignments`) to confirm the tenant's KMS
   assignments are intact.

## Notes
- Do not delete the KMS config mid-rotation; from v3.1.0 configs with active
  secrets can be deleted for destructive offboarding, which will orphan secrets.
- No idempotency key exists — do not blindly retry `commit`; re-check state first.
