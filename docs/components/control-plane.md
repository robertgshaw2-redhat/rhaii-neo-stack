# Component: Control Plane

**Source**: Neo Stack.

The control plane is the set of services and stores that hold the
authoritative state about tenants, projects, keys, endpoints, catalog,
prices, and audit. It is **not** in the request hot path. The hot path
is the gateway + data plane; the control plane is consulted by them
through narrow, cached interfaces.

## Services

### Tenancy service

- Owns the Org → Project → User hierarchy.
- Federates identity to an upstream IdP via OIDC (Keycloak as default;
  Okta, Auth0, Azure AD via reference integrations).
- SCIM at GA.
- RBAC over five built-in roles plus custom roles at GA.
- Emits identity events to audit.

Talks to: maas-api (for API key issuance / revocation), audit, console.

### Catalog service

- Holds the model catalog: SKUs, versions, accelerator profiles,
  benchmark sheets, model cards, defaults.
- Holds the **effective price book** per (org, project, SKU) — a small
  rules engine over (default price → org override → project override
  → promo code).
- Source of truth for which SKUs a project can call. The gateway reads
  this through an authoritative-cache pattern (5–30s TTL).

Talks to: maas-controller (to materialize KServe / llm-d configs),
console, billing.

### Endpoint-lifecycle controller

The Dedicated Endpoints product line lives here. The controller:

- Watches `Endpoint` CRDs (one per Dedicated Endpoint).
- Synthesizes KServe `InferenceService` + llm-d / IGW config + Kuadrant
  policy + Authorino AuthConfig + Limitador rules + per-endpoint
  metrics scrape configs.
- Tracks status, readiness, scale, replicas.
- Emits endpoint lifecycle events to audit.
- Handles LoRA adapter registration and routing.

Reconciles approximately every 30s and on event.

### Billing service

- Owns invoices, credits, payment methods (proxied), prepaid balances,
  spend caps.
- Owns the integration with Stripe / Metronome / Orb.
- Exposes APIs to consoles (read-only) and to the spend-cap controller
  (read of caps).
- Receives priced events from the metering pipeline.

### Audit service

- Append-only.
- Schema is small and stable: `(ts, actor, tenant, action, target, before, after, source_ip)`.
- Sink: local Postgres + optional sink to the neocloud's SIEM.
- Retention configurable; default 13 months.

## Stores

| Store | Used for | Notes |
|---|---|---|
| Postgres (primary) | Tenancy, catalog, billing, audit, endpoint metadata | One logical DB per service; one physical cluster acceptable for MVP, separable for scale. |
| ClickHouse | Metered events, aggregations, analytics | Per-tenant partitioned. |
| NATS JetStream / Redpanda | Event bus for metering, audit fan-out, endpoint reconciliation events | Choose one; we recommend NATS JetStream for OpenShift parity. |
| Object storage (S3-compatible) | Model artifacts, LoRA blobs, batch inputs/outputs, audit archives | Neocloud-provided. |

## APIs

The control plane exposes two API surfaces:

- **Admin API** for the operator console and for the neocloud's internal
  tooling. Authenticated by operator-tier tokens. Surface in
  [`api/admin-api.md`](../api/admin-api.md).
- **Tenant API** for tenant-facing console operations (create key,
  rotate key, create endpoint, view usage). Authenticated by user
  tokens.

The end-customer-facing inference API (chat / completions / etc.) is
**not** part of the control plane. It's gateway → data plane.

## Multi-tenancy & isolation

- One **Postgres schema** per logical tenant boundary at the service
  level — not per end customer. Per-end-customer rows live in shared
  tables with `tenant_id` predicates enforced at the service layer.
- Audited row-level security in Postgres on tables with PII / billing
  data (GA).
- Encryption at rest from the neocloud's storage; per-tenant KMS keys
  for sensitive fields at GA.
- Strict service-account boundaries; control-plane services do not
  share credentials.

## State migrations

The product evolves and the schema evolves with it. Migration rules:

- Forward-compatible: any new release accepts the previous release's
  data.
- Backward-compatible reads: any read path tolerates either schema
  version for one minor-release window.
- All migrations are auditable.
- We ship a `neostackctl` CLI with `migrate plan` / `migrate apply` /
  `migrate verify`. No raw `psql` for upgrades.

## Why this isn't just a CRUD app

Three things make the control plane non-trivial:

1. **Authoritative-cache interaction with the hot path.** The gateway
   cannot do a Postgres round-trip per request. We push read-models
   into Authorino / Limitador via maas-controller and refresh on
   TTL + invalidate-on-change.
2. **Billing correctness.** Every priced event must be deterministic,
   reproducible from the raw event, and idempotent. Edge cases (late
   arrivals, retries, partial outages) all need defined behavior.
3. **Multi-tenant operator UX.** The operator can act *as* a tenant
   for support purposes. That requires impersonation that is fully
   audited and that does not bypass billing.

These are the failure modes that bite in-house builds, which is why
they take 18 months. We start with an explicit design.
