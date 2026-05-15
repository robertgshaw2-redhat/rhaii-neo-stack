# Business Model

This document describes **how Red Hat monetizes Neo Stack**, separately
from how the neocloud monetizes its end customers (covered in
[`pricing.md`](./pricing.md)).

## Pricing shape

A subscription with two components:

1. **Platform subscription** — annual, tiered by neocloud size and
   features. Covers the supported distribution, the operator console,
   the catalog, the consoles, the GitOps install, and Premium support.
2. **Token uplift** — a small per-million-tokens uplift on traffic that
   passes through the gateway. Aligns Red Hat revenue with the
   neocloud's success.

We are deliberately **not** monetizing per-GPU. The neocloud already
prices on GPUs; another GPU-denominated meter on top would be friction.

### Illustrative tiers

> Real numbers are set with finance and design partners. The shape is
> what matters here.

| Tier | Platform fee (annual) | Token uplift | Includes |
|---|---|---|---|
| Startup | $250k | $0.03 / 1M | Up to $10M annual neocloud token revenue. Stripe only. 8 hr support response. |
| Growth | $750k | $0.025 / 1M | Up to $50M annual. + Metronome / Orb. 4 hr support. |
| Scale | $2M | $0.02 / 1M | Up to $250M annual. Whitelabel domain. 1 hr support. Dedicated TAM. |
| Enterprise | Custom | Custom (lower at volume) | Multi-region, air-gap, FedRAMP packaging, co-eng hours. |

The token uplift is collected as a separate line item against the
neocloud, **not** against their end customers. The end customer always
just sees the neocloud's brand and the neocloud's price.

## Why this shape

- **Predictable for finance**: a real subscription anchor on day one.
- **Aligned over time**: as the neocloud's token business grows, Red
  Hat's revenue grows with it. We are not flat-fee insulated from their
  success or failure.
- **Defensible against build-internal**: the per-million-tokens uplift
  is small enough that it does not justify a 20-engineer in-house build
  for any but the very largest neoclouds. The math works out to under
  $1–2M of uplift per $50M of token revenue, vs. $20–40M to staff and
  carry an in-house equivalent for two years.
- **Transparent**: the uplift is visible in their console. We do not
  hide it inside the platform fee.

## What is *not* in the price

- The underlying GPU costs (the neocloud's iron).
- The neocloud's compliance certifications.
- Third-party model licensing for any models that require it. Catalog
  models are open-weight or have appropriate licenses; if the neocloud
  wants to host closed-weight models, they own that relationship.
- Pass-through fees from Stripe / Metronome / Orb.

## Sales motion

- **Direct field sales** for Tier-1 and Tier-2 neoclouds (~20 named
  accounts).
- **Channel** through hardware OEMs (Dell, HPE, Lenovo, Supermicro) for
  Tier-3 regional and sovereign neoclouds.
- **Co-sell with hyperscaler marketplaces** for hybrid sovereign
  deployments (e.g., a national-champion cloud that wants Neo Stack in
  their region plus an Azure burst).
- **Partner program for system integrators** (Accenture, Deloitte,
  TCS, Wipro) for regional rollouts that need integration services.

## Channel economics

We do not displace existing RH sales motions. Neo Stack is additive:

- **OpenShift AI subscription** is still sold; Neo Stack runs on top.
- **RH Inference Server** entitlement is still sold; Neo Stack consumes
  it.
- **Premium support** is still sold; Neo Stack support is a separate
  SKU with its own SLAs.

Internal compensation alignment is non-trivial — Neo Stack overlaps with
OpenShift AI, Ansible, and the OEM ecosystem teams. Expect to need a
clear comp policy in flight before launch.

## Service revenue

- **Onboarding services**: $50–250k engagement, sold by RH Consulting,
  for first-time install + cluster sizing + GTM-launch SE assist.
- **Custom-model onboarding** for design partners' specific BYO models.
- **Capacity-planning engagements** for neoclouds growing past Tier 3.

Service revenue is meaningful in the first 18 months and tapers as the
product hardens.

## Operating model

- Run as a **business unit inside Red Hat AI** with P&L visibility.
- Co-located with the RH AI Inference Server and llm-d teams; shares a
  release train where it makes sense.
- A small embedded GTM team (PMM, PMM-tech, two sales overlays at
  launch).
- A design-partner advisory council that meets quarterly.

## Risks to the business model

- **Hyperscaler bundling.** If Azure / AWS / GCP package a similar
  product into their managed K8s offers, the value of cross-cloud
  neutrality erodes. Mitigation: lean into on-prem + sovereign +
  multi-cloud.
- **Top-tier neoclouds vertical-integrating.** CoreWeave / Together
  build their own. Mitigation: Tier-2 / Tier-3 is the target; tier-1
  becomes design-partner-only.
- **Open-source displacement.** A community-built equivalent absorbs
  the value. Mitigation: keep contributing upstream; the value is in
  the productization, support, and consoles, which are non-trivial to
  reproduce.
- **NVIDIA bundles DGX Cloud Lepton with neoclouds.** Mitigation:
  hardware neutrality + open governance + RH brand neutrality.
- **Per-token billing model gets disintermediated** as agents do many
  small calls. Mitigation: token uplift is per million; agent traffic
  is more tokens, not fewer.
