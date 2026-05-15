# Component: Observability

**Source**: Neo Stack (the integration and dashboards); the data
ultimately sinks to the neocloud's chosen observability backend.

We do not run an observability backend. We emit standard signals and
ship reference dashboards / alerts.

## Signals we emit

### Metrics

All Neo Stack services emit Prometheus / OpenMetrics with stable
labels:

```
neostack_<service>_<metric>{org, project, sku, endpoint, route, status, version}
```

Examples:

- `neostack_gateway_requests_total{org,project,sku,route,status}`
- `neostack_gateway_request_duration_seconds_bucket{...}`
- `neostack_meter_events_ingested_total{kind}`
- `neostack_meter_ingest_lag_seconds`
- `neostack_billing_export_lag_seconds{connector}`
- `neostack_spendcap_active_caps{level=warn|throttle|block}`
- `neostack_endpoint_replicas{endpoint,phase}`
- `neostack_endpoint_ttft_seconds_bucket{endpoint}`
- `neostack_endpoint_tpot_seconds_bucket{endpoint}`
- `neostack_endpoint_cold_starts_total{endpoint}`

Cardinality is bounded: `org` and `project` are present on per-tenant
metrics; aggregated rollups drop them. We do **not** label by API key
or by request id.

### Logs

Structured JSON logs from every service, with trace correlation. PII
redaction:

- API keys redacted to last-4 in logs.
- Prompts and completions never logged unless tenant has opted into
  logging (or is in the operator's "support fetch" flow).
- Email addresses partially redacted (`foo***@example.com`).

### Traces

OpenTelemetry across:

- Gateway → maas-api → Authorino → Limitador → IGW → vLLM.
- Control-plane services.
- Background workers (metering ingest, billing export, spend-cap
  evaluator, endpoint reconciliation).

Sampling default: 1% of successful requests + 100% of errors + 100% of
slow-tail (>p99). Tenants can request boosted sampling on a Dedicated
Endpoint.

### Audit

Separate stream, not mixed with general logs. Schema in
[`control-plane.md`](./control-plane.md). Sink:

- Local Postgres for queries from the operator console.
- Optional fan-out to a neocloud-side SIEM (Splunk, Sumo, Elastic,
  Datadog Audit).

## Backends we integrate with

| Backend | MVP | GA | Notes |
|---|---|---|---|
| Prometheus (in-cluster) | ✅ | ✅ | Always. |
| Grafana (in-cluster) | ✅ | ✅ | Reference dashboards shipped. |
| OpenTelemetry Collector | ✅ | ✅ | Default trace + metric egress. |
| Datadog | ✅ | ✅ | DogStatsD + Datadog APM. |
| Dynatrace | — | ✅ | OneAgent + OTel ingest. |
| New Relic | — | ✅ | OTel ingest. |
| Splunk Observability | — | partner | OTel ingest. |
| Elastic Observability | — | partner | OTel ingest. |

For each, we ship a **values file** (Helm) for one-step configuration
and a **dashboard pack** (JSON) ready to import.

## Dashboards we ship

### Platform health (operator)

- Cluster GPU utilization
- Pool queue depth + admission
- Gateway p50/p95/p99
- Error rate by class
- Metering lag
- Billing export status
- Endpoint replica health

### Tenant SLO (operator)

- One row per tenant: TTFT p95, TPOT p95, error rate, requests, spend.
- Sorted by SLO breach severity.

### Per-SKU performance (operator)

- Throughput, latency, utilization, queue depth per model SKU.
- Cold-start rate.
- KV cache pressure.

### Per-endpoint (operator + tenant)

- Replicas, scaling events, requests, latency, errors.
- LoRA usage.
- Trace samples.

### Billing health (operator)

- Priced events / minute.
- Reconciliation status per connector.
- Spend-cap events.
- Invoice generation status.

## Alert recipes (pre-built)

Shipped as Prometheus alerting rules + Datadog monitors. Examples:

- `Gateway error rate > 1% for 5m` → page.
- `Metering ingest lag > 90s for 3m` → page.
- `Billing export connector reconcile failed` → page.
- `SKU TTFT p95 > 2× target for 10m` → page (per SKU, per tenant
  optional).
- `Endpoint scale-up blocked due to capacity` → ticket.
- `Spend cap exceeded by > 5%` → notify finance.
- `Cold-start rate > 20% on an "always warm" endpoint` → page.

## What we don't do

- We don't run the dashboards in production for the neocloud. They run
  their own observability stack; we plug into it.
- We don't ingest into Red Hat or Neo Stack. No phone-home telemetry
  by default. Operators may opt-in to anonymous usage telemetry to
  inform our roadmap.
- We don't surface per-request prompts/completions in operator
  dashboards. That's a privacy boundary we enforce.
