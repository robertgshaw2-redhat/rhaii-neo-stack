# Roadmap

Phases are coarse. Dates are illustrative; staffing and design-partner
validation drive the real schedule.

## Phase 0 — Validation (now)

**Duration**: 4–6 weeks.

- Take this scope to three design-partner neoclouds.
- Validate MaaS pricing assumptions, DE pricing assumptions, and the
  catalog-of-eight choice.
- Land staffing plan and budget.
- Land partnership terms with the `opendatahub-io/models-as-a-service`
  maintainers (or fold it directly into a Red Hat-led release train).

**Exit criteria**: three signed design-partner LOIs, a staffed team, and
sign-off from RH AI leadership on the MVP scope in [`mvp.md`](./mvp.md).

## Phase 1 — Closed Beta / MVP

**Duration**: ~6 months from staffed start. Target: **Q4 2026**.

Deliverables: the [`mvp.md`](./mvp.md) scope. Closed beta with the three
design partners.

**Headline themes**:

- Productize the existing RHAI stack into a deployable bundle.
- Ship the billing / metering plane.
- Ship the Dedicated Endpoints product line.
- Ship the developer + operator consoles.
- Ship Stripe + Okta + Datadog reference integrations.

**Out-of-scope**: BYO model, fine-tuning, batch API, multi-region.

## Phase 2 — GA

**Target**: **Q2 2027**.

**Adds**:

- BYO model intake (signature verification, license scan, health probe).
- Batch / Files API on llm-d batch gateway once upstream goes GA.
- Responses API.
- Moderation endpoint.
- Reranking endpoint.
- Vision / multimodal where catalog supports.
- AMD MI300X and Intel Gaudi3 tuning profiles.
- Metronome + Orb billing connectors.
- Prepaid credits.
- SCIM user provisioning.
- Per-org full whitelabel (custom domain TLS, full theming).
- Per-tenant KMS-backed encryption.
- Tenant impersonation (audited) in operator console.
- Catalog at 25+ models.
- SOC 2 evidence-collection helpers.
- Backup / restore for metering data.
- Air-gapped install (full).

**Success measure**: 8–12 paying neocloud customers in production. >$5M
ARR runrate from Neo Stack.

## Phase 3 — Scale-out (v2)

**Target**: **Q4 2027 → Q2 2028**.

**Adds**:

- Fine-tuning service (LoRA SFT).
- Speech-to-text / text-to-speech endpoints.
- Multi-region active/active control plane.
- Per-GPU tenant pinning at DE.
- VPC peering / PrivateLink for Dedicated Endpoints.
- HIPAA reference deployment.
- FinOps export for the neocloud's accounting.
- Tighter integration with two billing partners (deep co-eng).
- Operator console: capacity planning + forecast.
- Developer console: logs explorer at scale.
- Self-serve "graduate a MaaS tenant to DE" upsell flow.

## Phase 4 — Platform (v3)

**Target**: **2028+**.

**Adds**:

- Agent / Responses-native API surface with stateful tool brokering.
- Cross-cloud / cross-neocloud federated routing (one customer key
  routes across multiple neoclouds based on price/capacity, with the
  selling neocloud as the contracting entity).
- Distillation / quantization service.
- FedRAMP-ready packaging.
- Marketplace mode (third parties publish models into the catalog with
  revenue share).
- Native RAG primitives (optional, controversial; revisit closer to
  date based on what other vendors have absorbed into the model layer
  by then).

## Cross-cutting themes (every phase)

- **Upstream contribution.** Every Neo Stack feature that makes sense
  upstream gets upstreamed first. We are not building a fork.
- **Catalog cadence.** Refresh the curated catalog quarterly.
- **Hardware coverage.** Each phase adds at least one new accelerator
  profile with first-class tuning.
- **Tenancy hardening.** Move steadily toward kernel-level isolation
  options for DE customers who want it.

## Non-goals across the roadmap

- Becoming a hyperscaler. We sell software to neoclouds; we do not run
  the cluster.
- Training foundation models.
- Selling tokens directly from Red Hat. All commerce goes through the
  neocloud's brand.
- Replacing or forking any of: vLLM, llm-d, KServe, Kuadrant,
  Authorino, Limitador.
