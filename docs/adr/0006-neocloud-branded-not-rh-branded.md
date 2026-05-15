# ADR-0006: Neo Stack is sold to neoclouds; the neocloud's brand fronts the customer

## Status

Accepted.

## Context

A platform built on the Red Hat AI stack and sold by Red Hat could
plausibly be branded several ways:

1. As **Red Hat's** product, with neoclouds as channel partners selling
   "Red Hat AI Cloud on CoreWeave" / "on Nebius" etc.
2. As the **neocloud's** product, with Red Hat as an underlying vendor
   not surfaced to the end customer.
3. As a **co-branded** product (both logos in front of the customer).
4. As **invisible infrastructure** with the neocloud entirely in control
   of branding.

NVIDIA chose option 1 with DGX Cloud Lepton. The result is the
neocloud becoming "capacity" and NVIDIA becoming "the AI cloud," which
many neoclouds have rejected.

Hyperscalers don't have this problem — they own the customer end-to-end.

Red Hat has historically been the **invisible enterprise infrastructure**
vendor: the OS, the K8s, the middleware. Customers know Red Hat as the
support contract, not as the brand they buy services from.

## Decision

Neo Stack is sold **to** the neocloud and **branded by** the neocloud
to end customers. The neocloud's name, logo, and identity front every
end-customer surface — the console, the API hostname, the docs, the
invoices. Red Hat appears in:

- The support contract with the neocloud.
- The operator console (Neo Stack name in the corner of internal pages,
  if the operator wants it).
- Optional, removable credit in the developer-console footer.
- Joint marketing where both parties opt in.

Red Hat does not appear in:

- The end customer's developer console by default.
- API responses or documentation.
- Invoices.
- Any place we don't explicitly control via an integration.

## Consequences

- Marketing materials, sales motions, and pricing all flow through the
  neocloud's existing surfaces. Red Hat marketing's path to the end
  customer is indirect (via the neocloud) by design.
- The reference customer logo program is "the neocloud's logo on our
  site," not "our logo on the neocloud's site."
- We need a robust whitelabel system in the consoles from day one.
- Channel-conflict risk with OpenShift / OpenShift AI is real;
  internal comp policy must clarify.
- The product name has to be commodity-feeling internally and
  invisible externally. "Red Hat AI Service Platform for Neoclouds"
  works internally; the customer never sees it.

## Notes

- Some Tier-1 neoclouds may negotiate for prominent co-branding ("Powered
  by Red Hat AI") in exchange for marketing commitments. Acceptable
  case-by-case.
- Sovereign / national-champion neoclouds may want Red Hat *more*
  visible for trust reasons ("certified by Red Hat"). Also acceptable
  case-by-case with the same whitelabel underneath.
