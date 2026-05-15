# Capability Map

This is the master list of capabilities Neo Stack provides, the boundary
between what we ship and what we expect from the neocloud's existing
environment, and what we explicitly do not provide.

> **Reminder**: many capabilities below are **delivered by upstream
> components** that Red Hat AI already ships (Kuadrant, Authorino,
> Limitador, maas-api, maas-controller, KServe, llm-d, vLLM). Neo
> Stack's contribution for those rows is **integration, configuration,
> productization, and support** — not greenfield engineering. See
> [`stack.md`](./stack.md) for the explicit delta.

The capabilities below are grouped into seven domains. Each capability is
tagged with the **release** it lands in (`MVP`, `GA`, `v2`, `v3`) and whether
it is part of the **MaaS** line, the **Dedicated Endpoints (DE)** line, or
both.

## 1. Identity & Tenancy

| Capability | Release | Lines |
|---|---|---|
| Org → Project → User hierarchy | MVP | both |
| Email/password and OIDC/SAML SSO | MVP | both |
| SCIM user provisioning | GA | both |
| Service accounts | MVP | both |
| API key issuance, rotation, scoping (project, model, endpoint) | MVP | both |
| Personal access tokens for console | MVP | both |
| RBAC (owner, admin, developer, billing, read-only) | MVP | both |
| Audit log of identity events | MVP | both |
| Per-org branding (logo, colors, custom domain) | GA | both |
| Multi-region tenancy (data residency tagging) | v2 | both |

## 2. API Surface

| Capability | Release | Lines |
|---|---|---|
| `POST /v1/chat/completions` (streaming + non-streaming) | MVP | MaaS, DE |
| `POST /v1/completions` (legacy) | MVP | MaaS, DE |
| `POST /v1/embeddings` | MVP | MaaS, DE |
| `GET  /v1/models` | MVP | both |
| `POST /v1/responses` (OpenAI Responses API parity) | GA | both |
| `POST /v1/moderations` | GA | both |
| Tool / function calling | MVP | both |
| Structured outputs (JSON schema) | MVP | both |
| Vision / multimodal inputs | GA | both |
| Reranking endpoint | GA | both |
| Speech-to-text / text-to-speech | v2 | both |
| `POST /v1/batches` (async batch inference) | GA | MaaS |
| `POST /v1/files` for batch inputs | GA | MaaS |
| Admin API (tenants, keys, endpoints, usage) | MVP | both |
| Metering / usage export API | MVP | both |

API surface details live in [`api/openai-compatible.md`](./api/openai-compatible.md)
and [`api/admin-api.md`](./api/admin-api.md).

## 3. Model Gateway / Routing

| Capability | Release | Lines |
|---|---|---|
| Per-key authentication | MVP | both |
| Per-key, per-project, per-org rate limits (RPM / TPM) | MVP | both |
| Per-key, per-project spend caps | MVP | both |
| Burst smoothing (token bucket) | MVP | both |
| Per-model, per-tenant quotas | MVP | both |
| Tier-based defaults (free, dev, pro, enterprise) | MVP | both |
| Predicted-latency-aware routing into llm-d (IGW) | MVP | both |
| KV-cache-aware routing | MVP | both |
| Prefix cache reuse across tenants (opt-in) | GA | MaaS |
| Per-tenant model preference / sticky routing | GA | DE |
| Smart retries with circuit breaker | MVP | both |
| Request fingerprinting & dedupe | GA | both |
| Prompt and response logging (opt-in per tenant, with redaction) | GA | both |
| Header passthrough (`x-request-id`, etc.) | MVP | both |
| Idempotency keys | GA | both |

## 4. Inference Data Plane

| Capability | Release | Lines |
|---|---|---|
| Curated catalog: 8–10 models at MVP, 25+ at GA | MVP | both |
| Versioned model SKUs (model + quant + accelerator) | MVP | both |
| Per-model perf and cost benchmarks | MVP | both |
| Heterogeneous accelerator pools (NVIDIA, AMD, Gaudi) | GA | both |
| vLLM tuning profiles per model SKU | MVP | both |
| llm-d P/D disaggregation enabled by default | MVP | both |
| KV cache pooling | MVP | both |
| Speculative decoding where supported | GA | both |
| LoRA hot-swap per request | MVP | DE |
| Per-endpoint replica autoscaling | MVP | DE |
| Scale-to-zero for Dedicated Endpoints | MVP | DE |
| Multi-replica MaaS pools with overflow into DE pool | GA | MaaS |
| BYO model with signed-artifact gating | v2 | DE |
| BYO fine-tune (LoRA upload) | GA | DE |
| Fine-tuning service (LoRA SFT) | v2 | DE |
| Distillation / quantization service | v3 | DE |

## 5. Metering, Billing, Cost

| Capability | Release | Lines |
|---|---|---|
| Per-request token accounting (input, output, cached) | MVP | both |
| Per-endpoint GPU-second accounting | MVP | DE |
| Idempotent usage events with cursor-based export | MVP | both |
| Late-arriving usage handling (24h grace) | MVP | both |
| Stripe connector | MVP | both |
| Metronome connector | GA | both |
| Orb connector | GA | both |
| Generic CSV / S3 export | MVP | both |
| Prepaid credits | GA | both |
| Postpaid invoicing | MVP | both |
| Per-tenant configurable pricing | MVP | both |
| Per-model price overrides (promo, enterprise) | MVP | both |
| Spend alerts (50/80/100% of cap) | MVP | both |
| Cost dashboards (developer-facing) | MVP | both |
| Revenue dashboards (operator-facing) | MVP | both |
| FinOps export for the neocloud's own accounting | GA | both |

## 6. Consoles

| Capability | Release | Lines |
|---|---|---|
| Developer console: sign-up, login, project switcher | MVP | both |
| Developer console: API keys page | MVP | both |
| Developer console: usage dashboard | MVP | both |
| Developer console: model catalog with pricing | MVP | both |
| Developer console: playground | MVP | both |
| Developer console: billing portal | MVP | both |
| Developer console: dedicated endpoint provisioning UI | MVP | DE |
| Developer console: logs explorer (opt-in) | GA | both |
| Operator console: tenants list, search, drill-in | MVP | both |
| Operator console: capacity & utilization | MVP | both |
| Operator console: model catalog management | MVP | both |
| Operator console: incidents and on-call view | MVP | both |
| Operator console: revenue / margin view | MVP | both |
| Operator console: tenant impersonation (audited) | GA | both |
| Whitelabel theming end-to-end | GA | both |

## 7. Operations, Security, Compliance

| Capability | Release | Lines |
|---|---|---|
| GitOps install (Argo CD / Flux) | MVP | both |
| OpenShift and vanilla Kubernetes parity | MVP | both |
| Helm chart + Operator for lifecycle | MVP | both |
| Tenant isolation: namespace + NetworkPolicy + PSS | MVP | both |
| Optional GPU-level tenant isolation (no shared GPU) | GA | DE |
| Per-tenant KMS-backed encryption | GA | both |
| TLS termination at gateway with SNI-aware routing | MVP | both |
| Prometheus / OpenTelemetry metrics | MVP | both |
| Datadog, Grafana, Dynatrace integrations | MVP | both |
| Centralized audit log (append-only) | MVP | both |
| SOC 2 control mapping & evidence collection helpers | GA | both |
| HIPAA reference deployment | v2 | both |
| FedRAMP-ready packaging | v3 | both |
| Backup / restore for control-plane state | MVP | both |
| Multi-region active/active control plane | v2 | both |
| Disaster recovery runbooks | MVP | both |

## What the neocloud provides

Neo Stack assumes the neocloud already operates the layers below. We
integrate; we do not re-implement.

- **Compute**: Kubernetes (OpenShift recommended), GPU node pools, GPU
  operator(s) for the relevant accelerator(s).
- **Storage**: object storage (S3-compatible) for model artifacts, logs,
  and batch inputs/outputs.
- **Networking**: VPC, load balancers, DDoS protection, WAF if desired.
- **Identity**: an IdP (Okta, Auth0, Azure AD, Keycloak) we federate into.
- **Billing**: an accounting system to receive our usage export
  (Stripe / Metronome / Orb / NetSuite / SAP).
- **Observability**: a metrics/logs/traces backend.
- **Compliance**: physical / data-center compliance posture. We compose
  on top.

## What Neo Stack explicitly does **not** provide

- **Frontier model training.** Not in scope. Ever.
- **A model marketplace where third parties sell their own models.**
  Possible later; not in scope for MVP or GA.
- **A vector database.** We integrate (pgvector, Milvus, Pinecone) but do
  not ship one.
- **A RAG framework.** Same as above.
- **An agent framework.** We expose tool-use APIs; users bring their own
  framework.
- **A frontend chat UI for end consumers.** This is an API product. The
  consoles are for developers and operators, not end consumers.
