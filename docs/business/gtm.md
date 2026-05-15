# Go-to-Market

## Launch sequence

### Phase 0 — Design partners (now)

- Identify 3 design-partner neoclouds. Profile: Tier-2, want tokens, no
  in-flight in-house platform that has shipped, friendly to Red Hat.
  Strawman list:
  - Voltage Park (US, AI training & inference, has appetite for tokens).
  - Nscale (UK / Europe, sovereignty story, growing fast).
  - A regional sovereign neocloud (G42, Stratus, or a European national
    champion).

  Tier-1 design partner (optional, valuable as a marquee): **CoreWeave**.
  Already has llm-d. May want to use Neo Stack components rather than
  the full bundle. Acceptable.

- Each design partner signs a no-fee LOI in exchange for:
  - Early access.
  - Direct line to RH AI eng and PM.
  - Joint launch announcement when MVP ships.
  - The right to negotiate non-standard pricing for the first 12
    months of GA.

- We commit to:
  - On-site SE for install.
  - Weekly product office hours.
  - Custom catalog additions on request (within reason).
  - A clear roadmap visibility window.

### Phase 1 — Closed beta launch

- All three design partners in production with paying internal
  customers.
- Joint press release with at least one design partner.
- Booth at KubeCon NA + Red Hat Summit demonstrating the operator
  console.
- First-customer story in the keynote at Red Hat Summit.

### Phase 2 — GA launch

- Public price list (the platform-fee tiers).
- Expanded catalog (25+ models).
- 8–12 paying neocloud customers by 12 months post-GA.
- Joint logo program for "Built on Neo Stack" (subtle, in the neocloud's
  marketing materials, optional).
- Co-marketing with billing partners (Stripe, Metronome, Orb).

## Messaging architecture

The product has different stories for different audiences. Do not blur
them.

### To the neocloud CEO

> Stop selling GPU-hours and start selling tokens. We're the software
> that gets you there in a quarter, with your name on it, our support
> contract, and predictable economics.

### To the neocloud VP Product

> Two product lines on one platform, with the consoles and the billing
> already built. Time-to-launch in weeks, not years. Comparable margin
> profile to OpenAI / Anthropic on the open models.

### To the neocloud VP Engineering

> Runs on your Kubernetes. Open-source-native. No proprietary CRDs we
> haven't upstreamed. GitOps install. We do not sit between you and the
> metal; we sit between you and your customer.

### To the neocloud's end customers (positioned by the neocloud)

> An OpenAI-compatible API, on dedicated hardware in a region you can
> name, with the open models your team actually wants to use, at prices
> that beat the hyperscalers.

Neo Stack and Red Hat names do not appear in end-customer-facing
materials by default. Operators may credit us; most will not.

## Channels

| Channel | Use | Comp |
|---|---|---|
| Red Hat direct field sales | Tier-1 / Tier-2 neoclouds, named accounts | Sales + overlay |
| Hardware OEMs (Dell, HPE, Lenovo, Supermicro) | Tier-3 regional | OEM co-sell |
| Hyperscaler marketplaces (Azure, AWS, GCP) | Sovereign hybrid plays | Marketplace fees |
| System integrators (Accenture, Deloitte, TCS, Wipro) | Sovereign / national-champion | Partner margin |
| Red Hat partner ecosystem | Existing Red Hat customers spinning up internal "neocloud" (e.g., a bank's internal AI cloud) | Standard |

## Reference customer plan

For each design partner, plan and execute:

1. A solution brief written jointly.
2. A KubeCon or Red Hat Summit talk.
3. A vanity URL for an end-customer-facing case study.
4. A logo in the Neo Stack web property.

Three reference customers is the minimum to make Tier-3 sales tractable
on its own.

## Competitive positioning sheet

| If a prospect is comparing… | Lead with… |
|---|---|
| Building in-house | "18 months and $30M, or 4 weeks and a license fee." |
| NVIDIA DGX Cloud Lepton | "Your brand, not theirs. Your hardware mix, not theirs." |
| Hyperscaler-managed (Bedrock, Vertex, Foundry) | "On your iron, in your region, under your contract." |
| Anyscale / Modal / BentoCloud | "Built for cloud operators, not for individual developers." |
| Together's stack | "Neutral software vendor, no end-customer conflict." |

## Launch risks

- **Naming**. "Neo Stack" is a placeholder. Final name should not lead
  with "Red Hat" because the product is sold *to* neoclouds who will
  white-label it. "Red Hat AI Service Platform" works internally but
  doesn't appear in end-customer surfaces.
- **Channel conflict** with OpenShift AI selling motions. Needs comp
  clarity from day one.
- **Design-partner expectation management**. Each DP will want bespoke
  features. PM rigor required.
- **Catalog legal review**. The catalog ships with model weights and
  licenses. Each model needs sign-off, and that work is on the critical
  path to MVP.
