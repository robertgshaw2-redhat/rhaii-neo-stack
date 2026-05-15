# Neo Stack — Product Brief

**One-pager for Red Hat AI leadership review.**

## Problem

Neoclouds (CoreWeave, Nebius, Crusoe, Lambda, Voltage Park, Nscale, RunPod,
Together, and a long tail of regional GPU providers) compete on GPU-hour price
and capacity. Margins on bare GPU-hours are compressing. Every neocloud
executive we talk to wants to move up the stack into **inference-as-a-product**
— sold per-token or per-endpoint — because the unit economics, stickiness, and
customer count are all materially better.

The blocker is not hardware. It is software:

- A production OpenAI-compatible API surface, with parity across chat,
  embeddings, batch, responses, files, and streaming.
- Multi-tenant identity, API keys, organizations, projects.
- Per-key and per-project quotas, rate limits, and spend caps.
- Accurate, audit-grade token metering with idempotent billing export.
- A curated, version-pinned model catalog with safety, performance, and
  cost-per-1M-tokens benchmarks the neocloud can stand behind.
- A self-service Dedicated Endpoints product: pick a model, pick a GPU SKU,
  attach a LoRA, get a URL — with autoscale, scale-to-zero, and a real SLO.
- A developer console (sign-up, keys, usage, playground, billing).
- An operator console (capacity, tenants, models, incidents, revenue).

Building this internally is an 18–24 month effort for a team of 30–60. Most
neoclouds will not (and should not) build it.

## Solution

Neo Stack is a **subscription software product** Red Hat sells to neoclouds.
It is **not** a greenfield platform; it is the commercial product wrapper on
top of pieces Red Hat AI already ships (see [`docs/stack.md`](./docs/stack.md)).

The runtime (KServe + llm-d + vLLM / RH AI Inference Server) and the gateway
(Kuadrant + Authorino + Limitador + maas-api/maas-controller from
`opendatahub-io/models-as-a-service`) are **upstream dependencies**. Neo Stack
adds:

1. **Productization wrapper** — a supported, versioned distribution of the
   above with a GitOps install bundle, tested upgrade matrices, and an
   air-gapped profile.
2. **Billing & metering plane** — turns the gateway's token events into
   invoices via Stripe / Metronome / Orb / S3 export, with idempotent
   reconciliation and spend-cap enforcement that loops back into Limitador.
3. **Dedicated Endpoints product line** — reserved-capacity, single-tenant,
   autoscale-and-scale-to-zero endpoints with LoRA hot-swap and BYO model
   support. Sits beside upstream MaaS, not inside it.
4. **Curated model catalog** — 8–10 models at MVP, 25+ at GA, each shipped
   with tuning profiles per accelerator, benchmark sheets, and model cards.
5. **Whitelabel developer console** — sign-up, keys, usage, catalog,
   playground, endpoint provisioning, billing portal — brandable end-to-end.
6. **Operator console** — tenants, capacity, catalog management, revenue,
   incidents, on-call.
7. **Reference integrations** — Stripe / Metronome / Orb for billing,
   Okta / Auth0 / Azure AD / Keycloak for identity, Datadog / Grafana /
   Dynatrace for observability.

Neo Stack is **branded by the neocloud**, not by Red Hat. The neocloud's
customers see "CoreWeave Inference" or "Nebius Tokens," not Red Hat. Red Hat
owns the platform underneath and the support contract.

## Why now

- llm-d is GA, in CNCF Sandbox, and has shipped to CoreWeave and Azure on
  managed Kubernetes. The runtime story is settled.
- Predicted-latency scheduling (v0.7, May 2026) and P/D disaggregation
  finally make multi-tenant inference economically viable at neocloud scale.
- Hyperscalers (Bedrock, Vertex, Azure AI Foundry) are pulling token revenue
  away from neoclouds. Without a product layer, neoclouds are stuck selling
  raw H100/H200/B200 hours into a buyer's market.
- The market is large enough to support 8–12 token-selling neoclouds. Most
  of them will not build the software themselves.

## Why Red Hat

- We already ship the entire runtime — vLLM, llm-d, KServe, Inference Gateway,
  OpenShift AI.
- We have the operator muscle. Neoclouds want a vendor with a 24×7 phone number
  for their inference plane, not a YC startup.
- Hybrid cloud and on-prem are part of our DNA. Neocloud regions look more like
  on-prem than like AWS. Our packaging fits.
- We don't compete with neoclouds for end customers. A hyperscaler offering
  this product would be conflicted. We are not.

## Commercial shape

- **Pricing**: subscription with a fixed platform fee + a per-token "platform
  uplift" component (small, e.g., $0.02 per 1M tokens served through the
  Gateway) to align our revenue with their growth.
- **Form factor**: GitOps-installable bundle on the neocloud's existing
  Kubernetes (OpenShift preferred; vanilla K8s supported), with a Red Hat-run
  SaaS control plane option for smaller neoclouds.
- **Support**: standard Red Hat Premium + a dedicated TAM for the first year.

## Risks

- **Build-vs-buy at top-tier neoclouds.** CoreWeave and Together can and may
  build this themselves. We design for the segment below them — the 20–30
  neoclouds that cannot.
- **Hyperscaler bundling.** Azure and GCP could bundle similar functionality
  with their managed K8s. Our answer: be the cross-cloud / on-prem option,
  and stay neutral on the underlying cloud.
- **NVIDIA NIM / DGX Cloud Lepton.** Single-vendor stack. We compete on
  hardware-independence (AMD, Intel Gaudi, NVIDIA) and openness (llm-d, vLLM,
  KServe are all upstream).

## What we are asking for

A green light to take this scope to three design-partner neoclouds for
validation in the next 30 days, and a staffing plan for the MVP described in
[`docs/roadmap/mvp.md`](./docs/roadmap/mvp.md).
