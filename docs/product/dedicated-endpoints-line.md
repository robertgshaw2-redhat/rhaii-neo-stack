# Product Line: Dedicated Endpoints (DE)

## What we are selling the neocloud

A self-service product their customers can use to spin up a
single-tenant, dedicated deployment of a curated model (or a customer's
own model / LoRA) on reserved GPU capacity, with autoscaling,
scale-to-zero, predictable latency, and a real SLO — without filing a
ticket.

## What the neocloud sells to its customers

A managed inference endpoint that:

- Runs the customer's chosen model on their chosen GPU SKU.
- Exposes the same OpenAI-compatible API as MaaS, at a private URL or
  under a project-scoped key.
- Autoscales replicas based on QPS, queue depth, and TPOT targets.
- Scales to zero when idle (optional; some customers want warm).
- Supports LoRA hot-swap so one endpoint serves many fine-tunes.
- Is billed per GPU-second of active capacity (plus optional per-token
  uplift the neocloud can configure for revenue parity with MaaS).
- Carries a real SLO the neocloud commits to in writing.

## Why this product exists

The DE product is where enterprises land — the ones who:

- Cannot use shared MaaS for data-handling reasons.
- Have a fine-tune and need it served.
- Need a predictable, isolated latency profile (e.g., voice agents,
  trading copilots, code-completion at scale).
- Want a contractual relationship with the inference vendor, not a
  click-through ToS.

This is the higher-ARR, lower-volume side of the platform. Each customer
is bigger, stickier, slower-to-close, and longer-tenured.

## Functional scope

### Endpoint provisioning (MVP)

A developer picks:

- **Model** — one of the curated catalog, *or* (GA) a previously
  uploaded BYO model, *or* (GA) a LoRA on top of a base model.
- **Accelerator SKU** — H100 80G, H200, MI300X, B200, Gaudi3,
  depending on what the neocloud offers.
- **Capacity shape** — min replicas, max replicas, scale-to-zero
  on/off, idle timeout.
- **Tenancy** — shared GPU acceptable / dedicated GPU required.
- **Region** — if the neocloud is multi-region.
- **Quotas** — RPM, TPM at the endpoint scope.

We provision in 60–180 seconds for an existing model, 5–15 minutes for a
BYO model that needs warming.

### LoRA & fine-tune support

- A single base-model Dedicated Endpoint can host **multiple LoRA
  adapters**. Each adapter has its own ID; the gateway routes requests
  with `model: "base@adapter-id"` to the right adapter on the same
  replicas. This is a key margin driver: many customers' LoRAs share
  the same expensive base model.
- LoRA adapters upload via the developer console or API. Capped size,
  signature-verified.

### Bring-your-own model (GA)

- Customer pushes a vLLM-compatible model artifact (HF Hub repo URL or
  signed tarball into customer-namespaced S3).
- We verify license, scan for known malicious patterns, run a
  health-check inference, and publish a private SKU.
- BYO models cost more per GPU-second (operator-configurable) because
  they bypass the operator's tuning library.

### Fine-tuning (v2)

- LoRA SFT only, at first. Full fine-tunes are out of scope until v3.
- Training jobs run on a separate scheduling pool from inference.
- The output is a LoRA artifact that auto-publishes as an addressable
  adapter on the customer's Dedicated Endpoint.

### Autoscaling and scale-to-zero

- Default policy: scale on queue depth + target TPOT, with a 60s warm
  buffer.
- Scale-to-zero default cold-start budget: 30–60s for in-cache
  artifacts, 2–8 min cold. Per-model promise published in the catalog.
- "Always warm" tier available for an uplift; replicas pinned ≥ 1.

### Pricing shape

Per GPU-second of active replica capacity:

```
  endpoint cost per hour = Σ (replica-GPU-count × price_per_gpu_hour)
                           over replica-on time
                           + per-token uplift × tokens served  (optional)
                           + idle-warm fee × hours warm-but-not-busy (optional)
```

The neocloud sets price-per-gpu-hour per SKU (typically a 1.5–4× markup
on their raw GPU-hour cost). Per-token uplift is rarely used but is
available for cases where the neocloud wants revenue parity with their
MaaS pricing.

### SLOs

We provide an SLO template for the neocloud to commit to:

| Metric | Target |
|---|---|
| Endpoint availability (monthly) | 99.9% |
| TTFT p95 (warm) | model-specific, published |
| TPOT p95 (warm) | model-specific, published |
| Cold-start p95 | model-specific, published |
| Time to recovery from a replica failure | < 60 s |

Credit policy for missed SLOs is the neocloud's call; we ship a default
percentage-of-monthly-spend grid they can adopt.

## Operator obligations

The neocloud commits to:

- Maintaining a documented matrix of supported model × accelerator SKUs.
- Honoring its published SLOs and crediting accordingly.
- Capacity planning for both the MaaS pools and the DE reservations
  (Neo Stack's operator console gives them the numbers; they own the
  business decision).

## Margin model

Illustrative:

```
A 1× H200 Dedicated Endpoint, billed at $5/GPU-hour to the customer:

  customer pays:                 $5.00/hr  per replica, while running
  GPU + power + cooling cost:    $2.00–$2.80/hr
  platform overhead (Neo Stack): ~5% effective
  gross margin:                  40–55%

Scale-to-zero shifts the unit economics meaningfully: if a customer
endpoint runs 30% of wall clock with cold-starts amortized at 60s, the
neocloud is selling 30% of an H200's hours at premium markup while the
GPU spends the other 70% serving MaaS or another endpoint.
```

The DE-on-MaaS-pool overflow design (in
[`reference-architecture.md`](../reference-architecture.md)) is the lever
that makes the dual-line platform unit-economically attractive: idle DE
capacity earns MaaS revenue, idle MaaS capacity is sold as DE reservation.

## Out of scope for DE line

- **Custom hardware drivers / kernels.** We support what the upstream
  vLLM and the neocloud's GPU operator support. No bespoke kernel work
  on a per-customer basis.
- **Foundation-model training.**
- **Distributed training across customer tenants.**
- **Per-endpoint custom security audits.** Customer can audit the
  shared platform; per-endpoint hardening beyond what tenancy provides
  is a services engagement, not a product feature.

## Open questions for design partners

1. Is "shared GPU acceptable" a meaningful price-point at MVP, or do
   most DE buyers refuse cohabitation regardless of cost?
2. What is the right default cold-start budget? We are picking 30–60s
   for warm-cache as the contract; design partners may want tighter.
3. How much LoRA storage per project should be included before
   overage fees? (Affects S3 spend on the neocloud.)
4. Do we expose a "VPC peering / PrivateLink to the endpoint" feature
   at MVP, or wait? Most enterprise DE buyers ask for this within
   month two.
