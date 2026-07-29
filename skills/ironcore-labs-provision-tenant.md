---
name: Provision a SaaS Shield tenant with customer-managed keys
description: >-
  Onboard a new customer (tenant) in IronCore SaaS Shield via the Vendor API
  Bridge: create a KMS configuration, create the tenant, assign the KMS config,
  and invite a tenant admin.
api: openapi/ironcore-labs-vendor-bridge-openapi.yml
operations:
  - post-kms-configs
  - post-tenants-v2
  - post-kms-configs-kmsConfigId-tenants-tenantId
  - post-tenants-tenantId-invite
  - get-tenants-tenantId
---

# Provision a SaaS Shield tenant

Use the self-hosted **Vendor API Bridge**. Every request carries the header
`Authorization: vab:1:<API_KEY>`. Errors come back as `{ "message", "statusCode" }`
JSON (see `errors/ironcore-labs-problem-types.yml`). There is no idempotency-key
contract, so guard retries yourself.

## Steps

1. **Create the KMS configuration** — `post-kms-configs` (`POST /1/kms/configs`).
   Provide the customer-managed key details for one of AWS, Azure, GCP, or Thales
   KMS. Capture the returned `KmsConfigId`.
2. **Create the tenant** — `post-tenants-v2` (`POST /2/tenants`). Capture the
   returned `TenantId`.
3. **Assign the KMS config to the tenant** — `post-kms-configs-kmsConfigId-tenants-tenantId`
   (`POST /1/kms/configs/{kmsConfigId}/tenants/{tenantId}`) using the two ids
   from steps 1-2.
4. **Invite a tenant admin** — `post-tenants-tenantId-invite`
   (`POST /1/tenants/{tenantId}/invite`) so the customer can manage their own keys.
5. **Verify** — `get-tenants-tenantId` (`GET /1/tenants/{tenantId}`) and confirm
   the tenant exists with its assignment.

## Notes
- A tenant and a KMS config form a many-to-many assignment (`data-model/ironcore-labs-data-model.yml`).
- Since v2.0.1, `get-kms-configs-kmsConfigId` no longer returns decrypted credentials.
