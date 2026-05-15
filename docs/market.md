# Market

## Sizing

Independent analysts put 2026 neocloud revenue between **$20B (Forrester)**
and **$35B (Mordor)**, growing 150–500% year over year for the largest
players. CoreWeave alone is on a ~$10B run rate by mid-2026; Nebius's
projected 2026 growth is 200%+.

The relevant *addressable* market for Neo Stack is the portion of that
revenue that the neocloud will eventually re-sell as tokens or endpoints
rather than as raw GPU-hours. Industry conversations and design-partner
interviews put that at **15–35% of neocloud revenue by end of 2027**,
implying a $4–12B token-and-endpoint TAM that today is largely unserved by
the neoclouds themselves and is being captured by OpenAI / Anthropic /
hyperscaler-managed model APIs instead.

Neo Stack monetizes a slice of that flow — both via subscription fees and
via a small per-token uplift on the gateway. A 1% capture of $8B of
neocloud-served token revenue is $80M ARR. That is the order of magnitude
we should be planning against, not the headline neocloud revenue number.

## Segmentation

We segment neoclouds into four tiers. Neo Stack is not built for all of them.

### Tier 1 — Hyperscale Neoclouds

CoreWeave, Nebius, Crusoe, Lambda.

- **Revenue**: > $500M run rate.
- **Engineering capacity**: 200+ infra/platform engineers.
- **Likely behavior**: build their own MaaS platform, or buy components,
  or partner deeply with one or two open-source projects.
- **Our play**: design partners. Sell them llm-d and Inference Gateway
  support contracts; let Neo Stack itself be an "available but not
  necessary" SKU. Use them as proof points. CoreWeave already runs llm-d.

### Tier 2 — Scaling Neoclouds

Voltage Park, Nscale, Together AI (as infra), Applied Digital, Sustainable
Metal Cloud, FluidStack, Vultr, regional sovereign clouds (G42, Mistral
Compute, Stratus, etc.).

- **Revenue**: $50M–$500M run rate.
- **Engineering capacity**: 20–100 infra engineers, no dedicated AI
  platform team, or one being assembled.
- **Likely behavior**: want to launch tokens in the next 12 months. Will
  buy software if it gets them there in a quarter instead of two years.
- **Our play**: **primary target.** Neo Stack collapses 18 months of
  build into a deployable bundle. This is where we win the next 24 months.

### Tier 3 — Regional & Sovereign Neoclouds

National-champion clouds, telco-owned GPU clouds, university and research
clouds (Pawsey, EuroHPC affiliates), corporate-spinout neoclouds.

- **Revenue**: $5M–$50M run rate.
- **Engineering capacity**: small. Often partner-led.
- **Likely behavior**: want tokens, often with sovereignty / residency
  constraints. Sensitive to vendor lock-in. Will value Red Hat's neutrality.
- **Our play**: **secondary target.** Often sold via partners
  (system integrators, hardware OEMs). Hosted control-plane option matters
  more here.

### Tier 4 — GPU Marketplaces

RunPod, Vast.ai, Salad. Long-tail self-service capacity, often with
consumer GPUs.

- **Revenue**: highly variable.
- **Likely behavior**: marketplace dynamics, not service dynamics. Tokens
  product is a stretch.
- **Our play**: **not target.** Optional thin community edition later.

## Competitive landscape

### Direct alternatives a neocloud might pick instead of Neo Stack

- **Build internally.** The default option. 18–24 month timeline, 20–60
  engineers. Most neoclouds underestimate the metering, billing, and
  abuse-handling cost.
- **NVIDIA DGX Cloud Lepton.** Strong runtime, but locks the neocloud
  into a single accelerator vendor and an NVIDIA-fronted relationship
  with end customers. Conflicts with neocloud branding.
- **Anyscale / BentoCloud / Modal-style platforms.** Built for developers,
  not for cloud operators. Wrong shape — no multi-tenant operator surface,
  no neocloud-branding model, no billing integration story.
- **Hugging Face Inference Endpoints under-the-hood OEM.** Possible but
  HF is a competitor, not a vendor.
- **Together's stack-as-a-service ambitions.** Together has hinted at
  selling its stack to other clouds. Conflict of interest; they want the
  end-customer revenue.

### Where Red Hat / Neo Stack wins

- **Hardware-neutral.** vLLM and llm-d run on NVIDIA, AMD, Intel Gaudi,
  AWS Trainium. We do not constrain the neocloud's procurement strategy.
- **Open-source-native.** Every layer is upstream. No lock-in story to
  defend against in procurement.
- **Cloud-neutral.** We run on the neocloud's K8s, wherever it sits. No
  hyperscaler conflict.
- **Operator-grade.** OpenShift's posture on lifecycle, upgrades,
  multi-tenancy, audit, and CVEs is mature in a way startup platforms
  cannot replicate quickly.
- **Distribution.** Red Hat sales already calls into the customers
  neoclouds want to land (regulated enterprises, federal, telco). Neo
  Stack becomes a co-sell channel back to neocloud customers.

### Where Neo Stack does **not** win

- **Tier-1 neocloud labs.** A platform team of 200 will build the in-house
  version anyway. We meet them with components, not with the product.
- **Pure consumer / marketplace plays.** Different shape of customer.
- **Trained model differentiation.** We do not have a frontier model.
  The neocloud isn't buying one from us either.

## What we are betting on

1. The neocloud market continues to fragment. There will be 10–25
   credible neoclouds globally by end of 2027, not 3.
2. End customers will continue to want neutral, non-hyperscaler options
   for sovereignty, price, and BYO-hardware reasons.
3. Tier-2 and Tier-3 neoclouds value time-to-market more than cost of
   software. A $2M/year platform subscription is rounding error against
   the revenue it unlocks.
4. The runtime layer (vLLM, llm-d, KServe) keeps consolidating. We win
   not because our runtime is best — though it is competitive — but because
   nobody else packages the service layer on top of it.
