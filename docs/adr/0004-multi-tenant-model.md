# ADR-0004: Multi-tenant isolation model

## Status

Accepted.

## Context

The product runs two lines of business with different isolation
requirements on the same fleet:

- **MaaS**: shared, multi-tenant. Tens of thousands of tenants per
  cluster eventually. Isolation must be cheap.
- **Dedicated Endpoints**: single-tenant per endpoint. Hundreds to
  thousands of endpoints per cluster. Isolation must be strong.

A single isolation policy that works for both would fail one of them.
Two policies on the same fleet is what we need.

Available isolation primitives:

1. Same pod, gateway-only separation (cheapest, weakest).
2. Same namespace, per-tenant ServiceAccount + NetworkPolicy.
3. Per-tenant namespace + NetworkPolicy + PSS (strong, K8s-standard).
4. Per-tenant nodes (very strong, very expensive).
5. Per-tenant clusters (overkill except for very large enterprises).

## Decision

We use **two isolation profiles** simultaneously on the same fleet:

### MaaS isolation profile

- One namespace per model SKU pool (not per tenant).
- Workers serve requests for many tenants concurrently.
- Tenant separation is at the **gateway** (Authorino + Limitador) and
  at the **KV cache partition layer** (per-tenant cache keys).
- Per-tenant labels propagate on every request for observability.
- Prefix-cache reuse across tenants is **opt-in** per tenant — the
  default is no reuse to avoid even theoretical cross-tenant leakage.

This is acceptable because:

- vLLM's per-request state isolation is good; no shared mutable state
  between requests of different tenants by default.
- KV cache content is non-recoverable across tenants when reuse is off.
- The blast radius of a worker compromise is the requests-in-flight on
  that worker, not the cluster.

### Dedicated Endpoints isolation profile

- One namespace per tenant.
- One `InferenceService` per endpoint inside the tenant namespace.
- NetworkPolicy default-deny with explicit allow-lists.
- PodSecurityStandards: restricted.
- Per-tenant ServiceAccount, never shared.
- KV cache partition is per-endpoint; **no** cross-tenant reuse.
- Per-GPU pinning available (GA) for customers who pay for it.

This is acceptable because:

- DE customers pay for isolation and expect it.
- The cost of the namespace overhead is small relative to GPU cost.
- We can audit per-namespace easily.

## Consequences

- The same cluster runs two policies with different security postures.
  Operators need to understand the difference.
- We document the difference explicitly in tenant-facing material:
  "MaaS is multi-tenant by design; if you need single-tenant, use a
  Dedicated Endpoint."
- Compliance posture is asymmetric: HIPAA / regulated workloads should
  use DE.
- The MaaS pool's "prefix cache reuse" feature is gated behind
  per-tenant opt-in, which is friction. Acceptable for MVP and GA.
- Per-GPU pinning at DE (GA) requires capacity-aware scheduling. The
  endpoint controller learns about GPU SKUs available and rejects
  endpoint creation if no compliant GPU is reservable.

## Notes

- Future hardware isolation (e.g., NVIDIA MIG slicing, confidential
  compute) is additive to this model. The endpoint CRD already has a
  `tenancy.sharedGpuAllowed` field and can be extended.
- We will revisit "cross-tenant prefix cache reuse" at GA based on
  empirical performance gains and customer comfort.
