# Red Hat AI Service Platform for Neoclouds

**Codename: Neo Stack**

> A turnkey commercial product that lets a neocloud stand up two new lines of
> business on its GPU fleet — a multi-tenant **Model-as-a-Service (MaaS)** API
> and self-service **Dedicated Endpoints** — by packaging, productizing, and
> extending the Red Hat AI stack (KServe, llm-d, vLLM, and the upstream
> [`opendatahub-io/models-as-a-service`](https://github.com/opendatahub-io/models-as-a-service)
> gateway built on Kuadrant + Authorino + Limitador).

This repository contains the first-draft product scope, reference architecture,
and roadmap. It is a planning artifact, not yet code.

> **Read this first:** [`docs/stack.md`](./docs/stack.md) — what Red Hat AI
> already ships vs. what Neo Stack adds. The runtime and the gateway exist
> upstream today. Neo Stack is the **commercial product layer** on top.

---

## TL;DR

Neoclouds rent GPUs. Their customers increasingly do not want GPUs — they want
**tokens** and **endpoints**. The gap between "I have an H200 cluster" and
"I run a credible inference business" is roughly 18–24 months of platform
engineering. Red Hat AI already closes most of that gap:

- **KServe + llm-d + vLLM** is the runtime.
- The upstream **`models-as-a-service`** project (Kuadrant + Authorino +
  Limitador + maas-api / maas-controller) is the multi-tenant gateway,
  auth, and rate-limit layer. It already does token tracking.

What it is **not** is a sellable, brandable, billable, fully-consoled
product a neocloud can hand to a customer. **Neo Stack** is the
commercial product wrapper that takes those existing pieces and adds the
**billing plane**, the **Dedicated Endpoints product line**, the
**curated model catalog**, the **whitelabel developer console**, the
**operator console**, the **reference integrations**, and the
**productization** (GitOps install, versioned upgrades, supported
distribution) needed to ship as a Red Hat product into the neocloud
channel.

We ship it as a subscription. Neoclouds get to market in a quarter instead of
two years. Red Hat captures a durable position in the fastest-growing layer of
the AI infrastructure stack.

---

## The two lines of business we enable

| | **Model-as-a-Service (MaaS)** | **Dedicated Endpoints** |
|---|---|---|
| **What the customer buys** | Tokens against shared, hosted open models | A reserved, single-tenant deployment of a model (or fine-tune) |
| **Pricing shape** | $ per million input/output tokens | $ per GPU-hour, with autoscale and scale-to-zero |
| **Who it's for** | App developers, agent builders, anyone who'd otherwise pay OpenAI | ML teams, regulated workloads, latency-sensitive workloads, BYO-model |
| **Why a neocloud wins** | Margin on idle capacity, anchor customers | High-value, sticky workloads at GPU-hour pricing they already understand |

A single platform serves both. The same model running on the same fleet can be
billed in either mode depending on which API the customer hits.

---

## Where to read next

Start here, in order:

1. [`PRODUCT.md`](./PRODUCT.md) — one-page product brief
2. [`docs/stack.md`](./docs/stack.md) — **what RHAI ships today vs. what Neo Stack adds** (the delta)
3. [`docs/vision.md`](./docs/vision.md) — why this, why now
4. [`docs/market.md`](./docs/market.md) — neocloud landscape and segmentation
5. [`docs/personas.md`](./docs/personas.md) — who uses Neo Stack
6. [`docs/capabilities.md`](./docs/capabilities.md) — the full capability map
7. [`docs/reference-architecture.md`](./docs/reference-architecture.md) — how it fits together
8. [`docs/product/maas-line.md`](./docs/product/maas-line.md) — MaaS product spec
9. [`docs/product/dedicated-endpoints-line.md`](./docs/product/dedicated-endpoints-line.md) — Dedicated Endpoints product spec
10. [`docs/roadmap/mvp.md`](./docs/roadmap/mvp.md) — what ships in v1
11. [`docs/roadmap/phases.md`](./docs/roadmap/phases.md) — phased roadmap to GA and beyond

Deeper material lives under [`docs/components/`](./docs/components),
[`docs/api/`](./docs/api), [`docs/ops/`](./docs/ops),
[`docs/business/`](./docs/business), and [`docs/adr/`](./docs/adr).

---

## Status

First draft. The intent of this PR is to give Red Hat AI product leadership
something concrete to react to — names, scope lines, and a roadmap they can
push back on — not to commit engineering or a ship date. Everything here is up
for revision.
