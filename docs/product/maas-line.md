# Product Line: Model-as-a-Service (MaaS)

## What we are selling the neocloud

A shared, multi-tenant, OpenAI-compatible token API that runs on the
neocloud's GPU fleet, branded as the neocloud's product, billed by token,
and operated by the neocloud's SRE team with Neo Stack tooling and Red
Hat support.

## What the neocloud sells to its customers

A drop-in alternative to `api.openai.com` for open-weight models:

- Top open chat models (Llama, Qwen, DeepSeek, Mistral, Gemma).
- Embeddings (e.g., BGE, E5, Nomic, OpenAI-compatible).
- Rerankers.
- Vision/multimodal where available.
- Tool calling and structured outputs across the catalog.
- Batch endpoint for cheaper async workloads (GA).

Priced per million input / output / cached tokens. Same payload shape as
OpenAI's API; the customer's SDK works without code changes.

## Why this product exists

End customers want tokens, not GPUs. A neocloud that only sells GPU-hours
is invisible to the developer who is choosing between OpenAI and an open
model. MaaS makes the neocloud visible in that decision, with five
advantages over hyperscaler MaaS (Bedrock, Vertex, Foundry):

1. **Price.** Neoclouds can undercut by 20–60% on most popular models.
2. **Performance.** llm-d + tuned vLLM beats most managed offerings on
   throughput and TTFT at comparable model sizes.
3. **Sovereignty.** Region-pinned, with a clear residency story.
4. **Neutrality.** Not tied to a hyperscaler's data plane or billing.
5. **Direct relationship.** Their customer, their account, their support.

## Functional scope

### Endpoints (MVP)

- `POST /v1/chat/completions` (streaming + non-streaming)
- `POST /v1/completions` (legacy)
- `POST /v1/embeddings`
- `GET  /v1/models`
- Tool / function calling
- Structured outputs (JSON schema)

### Endpoints (GA / v2)

- `POST /v1/responses` (parity with OpenAI Responses)
- `POST /v1/moderations`
- `POST /v1/batches` + `POST /v1/files`
- Reranking endpoint
- Vision / multimodal inputs

### Catalog at MVP

The MVP catalog is intentionally short, so we can stand behind every entry
with benchmarks, tuning, and SLOs. The exact list is decided with design
partners, but the shape is:

- 3 chat models at different size points (e.g., 8B, 70B, ~400B-equivalent
  MoE).
- 2 reasoning-tuned models (e.g., DeepSeek-R1-distill, Qwen-Reasoning).
- 2 embedding models (general + multilingual).
- 1 reranker.
- 1 small/cheap chat model for high-volume cheap use cases.

GA catalog grows to 25+ models with quarterly cadence. Each model SKU is
{family, size, quantization, accelerator profile}.

### Rate-limit tiers (MaaS defaults)

| Tier | RPM | TPM | Concurrent | Spend cap |
|---|---|---|---|---|
| Free / Trial | 20 | 40,000 | 4 | $5 lifetime |
| Developer | 500 | 1,000,000 | 32 | configurable |
| Pro | 5,000 | 10,000,000 | 128 | configurable |
| Enterprise | custom | custom | custom | none |

Tiers are templates the neocloud can edit per their business model.

### Pricing model for the neocloud's customers

Per million tokens, split by input / output / cached. Neocloud sets list
prices in the operator console; per-tenant overrides supported. Neo Stack
ships with a **reference price book** seeded from publicly observable
market prices, refreshed quarterly. This is guidance, not enforcement.

Cached input tokens (via prefix-cache hit) are billed at a configurable
discount (default 50% off input list).

### Abuse and safety

- Rate-limit by IP and by key.
- Bot-signal scoring at the edge (delegated to neocloud's WAF).
- Optional inline moderation: outbound tokens scanned against a
  configurable classifier; flagged responses can be blocked or just logged.
- Per-tenant kill-switch from the operator console.
- Anomaly detection on usage shape (e.g., a free-tier key suddenly
  hitting RPM ceiling for 10 minutes straight).

## Operator obligations

The neocloud commits to:

- Publishing a public price book and rate-limit grid.
- A documented SLO per model SKU (TTFT p95, TPOT p95, availability).
- A status page with per-SKU health.
- Support response times consistent with each tier.

Neo Stack ships the tooling to meet these obligations; the neocloud
chooses the numbers it commits to.

## Margin model

Illustrative (a real plan is built per-neocloud with their actual GPU
costs and capacity utilization assumptions):

```
A reasonable target for a 70B-class chat model on H200, well-tuned, at
60–75% pool utilization, with mixed prompt lengths:

  cost per 1M output tokens:       $0.20 – $0.55
  market list per 1M output:       $0.60 – $1.20
  gross margin per 1M output:      55%  – 75%

The neocloud's job is to keep utilization up. Neo Stack's job is to make
that easy via routing, caching, and overflow into Dedicated capacity
during idle periods.
```

## Out of scope for the MaaS line

- Training or fine-tuning of foundation models. (Fine-tuning *for a
  Dedicated Endpoint* is in scope at GA; foundation training is not.)
- Hosting closed-weight third-party models. (We do not have the
  licensing relationships; the neocloud may.)
- Acting as a model marketplace where third parties sell their own models
  on the platform. Possible later.

## Open questions for design partners

1. How aggressive should the default catalog be on reasoning models vs.
   classic chat? (Affects GPU mix.)
2. Is the "cached input discount" a feature operators want exposed to
   end customers, or do they want to capture it as margin?
3. What is the appetite for shared batch (queue-based, cheaper)
   inference at MVP vs. holding it to GA?
4. Where do we land on opt-in prompt logging for quality improvement —
   the neocloud's call, our default, or strict opt-in per project?
