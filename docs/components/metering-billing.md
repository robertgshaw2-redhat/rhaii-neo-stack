# Component: Metering & Billing

**Source**: Neo Stack.

The metering & billing plane is where Neo Stack does the work that the
upstream `models-as-a-service` project does not: turn measured usage
into priced events, into invoices, into money in the neocloud's bank
account.

This is the highest-stakes part of Neo Stack. A 0.1% billing error
across a year of tokens can be a six-figure dispute. Get this right.

## Two stages

### Stage 1: Metering (raw events → priced events)

Inputs:

- **Token events** from the Model Gateway (Kuadrant tap). Per-request,
  with input / output / cached token counts.
- **GPU-second events** from KServe / llm-d. Per-replica, per-second
  accounting of in-use GPU time. Aggregated to 60s rollups before
  ingestion.

Pipeline:

```
gateway tap ──► NATS JetStream ──► ingest worker ──► Postgres (hot, 7d)
                       │                          └─► ClickHouse (cold, multi-year)
                       │
KServe scraper ────────┘
```

Properties:

- **Idempotent.** Every event has a `request_id` (token events) or
  `(endpoint_id, second_bucket)` (GPU events). Duplicate ingest is
  a no-op.
- **At-least-once delivery.** Acks happen after write. Dedup is the
  ingest worker's job.
- **Late arrivals tolerated up to 24h.** Events older than 24h are
  rejected to bound retroactive billing changes.
- **Replayable.** All raw events archived to object storage daily;
  ClickHouse can be rebuilt.

Pricing happens at ingest:

- Look up the effective price for `(tenant, project, sku, ts)` from
  the catalog service.
- Compute priced event in micro-cents.
- Persist both raw and priced events; never throw away the raw.

### Stage 2: Billing (priced events → invoices)

Inputs:

- Priced events from Stage 1.
- Per-tenant payment methods, credit balances, spend caps, billing
  contact info.

Outputs:

- **Per-tenant invoices** generated nightly (configurable cadence).
- **Spend updates** to the spend-cap controller every minute.
- **Real-time usage** exposed in the developer console (with a
  documented "billing-final" cutoff at end of period).
- **Reconciliation export** to the neocloud's accounting system.

## Connectors

| Connector | MVP | GA | Notes |
|---|---|---|---|
| Stripe | ✅ | ✅ | Postpaid invoices. Cards + ACH. |
| Stripe Billing usage records | ✅ | ✅ | Push priced events; Stripe generates invoice. |
| Generic CSV / S3 export | ✅ | ✅ | One file per tenant per day; idempotent. |
| Metronome | — | ✅ | Tighter integration; usage-based billing. |
| Orb | — | ✅ | Same. |
| NetSuite / SAP | — | partner | Through Metronome / Orb. |

Each connector is a stateless adapter that reads from a canonical
"priced events to export" view and writes to its target system. The
connector tracks its own cursor in Postgres so it can resume cleanly.

## Spend caps

The closed loop is critical. Spend cap logic lives here:

1. Every minute, the spend-cap evaluator queries near-real-time
   priced-event totals per (key, project, org).
2. Compares against configured caps (lifetime, monthly, daily).
3. Decides: nothing, warn, throttle, block.
4. Writes Limitador rules through the gateway control loop.
5. Optionally pages the tenant's billing contact.

We document explicitly that **caps are not transactional with the
inference call** — a customer can exceed their cap by a small amount
during the 60-second evaluation window. The cap is a soft limit; the
operator may charge or waive the overage. Hard kills happen on a
secondary, slower loop with a coarser daily backstop.

## Edge cases we design for

- **Refunds and credits** flow through the same priced-event store,
  as negative-signed events with an explicit "adjustment" type and a
  reason code.
- **Out-of-band invoice adjustments** (the operator says "give this
  customer $500 in credit") are recorded as adjustments in the same
  schema and feed the same exports.
- **Plan changes mid-period.** A tier change applies prospectively. We
  always reprice off the effective-as-of timestamp.
- **Mid-period price changes.** Same — priced events reprice off the
  price in effect at event time.
- **Tenant offboarding.** Stops new requests. Existing usage continues
  to invoice for the period. Data retained per the agreed retention.

## Schema sketch (priced events, simplified)

```sql
create table priced_events (
  event_id        uuid primary key,
  source_event_id text not null,            -- request_id or (endpoint+bucket)
  source_kind     text not null,            -- 'token' | 'gpu_second' | 'adjustment'
  ts              timestamptz not null,     -- event time, not ingest time
  ingested_at     timestamptz not null,
  org_id          uuid not null,
  project_id      uuid not null,
  api_key_id      uuid,
  sku_id          uuid not null,
  endpoint_id     uuid,
  quantity        numeric not null,         -- tokens or GPU-seconds
  quantity_kind   text not null,            -- 'input_tokens' | 'output_tokens' | ...
  unit_price      numeric not null,         -- in micro-cents
  price_book_ver  bigint not null,
  amount          numeric not null,         -- quantity * unit_price, micro-cents
  metadata        jsonb,
  unique (source_event_id, quantity_kind)
);
```

Aggregations live in ClickHouse; this is the canonical reconcile source.

## Performance budget

- Ingest: 50k events/sec sustained, 200k/sec burst, per cluster.
- Spend update lag p99: < 60s end-to-end gateway → Limitador.
- Invoice generation: complete for a 10M-event tenant in < 10 minutes.
- Reconciliation pass against Stripe: nightly, never longer than 60 min.

## What we do **not** do

- We are not a billing platform. We integrate with one.
- We do not handle payment processing directly.
- We do not handle tax. Stripe / Metronome / Orb / the neocloud's
  finance team does.
- We do not produce GAAP reports. We export the events needed to
  produce them.

## SOC 2 implications

The metering/billing path is the most audit-relevant path in the
product. Design from day one for:

- Immutable raw-event store.
- Access logging on priced-events and adjustments.
- Quarterly reconciliation reports.
- Defined retention.
- Documented data flow from gateway tap to invoice line item.
