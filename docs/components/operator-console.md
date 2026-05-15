# Component: Operator Console

**Source**: Neo Stack.

The operator console is what the **neocloud's own team** uses to run
the Neo Stack platform. Internal, never end-customer-facing. Branded
with Neo Stack + the neocloud's name; not branded as Red Hat.

## Users

- VP / Head of Product: revenue and adoption.
- Platform Engineering: capacity, health.
- SRE on-call: incidents, kill switches.
- Solutions Engineering: tenant-facing support.
- Finance / RevOps: revenue and reconciliation.

(See [`personas.md`](../personas.md) for full personas.)

## Pages

### Overview / home

- Live KPIs: requests/sec, tokens/min, revenue today, GPUs in use,
  active tenants, top model, top endpoint.
- Incident banner if anything is on fire.
- Catalog updates available banner.

### Tenants

- List with filters (plan tier, region, spend bucket, signup date,
  status).
- Drill-in:
  - Members, roles, recent logins.
  - Keys (no secrets shown; revocation possible).
  - Endpoints owned.
  - Recent usage and spend.
  - Invoices and payment status.
  - Audit log of changes.
  - **Impersonate as** (audited; opens read-only tenant console view).
  - Kill switch (suspend tenant; audit-tracked).

### Capacity

- Per-pool utilization: MaaS pools, DE reservations.
- Per-accelerator GPU usage and headroom.
- Queue depth heat map.
- Cluster events affecting capacity.
- Capacity forecast (GA): expected utilization in 7 / 30 days from
  trend + scheduled endpoints.

### Catalog

- List SKUs with status (Published, Hidden, Deprecated).
- Enable / disable an SKU for this cluster.
- Per-SKU pricing override editor.
- Per-SKU adoption (tenants using, revenue, requests).
- "What changes if I bump price by N%" projection.

### SLO

- Per-SKU and per-endpoint SLO dashboards.
- Recent SLO breaches with affected tenants.
- Auto-credit projections.
- Trigger an SLO incident review.

### Revenue

- Revenue by tenant, by tier, by model class, by line (MaaS vs DE).
- Cohort retention by signup month.
- Forecast vs. actual.
- Stripe / Metronome / Orb reconciliation status (green / yellow /
  red on data freshness).

### Incidents

- Active incidents.
- On-call assignments.
- Recent post-mortems.
- Status-page sync state.

### Audit

- Search by actor / target / action / date.
- Export for SOC 2 evidence collection.

### Settings

- Operator org users + roles.
- Branding configuration (logo, colors, custom domain).
- Connector configuration (IdP, Stripe, Datadog, etc.).
- Catalog auto-update policy.

## What it does **not** show

- Per-request inputs and outputs by default. PII risk. Operator can
  pull individual records via the audit-log-tracked "support fetch"
  flow, which logs the operator's identity, the reason, and notifies
  the affected tenant within 24h (configurable).
- Detailed billing internals (priced events table) without an admin
  role.
- Anything that would let an operator change a tenant's data without
  audit.

## Why operator-console matters more than people think

Operators choose platforms by trying to support a difficult customer
on a Friday afternoon. If the console can't tell them in 30 seconds
"why is this tenant's TTFT bad on the 70B model right now," they will
write a runbook around it once and then call us every time.

The console is therefore designed for **diagnosis**, not just for
display:

- Every metric in every chart is clickable into the underlying time
  series.
- Every tenant row links to a "support page" that aggregates their
  recent requests, errors, capacity issues, and recent platform events.
- Tenant impersonation is one click; reading is read-only by default;
  writes require a typed reason.
- Kill-switch and drain are explicit and reversible.
