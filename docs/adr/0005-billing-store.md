# ADR-0005: Postgres + ClickHouse for metering and billing

## Status

Accepted.

## Context

Metering and billing have three hot needs:

1. **High-volume, append-mostly event ingest** (token events, GPU-second
   rollups). Tens of thousands of events per second per cluster.
2. **Multi-year retention** for audit, with fast aggregations
   ("what was project X's spend on 2025-10-15?").
3. **Low-latency lookups** for spend caps and current-period totals.

No single store optimally serves all three.

Alternatives considered:

- Postgres only: ingest scales to ~50k events/sec with effort but
  multi-year analytical queries crawl.
- ClickHouse only: incredible analytics, but transactional integrity
  on small operational records (tenants, invoices) is awkward.
- TimescaleDB: good middle ground; we are wary of taking on a
  proprietary extension when the open alternative (ClickHouse) is
  excellent.
- DynamoDB / Spanner / Bigtable: cloud-locked.
- A specialized metering service (M3, Cube, Hydrolix): another moving
  part we don't need; we own the data.

## Decision

We use **Postgres for operational and hot data**, and **ClickHouse for
cold and analytical data**:

- **Postgres**: tenants, projects, keys, endpoints, invoices, credits,
  spend caps, audit (operational tier), priced-events hot window (7
  days), state for connectors.
- **ClickHouse**: priced-events cold tier (multi-year), aggregations,
  analytics, operator-console reporting queries.

The metering pipeline writes to both: Postgres for the hot window,
ClickHouse for the cold store.

## Consequences

- Two stores to operate. Acceptable; both are well-understood and
  both have managed offerings the neocloud can use (Crunchy /
  CloudNativePG for Postgres; ClickHouse Operator or Altinity Cloud).
- Schema evolution discipline required: priced-events tables must be
  source-of-truth-equivalent across both, with documented dual-write
  semantics.
- Operator console queries must be aware which store to hit:
  recent-7d → Postgres for transactional correctness; everything else
  → ClickHouse for speed.
- Backup / restore is a two-store operation. Documented in DR
  runbooks.

## Notes

- For very small (Tier-3) deployments we may collapse to Postgres-only
  with TimescaleDB extension, gated behind a "lite" config flag. This
  is not the default.
- We commit to running on **stock Postgres 16+** and **stock
  ClickHouse 24+**. No fork dependencies.
- Raw (pre-pricing) event archive lives in object storage, not in
  either database. Both Postgres and ClickHouse are derivable from it.
