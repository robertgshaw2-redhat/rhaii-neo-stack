# Multi-Region & DR

## MVP

**One region. Single cluster. Manual DR.**

We design the schema and the service boundaries with multi-region in
mind — no service holds region-pinned state in places that block a
future split — but we do not ship multi-region active/active at MVP.

DR for MVP:

- Daily logical backups of Postgres + ClickHouse to object storage.
- Point-in-time recovery enabled (WAL shipping for Postgres).
- Restore-runbook tested at install time.
- RPO: 1 hour, RTO: 4 hours, in-region same-cluster.
- Cross-region: customer manually restores from object storage to a new
  cluster.

## GA

**Active / passive with explicit failover.**

- Two clusters; control-plane state replicates via logical replication
  (Postgres) and Kafka MirrorMaker / NATS-leaf or ClickHouse replication.
- Gateway is stateless and runs in both regions; DNS-level failover.
- Inference data planes are independent — each region serves its own
  customers.
- Catalog SKUs are replicated; per-cluster SKU enablement is local.
- RPO: 5 minutes, RTO: 30 minutes.

## v2

**Active / active control plane with regional inference.**

- Tenants are assigned a **home region** for control-plane purposes.
- Inference requests can be served from any region the tenant has
  endpoints in; routing is at the edge.
- Per-tenant data-residency tags enforced (a tenant tagged
  `residency: EU` cannot have endpoints provisioned outside the EU).
- Conflict-resolution for control-plane writes is last-writer-wins on
  most resources; idempotent writes for audit + priced events.

## v3

**Federated multi-cloud.**

- One Neo Stack control plane can manage data planes in multiple
  neocloud regions or multiple neoclouds entirely.
- Useful for sovereign deployments where a national champion neocloud
  partners with a hyperscaler-region for burst capacity.
- Requires cross-cluster trust establishment, federated catalogs, and
  cross-cluster metering reconciliation.

## What "region" means

For Neo Stack purposes, a region is:

- A single Kubernetes cluster (or a tight HA cluster pair).
- A single Postgres primary + replicas.
- A single ClickHouse cluster.
- A single object-storage realm.

A region is the unit of DR, the unit of data-residency, and the unit of
operator on-call rotation.

## Cross-region considerations

- **Latency**: inference traffic stays in-region; cross-region is only
  control plane and replication.
- **Data residency**: enforced at endpoint creation. The operator can
  pin certain models to certain regions for licensing reasons.
- **Billing**: invoices roll up across regions per tenant. The control
  plane that owns the tenant's home region runs the billing job.
- **Identity**: federated via the upstream IdP; Neo Stack does not
  duplicate identity.

## Failure modes

| Failure | Behavior (MVP) | Behavior (GA) | Behavior (v2) |
|---|---|---|---|
| AZ down | Region degraded; auto-recover via K8s. | Same. | Same. |
| Region down | Manual DR (4h RTO). | Failover to passive (30m RTO). | Other regions continue. Tenant pinned to lost region needs failover. |
| Control-plane Postgres down | Auth caches hold 60s; inference continues with degraded admin. | Replica promotes. | Same; per-region. |
| Inference cluster down | Customer requests fail. | Same. | Other-region endpoints continue serving that tenant. |
| Cross-region replication broken | n/a | Diverges; alert; reconcile manually. | Reconcile by priced-event idempotency. |

## What we are deliberately not doing

- **In-flight request migration across regions.** Streaming connections
  drop on failover; clients retry. We do not attempt to relocate active
  KV state across the WAN.
- **Synchronous cross-region writes for control-plane data.** Too slow
  and too fragile. Logical replication with eventual consistency only.
- **A custom multi-region database.** We pick managed Postgres (or
  CloudNativePG) and ClickHouse with their native replication. No
  bespoke engineering on the storage layer.
