# Reference Architecture

This document describes the runtime shape of Neo Stack — what runs where, and
how a request flows from a developer's `curl` to a token coming back.

**Prerequisite reading**: [`stack.md`](./stack.md). Neo Stack consumes the
upstream `opendatahub-io/models-as-a-service` project (Kuadrant + Authorino +
Limitador + maas-api + maas-controller) as its gateway. Where this document
says "Model Gateway," it means that upstream stack, configured and operated
by Neo Stack — not a from-scratch service.

## Logical view

```
                                    ┌──────────────────────────────────────────┐
                                    │              End customer                │
                                    │  (Alex / Maya / Jordan in personas.md)   │
                                    └────────────────┬─────────────────────────┘
                                                     │  HTTPS, OpenAI-compatible
                                                     │  Authorization: Bearer sk-…
                                                     ▼
┌───────────────────────────────────────────────────────────────────────────────┐
│                                  EDGE                                          │
│  TLS termination, anycast / DDoS, WAF (neocloud-owned)                         │
└────────────────────────────────────┬──────────────────────────────────────────┘
                                     ▼
┌───────────────────────────────────────────────────────────────────────────────┐
│        MODEL GATEWAY  (upstream models-as-a-service, configured by Neo Stack)  │
│                                                                                │
│   Kuadrant + Gateway API   ─  routing, policy                                  │
│   Authorino                ─  API key / token auth, header stripping           │
│   Limitador                ─  per-key / project / org rate & quota             │
│   maas-api                 ─  key issuance, project/org resolution             │
│   maas-controller          ─  tenant + model reconciliation                    │
│                                                                                │
│   Neo Stack contributes: spend-cap plug-in (writes Limitador rules from        │
│   billing state), tenant-tag emission, request-id correlation, audit fan-out.  │
└───────────────┬───────────────────────────────────────────────────┬───────────┘
                │                                                   │
                ▼                                                   ▼
┌──────────────────────────────────┐               ┌────────────────────────────┐
│   llm-d Inference Gateway        │               │  Metering pipeline         │
│   (upstream, unmodified)         │               │  (NEO STACK)               │
│   • Predicted-latency scheduling │               │  • Consumes gateway events  │
│   • KV-cache-aware routing       │               │  • Idempotent ingest        │
│   • P/D disaggregation           │               │  • Hot store: Postgres      │
│   • Batch gateway (GA later)     │               │  • Cold store: ClickHouse   │
└──────┬─────────────────┬─────────┘               │  • Export: Stripe/Metronome │
       │                 │                          └────────────────────────────┘
       ▼                 ▼
┌───────────────────────────────────────────────────────────────────────────────┐
│                        INFERENCE DATA PLANE                                    │
│                                                                                │
│  KServe InferenceService (one per model SKU / endpoint)                        │
│   ├── Prefill workers      (vLLM, llm-d operator-managed)                      │
│   ├── Decode workers       (vLLM, llm-d operator-managed)                      │
│   └── KV cache pool        (shared per pool, partitioned by tenant when DE)    │
│                                                                                │
│  Two pool types:                                                               │
│   • MaaS pools         — shared, multi-tenant, autoscaled to fleet demand     │
│   • Dedicated pools    — single-tenant, per-endpoint, scale-to-zero capable   │
└───────────────────────────────────────────────────────────────────────────────┘

                                     ▲
                                     │  All planes export to:
                                     ▼
┌───────────────────────────────────────────────────────────────────────────────┐
│   Control plane (Neo Stack) + Observability (neocloud-owned)                  │
│                                                                                │
│   ┌───────────────────────┐   ┌──────────────────────┐   ┌────────────────┐  │
│   │ Tenancy / IAM service │   │ Catalog service      │   │ Billing svc    │  │
│   │ (orgs, projects, keys)│   │ (models, SKUs, prices)│   │ (export, caps) │  │
│   └───────────────────────┘   └──────────────────────┘   └────────────────┘  │
│                                                                                │
│   Developer console (Next.js)        Operator console (Next.js)                │
│                                                                                │
│   Prom / OTel → Datadog / Grafana / Dynatrace                                  │
└───────────────────────────────────────────────────────────────────────────────┘
```

## Components

| Component | Source | Notes |
|---|---|---|
| Edge / TLS / WAF | Neocloud | We do not ship this. |
| Model Gateway (Kuadrant + Authorino + Limitador) | Upstream `models-as-a-service` | Neo Stack ships a pinned, supported distribution and config defaults. |
| maas-api / maas-controller | Upstream `models-as-a-service` | Same. |
| llm-d Inference Gateway | Upstream llm-d | We ship a curated bundle and tuning. |
| KServe | Upstream | Operator-managed. |
| vLLM workers | Upstream / Red Hat AI Inference Server | Tuned profiles per model SKU. |
| **Endpoint-lifecycle controller** | **Neo Stack** | Sits beside maas-controller for Dedicated Endpoints. |
| **Catalog service** | **Neo Stack** | Postgres-backed; ships with seed catalog. |
| **Billing service** | **Neo Stack** | Postgres + ClickHouse. |
| **Metering pipeline** | **Neo Stack** | Consumes events from the gateway; Redpanda/NATS → ClickHouse → exporters. |
| **Spend-cap controller** | **Neo Stack** | Closes the loop between billing state and Limitador. |
| **Developer console** | **Neo Stack** | Next.js. Whitelabel themable. |
| **Operator console** | **Neo Stack** | Next.js. Internal. |
| **Reference integrations** | **Neo Stack** | Helm values + secrets contracts + runbooks. |
| Observability | Neocloud | We emit standard Prom / OTel and provide dashboards. |

## Tenancy model

- **One Kubernetes namespace per tenant** for any tenant-scoped workload
  (Dedicated Endpoints, fine-tune jobs). Enforced by NetworkPolicy,
  PodSecurityStandards, and a per-tenant ServiceAccount.
- **MaaS pools share namespaces by model SKU**, not by tenant. Isolation is
  in the gateway, in KV cache partitioning, and in rate limiting — not in
  the kernel. This is an explicit tradeoff: MaaS economics require cache
  sharing.
- **Per-GPU tenant pinning** is available for DE on customer request,
  configurable at endpoint creation. It costs more; the operator console
  enforces minimum SKUs.

Full discussion: [`adr/0004-multi-tenant-model.md`](./adr/0004-multi-tenant-model.md).

## Request flow (MaaS)

1. Customer sends `POST /v1/chat/completions` to
   `https://api.<neocloud>.com` with `Authorization: Bearer sk-…`.
2. Edge terminates TLS and forwards to the Gateway API listener that
   Kuadrant manages.
3. Kuadrant + Authorino + Limitador (configured by Neo Stack defaults):
   - Authorino calls maas-api (cached) to resolve API key → project → org.
     Rejects if revoked/expired/over-spend-cap.
   - Limitador checks per-key, per-project, per-org RPM/TPM and the
     spend-cap-derived limits Neo Stack's spend-cap controller has
     written.
   - Authorino strips the customer Authorization header before forwarding
     so we never leak credentials to vLLM or the IGW.
   - The request is tagged with tenant + project + model SKU (Kuadrant
     header injection) and forwarded to the llm-d Inference Gateway.
4. IGW selects a worker using predicted-latency + KV-cache awareness and
   forwards. P/D disaggregation kicks in for long prompts.
5. vLLM streams tokens back through IGW and the Gateway to the client.
6. The gateway emits a usage event (request id, tenant, model, input
   tokens, output tokens, cached tokens, latency, status) to the Neo
   Stack metering pipeline (NATS JetStream or Redpanda topic).
7. Metering pipeline writes hot to Postgres, batches to ClickHouse, and
   exports to the configured billing connector. The spend-cap controller
   reads aggregated spend on a cadence and updates Limitador rules.

## Request flow (Dedicated Endpoint)

Same as MaaS except:

- The API key is endpoint-scoped, or the project is scoped to the endpoint.
- The Gateway routes to the endpoint's dedicated pool (sticky), bypassing
  the MaaS pool entirely.
- GPU-second accounting runs in parallel with token accounting; whichever
  is the billable dimension for that endpoint wins.

## Failure modes & blast radius

| Failure | Blast radius | Mitigation |
|---|---|---|
| One vLLM worker OOMs | One request, one replica | KServe restart, IGW reroutes. |
| One model SKU degrades | All tenants of that model SKU | Operator console isolates SKU; gateway returns 503 with `Retry-After`. |
| Gateway loses Postgres | New auth decisions fail; in-flight requests proceed | Cache last-known-good identity state in gateway, TTL 60s. Read-replica failover. |
| Metering pipeline backs up | Billing lag, no user-visible impact | Bounded queue with overflow to S3; replayable. |
| KV cache pool full | Latency spike for one model pool | Per-pool admission control with shed-to-503. |
| Whole cluster down | Region down | DR runbooks; multi-region is v2. |

## Performance budget (MVP target)

| Metric | Target |
|---|---|
| Gateway added p50 latency | < 5 ms |
| Gateway added p99 latency | < 20 ms |
| Metering ingest lag p99 | < 30 s |
| Auth lookup hit cache | < 1 ms |
| Auth lookup cold | < 10 ms |
| Console TTI | < 2 s on a warm session |

## What we explicitly are not designing

- **Cross-region active/active** for MVP. We design with multi-region in
  mind (clean separation of stateful vs stateless services; no
  cross-region required for any single request) but the GA artifact is
  active/passive with manual failover. Active/active is v2.
- **In-flight request migration** across workers. KServe handles replica
  failover; we do not attempt to relocate streaming connections.
- **A custom service mesh.** Standard K8s + Gateway API. If the neocloud
  runs Istio / Linkerd, we coexist; we do not require it.
