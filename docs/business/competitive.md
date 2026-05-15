# Competitive Landscape

This is the longer version of the competitive matrix in
[`gtm.md`](./gtm.md). For each competitor we record what they do, where
they win, where we win, and how we sell against them.

## NVIDIA — DGX Cloud Lepton

**What they are**: NVIDIA's productized inference and training cloud
sold through partner neoclouds. Includes a model gateway, catalog, and
endpoints.

**Strengths**:

- Tight NVIDIA hardware integration; optimal kernels and libraries.
- Vendor-of-record customer relationship with the developer.
- Brand pull.

**Weaknesses we sell against**:

- Locks the neocloud into NVIDIA exclusively. Any AMD, Intel Gaudi, or
  custom-silicon strategy is incompatible.
- NVIDIA fronts the customer relationship — the neocloud becomes
  capacity, not brand. Most neocloud CEOs reject this once they read
  the fine print.
- Closed-source. Difficult to fork, fix, or audit.

**Our sell**: hardware neutrality + brand neutrality + open governance.

## Together AI — selling its stack

**What they are**: Operates its own MaaS at scale; has signaled willingness
to license its stack to other clouds.

**Strengths**:

- Battle-tested at high QPS.
- Strong catalog of fine-tuned and optimized models.

**Weaknesses**:

- Together is a competitor to the neocloud's own end customers. They
  also serve developers directly. A neocloud licensing Together's stack
  is funding its own competitor.
- Closed-source / proprietary integrations.

**Our sell**: Red Hat is not in the end-customer market. We sell software,
period. No conflict of interest.

## Anyscale, Modal, BentoCloud, Baseten, Replicate

**What they are**: Developer-facing inference platforms. Anyscale is
Ray-centric, Modal is serverless-Python-centric, BentoCloud is wrapper
around BentoML, Baseten / Replicate are deployment platforms.

**Strengths**:

- Strong DX.
- Good for individual developers and small teams.

**Weaknesses**:

- Built for developers, not for cloud operators. Multi-tenant operator
  surface is thin or absent.
- Billing integrations are inward-facing (their billing) not
  outward-facing (your billing).
- Cannot be branded as the neocloud's product.

**Our sell**: Neo Stack is for *operators*. It is what a neocloud
deploys to be the platform Anyscale or Modal compete with — not what
they install instead.

## OpenShift AI alone

**What it is**: Red Hat's own platform-AI offering. Includes KServe,
model serving, pipelines, notebooks.

**Strengths**:

- It's Red Hat.
- Already adopted in regulated enterprises.

**Weaknesses we resolve**:

- No multi-tenant token product surface.
- No billing plane.
- No Dedicated Endpoints consumer-facing product line.
- No consoles for selling to external customers.

**How we coexist**: Neo Stack runs *on* OpenShift AI. We do not replace
it. Neo Stack adds the commercial product layer that OpenShift AI does
not address.

## Build in-house

**The default option** for every neocloud over $200M in revenue.

**Strengths**:

- Full control.
- No vendor lock-in.
- Hire ex-OpenAI / ex-Anthropic infra engineers.

**Weaknesses**:

- 18–24 months minimum to credible product.
- $20–40M to staff and carry for two years.
- Roadmap drag pulls scarce engineering away from the GPU business.
- Billing, abuse handling, and metering audit-grade are routinely
  underestimated.

**Our sell**: a 4-week-to-launch alternative at a cost that is rounding
error against the in-house option, with the supported upgrade path of
a Red Hat product.

## Hyperscaler-managed alternatives (Bedrock, Vertex, Foundry)

**What they are**: AWS, GCP, Azure's own MaaS platforms.

**Strengths**:

- Brand. Direct buyer relationship.
- Tight integration with each hyperscaler's other services.

**Weaknesses**:

- The neocloud is not a hyperscaler. This isn't a competitor *to* Neo
  Stack — it is a competitor to the neocloud's own MaaS product. We are
  in the same boat as the neocloud.
- Single-cloud. Many enterprises want non-hyperscaler placement.

**Our sell**: the reason a neocloud is launching MaaS in the first
place is to compete with Bedrock / Vertex / Foundry. Neo Stack is how
they do that without writing the software.

## Hugging Face Inference Endpoints

**What they are**: HF's hosted inference for HF Hub models.

**Strengths**:

- Best-in-class developer onboarding for HF users.
- Massive model catalog.

**Weaknesses**:

- Not a platform a neocloud can deploy and rebrand.
- HF is closer to a competitor than to a vendor at this scale.

**Our sell**: not applicable; different shape.

## Inferless / Mystic / fal / others

A long tail of inference-platform startups, mostly closed-source SaaS,
mostly developer-facing.

**Our sell**: same as Anyscale / Modal — these are end-customer
products, not operator products. They are who the neocloud's MaaS is
*competing with*, not who they should buy from.

## The matrix

| | Brand of end customer | Multi-tenant operator UX | Billing plane | DE product line | Hardware neutrality | Open source | Sovereignty story |
|---|---|---|---|---|---|---|---|
| Neo Stack | Neocloud | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| DGX Cloud Lepton | NVIDIA-fronted | partial | partial | ✅ | ❌ | ❌ | partial |
| Together stack | Together | partial | partial | partial | partial | ❌ | ❌ |
| Anyscale / Modal / Baseten | Anyscale / Modal / Baseten | ❌ | ❌ | partial | partial | partial | ❌ |
| Build in-house | Neocloud | ✅ if they build it | ✅ if they build it | ✅ if they build it | ✅ | depends | depends |
| Bedrock / Vertex / Foundry | Hyperscaler | ✅ | ✅ | ✅ | ❌ | ❌ | partial |
