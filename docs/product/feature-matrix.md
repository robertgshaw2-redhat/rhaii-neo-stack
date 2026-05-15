# Feature Matrix: MaaS vs Dedicated Endpoints

A side-by-side reference. Use this when explaining the product to the
neocloud's go-to-market team. Most features are shared; this document
flags the differences.

## Buyer & motion

| | MaaS | Dedicated Endpoints |
|---|---|---|
| Primary buyer | Indie dev → startup → mid-market | Mid-market → enterprise |
| Buying motion | Self-service, credit card | Self-service start, sales-assisted expand |
| Time to first token | Seconds (sign-up to `curl`) | 60s–15min (provisioning) |
| Contract shape | Click-through ToS | MSA, DPA, sometimes BAA |
| Stickiness | Medium (API key migration) | High (endpoint URL, fine-tune lives there) |

## API surface

| Capability | MaaS | DE |
|---|---|---|
| OpenAI-compatible chat / completions / embeddings | ✅ | ✅ |
| Tool calling, structured outputs | ✅ | ✅ |
| Responses API (GA) | ✅ | ✅ |
| Reranking (GA) | ✅ | ✅ |
| Multimodal (GA) | ✅ | ✅ |
| Batch + Files API (GA) | ✅ | optional |
| Speech (v2) | ✅ | ✅ |
| Custom URL / vanity hostname | ❌ | ✅ |

## Models

| | MaaS | DE |
|---|---|---|
| Curated catalog | ✅ | ✅ |
| BYO model (HF Hub or signed artifact) | ❌ | ✅ (GA) |
| LoRA hot-swap per request | ❌ | ✅ |
| Fine-tuning service | ❌ | ✅ (v2) |
| Model version pinning | best-effort | exact |

## Tenancy & isolation

| | MaaS | DE |
|---|---|---|
| Per-tenant K8s namespace | shared by SKU | per endpoint |
| Per-tenant rate limits & quotas | ✅ | ✅ |
| Dedicated GPU possible | ❌ | ✅ |
| Cross-tenant KV cache reuse | ✅ (opt-in) | ❌ |
| Per-tenant log retention controls | ✅ | ✅ |
| VPC peering / PrivateLink | ❌ | ✅ (GA) |

## Routing & performance

| | MaaS | DE |
|---|---|---|
| Predicted-latency routing | ✅ | ✅ |
| KV-cache-aware routing | ✅ | ✅ |
| P/D disaggregation | ✅ | ✅ |
| Sticky routing (session affinity) | ❌ | ✅ |
| Speculative decoding | ✅ where applicable | ✅ |
| Per-endpoint warm replicas | n/a | ✅ |

## Scaling

| | MaaS | DE |
|---|---|---|
| Pool autoscaling | platform-managed | n/a |
| Replica autoscaling | n/a | customer-managed (min, max, target TPOT) |
| Scale to zero | n/a | ✅ |
| Cold-start budget | n/a | published per model |
| Burst capacity from overflow pool | ✅ | ✅ (overflow into MaaS pool, opt-in) |

## Billing & metering

| | MaaS | DE |
|---|---|---|
| Billing basis | Tokens (in/out/cached) | GPU-second + optional per-token |
| Cached-input discount | ✅ | n/a |
| Volume discounts | tier-based | per-contract |
| Idle-warm fee | n/a | ✅ (optional) |
| Egress charges | passthrough | passthrough |
| Spend caps | ✅ | ✅ |
| Prepaid credits | ✅ | ✅ |

## SLO

| | MaaS | DE |
|---|---|---|
| Availability target | shared, per-SKU | per-endpoint |
| Latency target | per-SKU template | endpoint-customizable |
| Service credits | platform-wide | per-endpoint, contractual |

## Observability available to the customer

| | MaaS | DE |
|---|---|---|
| Usage dashboards | ✅ | ✅ |
| Per-request logs (opt-in) | ✅ | ✅ |
| Latency histograms | aggregated | per endpoint |
| Replica-level metrics | ❌ | ✅ |
| Live trace export | ❌ | ✅ (OTel) |

## Where the two lines share infrastructure

- **Same Model Gateway** for auth, rate-limit, metering, audit.
- **Same Inference Gateway (llm-d)** for routing, with different pool
  selection.
- **Same metering pipeline** and billing service.
- **Same developer console**, with the DE-specific pages gated by
  feature flag per project.
- **Same operator console.**
- **Same catalog** (DE may publish private SKUs).

## Where they diverge

- **Pool topology.** MaaS pools are shared and right-sized to demand
  curves; DE pools are reserved capacity slices.
- **Scheduler policy.** MaaS pool prefers throughput; DE pool prefers
  latency consistency and respects per-endpoint SLO.
- **Billing dimension** as discussed.
- **Tenant boundary.** Kernel / namespace boundary for DE; gateway and
  cache-partition boundary for MaaS.
