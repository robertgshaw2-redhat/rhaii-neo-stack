# Security & Compliance

## Threat model

The product faces a multi-tenant SaaS-style threat model with the
following actors:

| Actor | Concern |
|---|---|
| Neocloud's end customer (legitimate) | Sees only their own data. |
| Neocloud's end customer (adversarial) | Cannot escalate to other tenants or to the cluster. |
| Neocloud operator (legitimate) | Can support customers without seeing PII unnecessarily. |
| Neocloud operator (adversarial / insider) | Bounded by audit, RBAC, least-privilege. |
| Cluster compromise (worst case) | Per-tenant blast radius limited; auditable; key rotation possible. |
| Network attacker | TLS everywhere; no in-cluster plaintext outside loopback. |

## Tenancy boundaries

| Boundary | Mechanism | Strength |
|---|---|---|
| API key → project → org | Authorino + maas-api | Strong; battle-tested. |
| Rate limit | Limitador | Strong; per-key, per-project, per-org. |
| Inference pool (MaaS) | Shared by SKU | Soft; KV cache partition + scheduler. |
| Inference pool (DE) | Per-endpoint, per-tenant | Strong; namespace + NetworkPolicy. |
| Compute (kernel) | Namespace + PSS | Standard K8s. |
| Compute (GPU) | Per-pod by default; per-GPU pin at premium | Operator policy. |
| Storage | Per-tenant prefixes; KMS keys at GA | Operator-configurable. |
| Logs / metrics | tenant_id partitioning | Service-enforced. |

## Authentication paths

- **End customer API**: bearer API key (`sk-…`) issued by maas-api.
- **End customer console**: SSO via configured IdP (Keycloak/Okta/Auth0/
  Azure AD), session cookies, optional 2FA.
- **Operator console**: SSO via neocloud's staff IdP, mandatory 2FA, IP
  allowlist optional.
- **Admin API**: OIDC bearer tokens.
- **Service-to-service**: SPIFFE / mTLS within the cluster (GA target),
  Kubernetes ServiceAccount tokens for MVP.

## Secrets

- All secrets in Kubernetes Secrets, ideally backed by the neocloud's
  external secret manager (Vault, AWS Secrets Manager, GCP Secret
  Manager, Azure Key Vault) via External Secrets Operator.
- No secrets in Helm values committed to Git. The CLI generates
  sealed-secret manifests where applicable.

## Key data handling

- **Customer prompts and completions**: in-memory in workers; not
  persisted unless tenant explicitly opts in.
- **Customer API keys**: stored hashed (argon2id) in maas-api; never
  logged plain; surfaced once on creation.
- **PII in identity**: encrypted at rest; per-tenant KMS key (GA).
- **Audit log**: append-only, retention configurable, exportable.

## Header stripping

The Authorization header carrying `sk-…` is stripped at Authorino
**before** the request leaves the gateway. The IGW, vLLM, and any
downstream component never see the customer's credential. Internal
identity is propagated via signed `X-Neostack-*` headers.

## Audit obligations

Every action on the Admin API and every authentication decision writes
an audit record. The audit schema is documented in
[`../components/control-plane.md`](../components/control-plane.md). The
operator can export it to their SIEM.

## Compliance targets

| Standard | MVP | GA | v2 | Notes |
|---|---|---|---|---|
| SOC 2 control mapping | helpers | full | full | Customer owns the audit; we ship the controls. |
| ISO 27001 mapping | — | partial | full | Same. |
| HIPAA reference deployment | — | — | full | Strict isolation profile + BAA template. |
| FedRAMP-ready packaging | — | — | partial | Inheritance from RH FedRAMP-authorized stack. |
| GDPR | from day 1 | from day 1 | from day 1 | DPA template, data-residency tagging, deletion API. |
| PCI | — | — | inherit | We don't process card data ourselves; Stripe / Metronome do. |

We ship **control-mapping documents** for SOC 2 and ISO 27001 starting
GA, including which controls are inherited (OpenShift, the neocloud's
SOC 2), which are shared (gateway, audit), and which are tenant-owned.

## Supply-chain security

- All Neo Stack images signed (cosign).
- SBOMs (CycloneDX) published per release.
- Image base: ubi9-minimal where possible.
- Trivy / Grype scans gating CI.
- Provenance attestations (SLSA L3 target).

## Catalog model security

- Model artifacts signed (cosign) on the catalog side.
- Tokenizer parsers and runtime kernels: vulnerability monitoring with
  upstream tracking; security patches ship out-of-band when needed.
- BYO model intake (GA) runs static analysis on artifacts for known
  malicious patterns; runs in a sandboxed health-probe pod.

## Incident response

- Severity levels and response time SLAs documented.
- Per-tenant breach-notification template.
- Audit-export script for post-incident analysis.
- Quarterly tabletop with design partners.

## What we do not handle for the neocloud

- Their data-center physical security.
- Their hypervisor / kernel-level vulnerability management beyond the
  components we ship.
- Their corporate compliance program.
- The legal relationship with their end customers (DPAs, BAAs, MSAs).
  We provide templates and integration; they sign the contracts.
