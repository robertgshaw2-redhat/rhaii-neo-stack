# Admin API

The Admin API is the control-plane API used by the developer console,
the operator console, and the neocloud's internal automation. It is
**not** OpenAI-compatible. Its design center is operator and tenant
management, not inference.

## Authentication

Two kinds of caller:

- **Tenant tokens** — issued by the IdP to a user; scoped to their org/projects.
- **Operator tokens** — issued to neocloud staff; carry the
  `neostack:operator` scope and can read across all tenants.

We do **not** allow customer API keys (`sk-…`) on the Admin API. Two
separate auth contracts.

## Versioning

`/admin/v1/...`. Same stability guarantees as the OpenAI-compatible
API. Migration paths documented per major.

## Resources (sketch)

```
GET    /admin/v1/orgs
POST   /admin/v1/orgs
GET    /admin/v1/orgs/{org_id}
PATCH  /admin/v1/orgs/{org_id}
DELETE /admin/v1/orgs/{org_id}

GET    /admin/v1/orgs/{org_id}/projects
POST   /admin/v1/orgs/{org_id}/projects
…

GET    /admin/v1/projects/{project_id}/api-keys
POST   /admin/v1/projects/{project_id}/api-keys
DELETE /admin/v1/projects/{project_id}/api-keys/{key_id}

GET    /admin/v1/projects/{project_id}/usage
GET    /admin/v1/projects/{project_id}/invoices
GET    /admin/v1/projects/{project_id}/spend-caps
PUT    /admin/v1/projects/{project_id}/spend-caps/{kind}

GET    /admin/v1/endpoints
POST   /admin/v1/endpoints
GET    /admin/v1/endpoints/{endpoint_id}
PATCH  /admin/v1/endpoints/{endpoint_id}
DELETE /admin/v1/endpoints/{endpoint_id}
POST   /admin/v1/endpoints/{endpoint_id}/adapters

GET    /admin/v1/catalog
GET    /admin/v1/catalog/{sku_id}
PATCH  /admin/v1/catalog/{sku_id}                # operator only
POST   /admin/v1/catalog/{sku_id}/enable          # operator only
POST   /admin/v1/catalog/{sku_id}/disable         # operator only

GET    /admin/v1/pricing/{project_id}             # effective prices

GET    /admin/v1/audit                             # operator only
GET    /admin/v1/metrics/...                       # operator only, narrow set
```

## Idempotency

All non-GET endpoints accept `Idempotency-Key` and dedupe on it for 24h.

## Pagination

Cursor-based:

```
GET /admin/v1/endpoints?limit=50&after=<cursor>

→
{
  "data": [...],
  "next": "<cursor or null>"
}
```

## Rate limits

Admin API has its own rate limits, separate from the inference API:

- Tenant tokens: 60 RPM per user, 600 RPM per project.
- Operator tokens: 600 RPM per operator user.

Limits configurable per environment.

## Webhooks (GA)

The Admin API supports webhooks for:

- Invoice generated.
- Spend cap warning / threshold.
- Endpoint state change.
- Adapter uploaded.
- Audit event in subscribed categories.

Webhooks are signed with a shared secret per registration; we ship a
verification helper in the developer SDKs.

## OpenAPI

We ship an OpenAPI 3.1 spec for every release. Generated SDKs in Go,
Python, TypeScript, Java.

## What is **not** in the Admin API

- The inference verbs (chat, completions, embeddings). Those are the
  OpenAI-compatible API.
- Long-running mutations beyond endpoint creation. The endpoint
  creation flow is the only one that can take minutes; everything else
  returns < 1s or is rejected.
- Raw access to priced-events. Only summarized usage / invoices.
