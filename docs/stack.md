# What Red Hat AI ships today vs. what Neo Stack adds

This document is the single source of truth for the **delta** Neo Stack
represents on top of the existing Red Hat AI portfolio. Anything in the
"already shipped" column is a dependency we consume, not a thing we build.

## What Red Hat AI already ships

Neo Stack is not a greenfield platform. The runtime, the gateway, the auth
plane, the rate limiter, the model-serving operator, and the multi-tenant
MaaS controller all exist today inside the Red Hat AI portfolio.

### Inference runtime

- **vLLM** — high-throughput inference engine.
- **Red Hat AI Inference Server** — supported, hardened vLLM build.
- **llm-d** — distributed inference orchestration on Kubernetes. P/D
  disaggregation, KV-cache pooling, predicted-latency scheduling (GA in
  v0.7, May 2026), batch gateway (experimental).
- **KServe** — Kubernetes model serving abstraction.

### Gateway, auth, rate limit, token tracking

The OpenDataHub
[`models-as-a-service`](https://github.com/opendatahub-io/models-as-a-service)
project, which underlies Red Hat AI's MaaS layer, already provides:

| Concern | Shipped by | Notes |
|---|---|---|
| API gateway / routing | **Kuadrant** + Gateway API | Policy-driven traffic management. |
| AuthN/AuthZ | **Authorino** | API keys, OpenShift tokens, header stripping. |
| Rate limiting | **Limitador** | RPM / TPM / concurrent limits. |
| Multi-tenant key management | **maas-api** | Go service over Postgres. |
| Tenant reconciliation, model lifecycle | **maas-controller** | K8s controller. |
| Internal + external model registration | **maas-controller** | KServe-backed and external providers. |
| Per-request token accounting | shipped, surfaced through the gateway path |
| Reference React console | shipped, intended as a starting point |

The MaaS project is explicit that it is **not yet production-ready** and
is intended as the foundation, not the finished product. That is the
opportunity Neo Stack addresses.

### Cluster platform

- **OpenShift** (with OpenShift AI) — preferred substrate.
- **GPU operators** — NVIDIA, AMD, Intel Gaudi.

### Tooling

- OpenShift GitOps (Argo CD) for install.
- OpenShift Pipelines (Tekton) for model-artifact promotion.
- OpenShift Service Mesh (Istio) optionally.

## What Neo Stack adds

Everything below is the delta. None of it exists in productizable form in
the current portfolio; building it is the work this repository is
scoping.

### 1. A commercial product wrapper around upstream MaaS

The upstream `models-as-a-service` repo is a working platform, not a
sellable product. Neo Stack adds:

- A supportable, versioned distribution of MaaS pinned against specific
  Kuadrant / Authorino / Limitador / KServe / llm-d versions.
- A documented upgrade path between versions.
- Tunable defaults and reference profiles per accelerator × model SKU.
- The Red Hat support contract and SLAs that neoclouds need to commit
  to their own customers.

### 2. Billing & monetization plane

Upstream MaaS counts tokens. It does not turn those tokens into invoices.
Neo Stack adds:

- **Metering pipeline.** Ingests the per-request token events from
  Kuadrant / maas-api, deduplicates them, computes priced events, and
  stores hot (Postgres) + cold (ClickHouse).
- **Billing service.** Per-model price book, per-tenant overrides,
  prepaid credits, postpaid invoicing, spend caps with enforcement that
  feeds back into Limitador.
- **Connectors.** Stripe (MVP), Metronome and Orb (GA), generic
  S3/CSV export (MVP), NetSuite/SAP via partner (v2).
- **Revenue reporting** for the neocloud operator.
- **Audit-grade idempotence** with cursor-based reconciliation suitable
  for finance and external audit.

### 3. Dedicated Endpoints product line

Upstream MaaS is, by design, a multi-tenant model-as-a-service layer. The
Dedicated Endpoints product — reserved capacity, per-endpoint SLO,
scale-to-zero, LoRA hot-swap, BYO model — is **not** in the upstream MaaS
scope. Neo Stack adds:

- An endpoint-lifecycle controller that sits beside maas-controller and
  manages per-tenant KServe `InferenceService`s with per-endpoint
  policy.
- Endpoint provisioning UX (developer console + Admin API).
- Per-endpoint autoscaling integration with KServe / Knative + a TPOT-
  and queue-depth-aware policy.
- LoRA adapter management and hot-swap routing through the gateway.
- BYO model intake (signature verification, license check, health
  probe, private SKU publishing).
- Per-endpoint SLO templates and credit grids.

### 4. Curated model catalog

Upstream MaaS supports registering models; it does not curate them. Neo
Stack adds:

- A **shipped catalog** of 8–10 models (MVP) and 25+ (GA), each with a
  signed artifact, tuning profile per accelerator, benchmark sheet
  (throughput, TTFT, TPOT, $/1M-tokens), and a model card.
- Quarterly catalog refresh as part of the subscription.
- Operator console workflow to promote / demote / hide catalog SKUs.

### 5. End-customer developer console

The upstream React UI is a starting point. Neo Stack adds a
**whitelabel-ready developer console** with:

- Sign-up, login, project management.
- API key issuance and scope management (project, model, endpoint).
- Usage dashboards (tokens, latency, spend).
- A model catalog browser with pricing and benchmarks.
- A playground for chat / completion / tool-call / embeddings.
- Dedicated Endpoint provisioning UI.
- A billing portal (credit cards, prepaid credits, invoices).
- A logs explorer (opt-in).
- Branding system (logo, colors, custom domain) so the neocloud's
  customers never see "Red Hat" or "Neo Stack."

### 6. Operator console

There is no upstream operator console. Neo Stack adds:

- Tenant list, search, drill-in, impersonation (audited).
- Capacity and utilization across pools, per accelerator SKU.
- Per-tenant SLO dashboards.
- Catalog management.
- Revenue / margin dashboards.
- Incident and on-call view.
- Tenant kill-switch and graceful drain.

### 7. Reference integrations

Each of these is an integration kit (Helm values, secrets contract,
runbook, smoke tests), not a from-scratch build:

- IdP: Okta, Auth0, Azure AD, Keycloak.
- Billing: Stripe, Metronome, Orb.
- Observability: Datadog, Grafana, Dynatrace.
- Object storage: S3-compatible.
- Notification / status: Statuspage, Incident.io.

### 8. Productization plumbing

- One-shot **GitOps install bundle** that lays down Kuadrant,
  Authorino, Limitador, KServe, llm-d, the maas-* components, plus the
  Neo Stack additions, configured to talk to each other.
- Tested **upgrade matrices** between Neo Stack releases.
- Air-gapped install profile for sovereign neoclouds.
- A neocloud-branded **status page generator** that pulls from
  per-SKU SLOs.

## Visualizing the delta

```
                ┌─────────────────────────────────────────────────────────┐
                │                     END CUSTOMERS                       │
                └────────────────────────────┬────────────────────────────┘
                                             │
                ┌────────────────────────────┼────────────────────────────┐
                │                NEO STACK (this repo's scope)             │
                │                                                          │
                │  Developer console       Operator console                │
                │  Billing svc / metering  Catalog curation                │
                │  Dedicated Endpoints     Reference integrations          │
                │  Productization & GitOps install bundle                  │
                └────────────────────────────┬────────────────────────────┘
                                             │  (consumes upstream APIs)
                ┌────────────────────────────┼────────────────────────────┐
                │     RED HAT AI today (consumed, not rebuilt)             │
                │                                                          │
                │  opendatahub-io/models-as-a-service                      │
                │   ├─ Kuadrant + Authorino + Limitador (gateway/auth/RL)  │
                │   ├─ maas-api (keys)                                     │
                │   └─ maas-controller (tenants, models)                   │
                │                                                          │
                │  KServe   llm-d   vLLM / RH AI Inference Server          │
                │  OpenShift AI    GPU operators                           │
                └─────────────────────────────────────────────────────────┘
                                             │
                ┌────────────────────────────┼────────────────────────────┐
                │              NEOCLOUD INFRASTRUCTURE                     │
                │  K8s, GPUs, storage, networking, IdP, billing system     │
                └─────────────────────────────────────────────────────────┘
```

The implication for product strategy is that Neo Stack ships **faster**
and **leaner** than a greenfield platform would. The runtime and gateway
layers exist; the work is the wrapper, the commercial plane, and the DE
product line.

## Implications for staffing & timeline

Because the runtime + gateway are upstream-supplied:

- We do not need a vLLM team, an llm-d team, a gateway team, or a
  rate-limit team. We have an integration / SRE relationship with those
  teams.
- We **do** need:
  - A control-plane / billing team (Go, Postgres, ClickHouse).
  - A consoles team (Next.js, design).
  - A catalog & benchmarking team (small, ML / perf-engineering).
  - A productization / GitOps / SRE team.
  - A product / PMM team for GTM.
- A reasonable MVP team is ~25–35 people; a greenfield equivalent would
  be 60–90.
