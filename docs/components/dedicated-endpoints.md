# Component: Dedicated Endpoints

**Source**: Neo Stack.

The Dedicated Endpoints (DE) controller is the engine behind the DE
product line. It sits beside the upstream maas-controller (which
handles MaaS pools) and manages the lifecycle of single-tenant, named,
reserved-capacity endpoints.

## What it manages

For each Dedicated Endpoint, the controller materializes:

- A KServe `InferenceService` (or set of them, for prefill+decode
  disaggregation) configured per the endpoint's model SKU and SKU
  defaults from the catalog.
- llm-d / IGW pool membership scoped to this endpoint.
- Kuadrant `HTTPRoute` + AuthPolicy + RateLimitPolicy scoped to the
  endpoint's URL and the endpoint's allowed keys.
- Authorino `AuthConfig` for endpoint-scoped tokens.
- Per-endpoint metric scrape configs and recording rules.
- Optional LoRA registry entries for adapters loaded on this endpoint.
- Per-endpoint scale policy.

## CRD shape (sketch)

```yaml
apiVersion: neostack.redhat.com/v1alpha1
kind: Endpoint
metadata:
  name: my-customer-prod
  namespace: tenant-acme
spec:
  baseModel: llama-3.1-70b-instruct@h200-fp8
  accelerator:
    sku: nvidia-h200-80gb
    perReplica: 1
  scaling:
    min: 1
    max: 4
    target:
      tpotMs: 30
      queueDepth: 16
    scaleToZero:
      enabled: false
      idleSeconds: 300
  tenancy:
    sharedGpuAllowed: false
  adapters:
    - id: marketing-copilot-v3
      uri: s3://acme/loras/marketing-copilot-v3.safetensors
  routing:
    sticky: false
  slo:
    availabilityTarget: 99.9
    ttftP95Ms: 200
    tpotP95Ms: 40
  pricing:
    profile: standard         # references operator price book
    perTokenUpliftPer1M: 0    # optional
status:
  url: https://api.acme.cloud/v1/endpoints/my-customer-prod
  phase: Ready
  replicas: 2
  conditions: [...]
```

## Why a separate controller and not "just maas-controller"

The upstream maas-controller is built around the MaaS model: shared
multi-tenant pools, model registration, key-bound access. Dedicated
Endpoints have a different shape:

- A 1:1 relationship between the endpoint and the customer.
- Per-endpoint scale, SLO, accelerator pinning, LoRA list.
- Per-endpoint billing dimension (GPU-second), not just tokens.
- Per-endpoint URL and Kuadrant routing.
- A different lifecycle (active hours, autoscaling, scale-to-zero,
  archive on customer churn).

Forcing this into maas-controller would either bloat the upstream or
mangle our shape. We keep the controllers separate and the CRDs
distinct. We **may** upstream commonalities once the shape stabilizes.

## Scaling policy

The scaling policy is the most product-visible knob in DE. Default:

- Target TPOT (e.g., 30ms p95) and target queue depth (e.g., 16).
- Scale up when either is exceeded for 30s.
- Scale down when both are below 50% of target for 5 minutes.
- Scale-to-zero only if `spec.scaling.scaleToZero.enabled`.

Customers can override via the developer console or API. Operators set
ceilings (max replicas, max scale-up rate) to protect the cluster.

## Cold starts

Cold-start strategy per phase:

- **MVP**: in-image weights for catalog models (fast cold start, ~30–60s
  warm-from-cache); per-image cold pull on first replica of an SKU.
- **GA**: prefetching weights to local SSD on nodes likely to schedule
  the SKU; LoRA hot-load.
- **v2**: in-memory weight handoff between replicas during scale-to-zero
  to compress cold-start budget.

Each model SKU publishes a **cold-start contract** in its model card.
Customers can pay for "always warm" to bypass cold starts entirely.

## LoRA hot-swap

- LoRA adapters are first-class. An endpoint can host many adapters
  on the same base-model replicas.
- Routing by `model: "base@adapter-id"` is handled by the gateway
  with hints into the IGW.
- Per-adapter accounting: priced events carry the adapter ID so
  customers can analyze per-product cost.

## BYO model (GA)

The intake pipeline (separate doc to come; outline here):

1. Customer pushes artifact (HF Hub repo or signed tarball).
2. Verification: license parse, signature check (cosign), vulnerability
   scan (clamav-style + ML-specific patterns), schema check
   (vLLM-compatible).
3. Health probe: launch a single replica, run a tiny inference, verify
   response shape and a smoke benchmark.
4. Publish as a **private SKU** in the catalog, visible only to the
   tenant.
5. The tenant can now select it on a new or existing endpoint.

## Observability

Per-endpoint dashboards (developer-facing):

- Requests, error rate, latency p50/p95/p99.
- TTFT and TPOT histograms.
- Token throughput.
- Replicas (current, min, max, target).
- LoRA adapter usage breakdown.
- GPU utilization, KV cache pressure.

Per-endpoint OpenTelemetry trace export at GA.

## Failure modes

| Failure | Behavior |
|---|---|
| Replica OOM | KServe restarts. IGW routes around. Customer sees brief failures, retried if their SDK retries. |
| GPU node fail | Pods reschedule onto another node from the same pool. Endpoint shows degraded status until replicas return. |
| Out of GPU capacity | Scale-up blocked. Endpoint stays at current replicas. We surface this in the operator console (capacity pressure) and to the customer (deny scale-up event in audit). |
| Customer hits spend cap on endpoint | Limitador throttles → eventually blocks. Endpoint health is "blocked-by-spend-cap." |
| Customer LoRA broken on upload | Health probe fails, adapter not published, error returned to customer. |

## SLO accounting

SLOs are evaluated **per endpoint per month** against published TTFT
p95, TPOT p95, and availability targets. Credits are computed by the
billing service from breach severity and emitted as adjustments. The
neocloud sets the credit grid; Neo Stack provides a default.

## What this controller is **not**

- Not a fine-tuner. Fine-tuning is a separate component (v2).
- Not a training platform.
- Not a general workflow engine. Endpoints are long-lived running
  services; one-shot jobs go elsewhere.
