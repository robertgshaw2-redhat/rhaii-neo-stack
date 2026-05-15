# Sizing & Capacity Planning

The hard problem at a neocloud is **GPU sizing**. CPU/memory for Neo
Stack itself is rounding error against the GPU bill. This document
sizes both.

## Control plane (non-GPU) sizing

For a cluster supporting up to 10,000 concurrent in-flight requests
and 10M priced events per day:

| Service | Replicas | CPU | Memory | Notes |
|---|---|---|---|---|
| Gateway (Kuadrant + Authorino + Limitador) | 3–6 | 2 vCPU each | 4 Gi each | Stateless; scale horizontally. |
| maas-api | 3 | 1 vCPU | 2 Gi | Stateless; Postgres-backed. |
| maas-controller | 1 | 1 vCPU | 1 Gi | Leader-elected. |
| Endpoint controller | 1 | 1 vCPU | 1 Gi | Same. |
| Catalog service | 2 | 1 vCPU | 1 Gi | Cache-heavy. |
| Billing service | 2 | 2 vCPU | 4 Gi | Cron-driven for invoicing. |
| Spend-cap controller | 1 | 1 vCPU | 1 Gi | Reads ClickHouse + writes Limitador. |
| Metering ingest | 3 | 2 vCPU | 2 Gi | Scale with event volume. |
| Developer console | 3 | 1 vCPU | 1 Gi | Next.js. |
| Operator console | 2 | 1 vCPU | 1 Gi | Same. |
| Postgres (primary + replica) | 2 | 4 vCPU | 16 Gi | + storage. |
| ClickHouse | 3 | 4 vCPU | 16 Gi | + storage. |
| NATS JetStream | 3 | 2 vCPU | 4 Gi | Or Redpanda. |

Total ballpark: **~80 vCPU, ~200 GiB RAM, ~5 TiB persistent storage**
for the control plane of a mid-sized deployment.

For a tier-1 deployment (multiple thousand concurrent, 100M+ events/day),
scale Postgres, ClickHouse, NATS, ingest workers proportionally.

## GPU sizing

There is no formula; there is a process. Neo Stack supports the
process with the operator console and a sizing CLI.

### Inputs

- Expected MaaS QPS distribution per model class (chat-small,
  chat-medium, chat-large, frontier-open, reasoning, embeddings,
  reranker).
- Expected prompt length distribution.
- Expected generation length distribution.
- Target TTFT p95 and TPOT p95 per model class.
- Target pool utilization (typically 60–75%; higher = better margin
  but worse tail latency).
- Reservation for DE endpoints.

### Outputs (per accelerator SKU)

- Replicas required.
- Headroom for burst.
- Implied $/1M tokens cost given the neocloud's per-GPU-hour cost.
- A breakeven curve: at what QPS does this SKU produce positive margin?

### Heuristics (illustrative)

For Llama-3.1-70B-Instruct FP8 on H200, with 1k-token prompts and
500-token generations, target TTFT p95 < 300ms, TPOT p95 < 30ms, target
utilization 65%:

- ~3–4 H200 replicas serve ~80–120 RPS sustained, depending on prompt
  length distribution and KV cache reuse.
- Per replica throughput: ~25–30 RPS.
- Cost-per-1M output tokens at $4/H200-hour: ~$0.25–0.30.

These numbers are not promises; they are starting points. The catalog
ships per-SKU benchmark sheets with the real curves.

## DE reservation strategy

A useful pattern:

- Provision a **shared MaaS pool** sized to the steady-state MaaS
  demand.
- Provision a **DE reservation pool** sized to the sum of committed DE
  minimum replicas.
- Maintain an **overflow buffer** that can be either MaaS or DE
  depending on demand.

The overflow pool gives both lines elasticity without doubling
inventory. The operator console exposes utilization for all three
pools so the team can right-size weekly.

## Storage sizing

| What | Per-tenant | Per-cluster | Notes |
|---|---|---|---|
| Model artifacts | 0 | ~5 TiB for MVP catalog | Pre-staged. Grows with catalog. |
| LoRA adapters | ~1–50 GiB | depends | Limit per tenant. |
| Priced events (Postgres hot) | ~10 GiB per 1M req | ~1 TiB | 7-day window. |
| Priced events (ClickHouse cold) | ~3 GiB per 1M req | ~10–100 TiB | Compressed; multi-year. |
| Audit | ~1 GiB per 1M events | ~10 TiB | 13-month default retention. |
| Backups | + 30% | + 30% | Daily incremental, weekly full. |

## Network sizing

- Gateway and console traffic is small (control plane).
- Inference traffic dominates: per-token is sub-kB, but streaming long
  responses at scale is meaningful.
- For 1k RPS on long responses averaging 5KB out, that's ~5 MB/s
  egress per gateway pod. Horizontal scale takes it from there.
- KV cache traffic between prefill and decode workers (P/D
  disaggregation) is **large** and stays inside the cluster. Plan
  intra-rack bandwidth.

## The sizing CLI (MVP)

```
$ neostackctl sizing recommend \
    --maas-qps llama-3.1-70b=80 \
    --maas-qps llama-3.1-8b=300 \
    --accelerator h200 \
    --target-utilization 0.7 \
    --reserve-de 4

  llama-3.1-70b-instruct@h200-fp8:  6 replicas  (5 base + 1 burst headroom)
  llama-3.1-8b-instruct@h200-fp8:   4 replicas
  DE reservation:                    4 replicas
  Overflow pool:                     2 replicas

  Estimated $/1M output (70B):       $0.27
  Estimated $/1M output (8B):        $0.08
  Estimated margin at list price:    62% (70B), 55% (8B)
```

This is a starting point, not a contract. The operator validates
against real traffic over the first two weeks of running.
