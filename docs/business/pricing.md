# Pricing Reference (for the neocloud's end customers)

This is **not** how Red Hat prices Neo Stack to the neocloud — that lives
in [`business-model.md`](./business-model.md). This is the **reference
price book** Neo Stack ships, updated quarterly, that a neocloud can
adopt, edit, or override.

## Why we ship a reference price book

Neoclouds need to launch with credible prices on day one. They do not
have years of cost data, competitive intelligence on every model, or
benchmarking capacity to determine list prices. Neo Stack ships a
reference and the neocloud picks a multiplier.

## MaaS reference (illustrative, May 2026)

| Model class | Example | Input $/1M | Output $/1M | Cached input $/1M |
|---|---|---|---|---|
| Small chat | 8B-class | $0.05 | $0.10 | $0.025 |
| Medium chat | 30–40B-class | $0.18 | $0.55 | $0.09 |
| Large chat | 70B-class | $0.35 | $1.10 | $0.18 |
| Frontier-open | 400B-MoE-class | $0.90 | $3.00 | $0.45 |
| Reasoning | reasoning-tuned 70B | $0.55 | $2.40 | $0.28 |
| Embeddings small | bge-small-class | $0.015 | — | — |
| Embeddings large | e5-large / multilingual | $0.06 | — | — |
| Reranker | bge-reranker-class | $0.10 | — | — |

Numbers reset every quarter based on observed market prices. The neocloud
can:

- Adopt the reference.
- Multiply by a coefficient (e.g., 0.85× to undercut, 1.10× to match
  a premium positioning).
- Override per model per tenant.

## Dedicated Endpoints reference

Per GPU-hour, markup over the neocloud's raw GPU cost:

| Accelerator | Reference markup |
|---|---|
| H100 80G | 1.6×–2.2× |
| H200 | 1.5×–2.0× |
| B200 | 1.4×–1.9× (early; thin supply) |
| MI300X | 1.5×–2.0× |
| Gaudi3 | 1.5×–2.0× |

Optional per-token uplift on DE: $0.05–$0.20 per 1M output tokens for
parity-with-MaaS billing. Most DE buyers reject this; it is opt-in per
endpoint.

Optional idle-warm fee: 40–60% of running rate for replicas kept warm
during idle windows.

## Discounts the operator can configure

- **Volume tiers**: e.g., -10% past 100M tokens / month, -20% past 1B.
- **Annual commit**: e.g., -15% on a 12-month commit at a usage floor.
- **Prepaid credits**: e.g., -5% on $10k+ prepaid balance.
- **Cached-input discount**: default 50%, configurable 0–80%.
- **Promotional codes**: free-tier credit grants, conference codes,
  partner channel codes.

## What we do not ship

- **A list of "fair" prices**. This is a competitive market; prices move.
- **Geo-specific pricing**. Out of scope for MVP; GA target.
- **Hidden fees**. The end customer's invoice line-items map 1:1 to
  metered events with no fudge factor.

## Surfacing the price book in the consoles

- Developer console: catalog page shows current effective price per
  model with per-org overrides applied.
- Operator console: price book editor; per-tenant overrides; preview of
  what an existing tenant would pay under a proposed change.
- API: read-only `GET /v1/pricing` endpoint authenticated to a project,
  returning effective prices for that project.

## Audit story

Every price change writes an entry to the audit log including the actor,
the previous price, the new price, the effective time, and which tenant
overrides exist. Billing always reprices off the price-as-of-event-time,
not the price-now. This is the only way to defend invoices.
