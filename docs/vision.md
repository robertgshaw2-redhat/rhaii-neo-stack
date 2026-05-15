# Vision

## Where the AI infrastructure market is heading

By the end of 2026 the AI infrastructure stack has split into four
recognizable layers, each with its own buyer, margin profile, and dominant
vendor set:

1. **Silicon & systems.** NVIDIA, AMD, Intel, custom (TPU, Trainium, MTIA).
   Sold to hyperscalers and neoclouds. Margins set by supply.
2. **GPU-as-a-service / capacity.** Hyperscalers and neoclouds. Sold to
   companies that still want raw infrastructure. Margins compressing as
   capacity comes online.
3. **Inference-as-a-product.** Tokens and endpoints, sold to application
   developers. Margins healthy and expanding. Dominated today by OpenAI,
   Anthropic, Google, plus Fireworks / Together / Groq / Cerebras on open
   models. Hyperscalers participate via Bedrock / Vertex / Foundry.
4. **Applications.** Agents, copilots, vertical AI. Where the long-tail
   spend ultimately lands.

Neoclouds today sit firmly in **layer 2**. Their customers — and their boards
— are pulling them toward **layer 3**, because:

- Per-token gross margins are 60–80% vs. 25–45% on bare GPU-hours.
- Per-token customers are 10–100× more numerous than per-GPU-hour customers.
- Per-token contracts are stickier (an API key with usage history is harder
  to migrate than a Kubernetes namespace).
- Per-token revenue is more predictable and easier to forecast.

Every credible neocloud roadmap we have seen includes some version of "launch
a tokens product." Few of them are credible plans, because the platform work
is huge and the talent to do it is scarce and expensive.

## What Red Hat sees

We sit in a privileged position: we already ship the entire runtime that
companies like Fireworks and Together built internally over the last three
years. vLLM is the default inference engine. llm-d is the default
distributed-inference orchestrator. KServe is the default Kubernetes serving
abstraction. The Inference Gateway is the default routing layer.

What we **don't** ship — yet — is the **product layer** that turns those into
a sellable AI cloud. That layer has roughly the same shape for every neocloud
that wants to build one. It is a textbook packaging opportunity.

## What Neo Stack is

Neo Stack is the product layer Red Hat ships on top of the Red Hat AI runtime,
sold to neoclouds, so they can sell tokens and endpoints under their own
brand. It bundles:

- Multi-tenant identity, organizations, projects, API keys.
- A production OpenAI-compatible API surface.
- Quotas, rate limits, spend caps, abuse controls.
- Audit-grade token metering and billing export.
- A curated model catalog with versioning, benchmarks, and SKUs.
- A self-service dedicated-endpoints product with autoscale and scale-to-zero.
- Developer and operator consoles.
- Reference integrations with the billing / IAM / observability tools
  neoclouds already use.

## What Neo Stack is not

- **Not a competitor to neoclouds.** We do not run our own MaaS endpoint or
  resell their tokens. We sell them software.
- **Not a competitor to OpenAI / Anthropic.** We do not train models. We
  curate and host open ones.
- **Not a hosted SaaS only.** While we offer a hosted control-plane option
  for small neoclouds, the supported deployment shape is "in your cluster,
  on your iron, in your region."
- **Not a fork of the upstream.** Everything Neo Stack consumes — vLLM,
  llm-d, KServe, IGW — stays upstream. Neo Stack is the packaging, control
  plane, and consoles around them.

## The three-year picture

- **2026 (year 1):** Launch with 3 design-partner neoclouds. MVP scope is
  MaaS for 8–10 curated models + Dedicated Endpoints for those same models.
  Stripe + Okta + Datadog reference integrations. GA by Q4 2026.
- **2027 (year 2):** BYO-model with safety scanning. Fine-tuning service.
  Multi-region. Embeddings, reranking, speech models. 15–20 neocloud
  customers. Partner with one or two billing vendors for tighter integration.
- **2028 (year 3):** Agent / Responses API, batch and async inference at
  scale, tool-use brokering, RAG primitives. Federated multi-cloud
  inference (route a customer's tokens across multiple neoclouds based on
  capacity and price). Become the de-facto control plane for non-hyperscaler
  inference.

## Strategic value to Red Hat

- Adds a product-led, usage-coupled revenue line to a portfolio that is
  otherwise priced on cores and entitlements.
- Anchors OpenShift AI in the fastest-growing buyer segment in cloud
  infrastructure.
- Creates a credible, neutral alternative to hyperscaler-bundled inference
  for enterprises that want sovereignty or hybrid placement.
- Strengthens upstream investment in vLLM, llm-d, and KServe by giving them
  a directly monetizable surface.

## What this document is not committing to

Pricing numbers, customer counts, and timelines in this repo are
illustrative. The point of the first draft is to align on **shape, scope,
and strategic intent**. Numbers come from design-partner conversations.
