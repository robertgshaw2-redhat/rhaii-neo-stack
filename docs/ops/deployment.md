# Deployment

## Install shape

Neo Stack ships as a **bundle** of:

- Upstream operators / Helm charts (Kuadrant, Authorino, Limitador,
  KServe, llm-d, the maas-* components).
- Neo Stack Helm charts (control-plane services, consoles, metering,
  billing).
- An Argo CD `ApplicationSet` that wires it all together.
- A `neostackctl` CLI for install / upgrade / status / migrate /
  diagnose.

The supported install shapes are:

| Shape | Supported substrate | Notes |
|---|---|---|
| OpenShift AI bundle | OpenShift 4.19+ with OpenShift AI | Preferred. Uses the OpenShift Operator Hub. |
| Vanilla Kubernetes bundle | K8s 1.30+ | Gateway API + cert-manager required. |
| Air-gapped | Either | Mirror registry + offline manifests. Full air-gap at GA. |

## Day-0 install (sketch)

```
$ neostackctl install \
    --shape openshift \
    --domain api.acme.cloud \
    --idp keycloak://idp.acme.cloud \
    --billing stripe://… \
    --catalog quarterly:2026Q2
```

The CLI:

1. Verifies cluster prerequisites (versions, CRDs, GPU operator,
   storage class).
2. Generates a values bundle.
3. Pushes the bundle to a Git repo (or a local manifest tree) for
   Argo CD to reconcile.
4. Watches Argo CD sync; reports until all components are Ready.
5. Runs post-install smoke tests (a synthetic key, a synthetic chat
   call to a tiny catalog model, an invoice generation dry-run).

## Day-1 configuration

After install, the operator console is the primary surface. Day-1
tasks include:

- Connect the IdP fully (claims mapping).
- Configure Stripe (or Metronome / Orb) keys and webhook endpoint.
- Enable initial catalog SKUs.
- Set price book overrides.
- Configure rate-limit tiers.
- Configure branding.
- Connect observability backend.

## Day-2 upgrades

- Upgrades are GitOps-driven. Bump the Argo CD application's target
  revision, Argo CD reconciles.
- `neostackctl upgrade --to <version>` writes the bump, optionally
  drains, watches sync.
- Pre-flight checks: cluster version compat, CRD migration plan, DB
  migration plan.
- Rollback path: revert the Argo CD target, re-run any DB-down
  migrations from the previous release.

We commit to **N-1 supportability**: any release in supported windows
can be reached from the previous release. We do not require multi-hop
upgrades within a major.

## Air-gapped profile

GA target. Components:

- Mirror registry with all images.
- Offline catalog Helm subchart (model artifacts pre-staged in the
  customer's object storage).
- License files baked in (no phone-home for entitlement validation).
- `neostackctl install --air-gapped --bundle <tarball>` does it all
  from disk.

## Multi-region (v2)

Out of scope for MVP. The MVP shape is single-region with manual DR.
See [`multi-region.md`](./multi-region.md).

## Sizing

See [`sizing-capacity.md`](./sizing-capacity.md).

## What `neostackctl` does **not** do

- It does not install Kubernetes. The neocloud owns that.
- It does not install the GPU operator. The neocloud owns that.
- It does not provision external dependencies (Stripe accounts, IdP
  tenants). It connects to them.
