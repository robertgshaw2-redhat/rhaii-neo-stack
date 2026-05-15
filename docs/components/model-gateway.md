# Component: Model Gateway

**Source**: Upstream `opendatahub-io/models-as-a-service` —
Kuadrant + Gateway API + Authorino + Limitador + maas-api +
maas-controller.

**Neo Stack contribution**: pinned distribution, default configurations,
spend-cap controller, metering tap, audit fan-out, supported upgrade path.

## Responsibilities

- Terminate API traffic at the cluster's Gateway API listener.
- Authenticate via API key (or OpenShift / OIDC token where the customer
  uses one).
- Resolve API key → project → org via maas-api.
- Enforce rate limits and quotas (RPM, TPM, concurrency, daily spend)
  through Limitador.
- Strip customer credentials before forwarding upstream (Authorino).
- Inject tenant tags into outbound headers so the llm-d Inference
  Gateway and downstream metering know whose request this is.
- Emit a usage event for every completed request.

## What we contribute, in detail

### Spend-cap controller

A small Neo Stack service that:

1. Reads aggregated near-real-time spend from the metering pipeline
   (Postgres hot store).
2. Compares against per-key, per-project, per-org caps.
3. Writes Limitador rate-limit rules that throttle or block when the
   cap is approached or hit (50% warn, 80% throttle, 100% block).
4. Tells the billing service to emit "approaching cap" alerts to the
   tenant.

The controller is independent of Limitador's normal RPM/TPM rules — it
*adds* spend-derived rules on top.

### Metering tap

Kuadrant's request-response cycle emits an event we capture. We
**don't** change Kuadrant's hot path; we attach an OpenTelemetry-style
exporter that publishes a structured usage record to NATS JetStream /
Redpanda. The pipeline downstream is Neo Stack's.

Fields per event:

```json
{
  "request_id": "...",
  "ts": "2026-05-15T12:34:56Z",
  "tenant_id": "org_…",
  "project_id": "proj_…",
  "api_key_id": "key_…",
  "model_sku": "llama-3.1-70b-instruct@h200-fp8",
  "endpoint_id": null,                  // set for DE
  "input_tokens": 1283,
  "output_tokens": 412,
  "cached_input_tokens": 0,
  "latency_ms": 1843,
  "ttft_ms": 67,
  "status": 200,
  "route": "v1.chat.completions",
  "gateway_version": "…",
  "data_plane_pool": "maas-h200",
  "request_size_bytes": 12345,
  "response_size_bytes": 9876
}
```

Events are idempotent on `request_id`; the metering pipeline dedupes.

### Audit fan-out

Auth decisions (issued key, revoked key, blocked-by-cap, rate-limit hit
above N%) get fanned out to the audit log so the operator can defend
the tenant experience in support tickets.

### Default config bundle

Neo Stack ships a Helm chart `model-gateway-defaults` that lays down:

- Kuadrant policies for the tiered rate-limit grid in
  [`product/maas-line.md`](../product/maas-line.md).
- AuthPolicy templates per role (developer, billing, etc.).
- RouteRules for the OpenAI-compatible paths.
- A reference set of HTTP retry / timeout policies.

The defaults are explicit and overridable. Operators are expected to
inspect them, not adopt blindly.

## Why we don't fork the upstream gateway

Upstream Kuadrant / Authorino / Limitador are mature CNCF projects.
Forking creates an indefinite maintenance burden. Where we need
behavior that does not exist upstream, we contribute it upstream and
ship via the next release. The spend-cap loop is the one place where we
keep a downstream-only component because the controller depends on
billing state that lives in Neo Stack.

## Failure modes

| Failure | Behavior |
|---|---|
| Authorino loses Postgres link to maas-api | Authorino's cache holds for 60s; new auth decisions return 503 after that. |
| Limitador unavailable | Configured fail-open or fail-closed per policy; default fail-closed. |
| Spend-cap controller falls behind | Cap enforcement lags; finance reconciles. Hard cap remains via Limitador absolute daily RPM as a backstop. |
| Metering tap loses connection | Events buffer to local disk, replay when connection restored. Bounded buffer. |
| Gateway pod crashes | K8s reschedules; in-flight streams drop and client retries. |

## Observability

- Prom metrics: requests by tenant × model × status × route, latency
  histograms, rate-limit denials, spend-cap hits.
- Per-tenant SLO dashboards built off these metrics.
- Audit log streamed to the centralized audit store.

## Out of scope for the gateway

- Inference routing beyond "send to the right llm-d IGW." Smart routing
  is the IGW's job.
- Model selection logic. The catalog service decides the SKU; the
  gateway honors it.
- Payment processing. Billing service.
