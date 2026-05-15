# MVP Scope

> **Goal**: get a first design-partner neocloud into closed beta with paying
> internal-team customers in **two quarters**, and a second / third design
> partner shortly after. MVP is **not GA**. It is the smallest credible
> product surface that lets a neocloud operate both lines of business with
> a small set of customers and a manageable on-call load.

## What MVP includes

### Two product lines, end-to-end

- **MaaS**: 8–10 curated models served via OpenAI-compatible chat,
  completions, embeddings. Streaming + non-streaming. Tool calling.
  Structured outputs.
- **Dedicated Endpoints**: provision a single-tenant endpoint from the
  same catalog, on a chosen GPU SKU, with autoscaling and scale-to-zero.
  LoRA hot-swap supported. BYO model is **not** in MVP.

### Identity & tenancy

- Org → Project → User hierarchy.
- Email/password and one OIDC IdP (Keycloak as default; Okta as
  reference partner integration).
- API keys per project, scoped to model and endpoint sets.
- RBAC with five built-in roles.
- Audit log.

### Gateway / control (upstream MaaS, configured by Neo Stack)

- Kuadrant + Authorino + Limitador in supported configuration.
- maas-api / maas-controller for keys and tenant lifecycle.
- Rate limit tiers: Free, Developer, Pro, Enterprise (operator-editable).
- Spend caps with Neo Stack's controller writing Limitador rules.
- Bearer-token auth, header stripping.

### Inference (upstream stack)

- KServe + llm-d, predicted-latency scheduling, P/D disaggregation,
  KV-cache pooling.
- vLLM workers with Neo Stack tuning profiles per model SKU on at
  least **NVIDIA H100/H200**. AMD MI300X and Intel Gaudi3 are GA targets.
- Per-replica autoscaling, scale-to-zero for DE.

### Billing & metering (Neo Stack)

- Token + GPU-second event ingest from gateway and KServe.
- Hot store (Postgres) + cold (ClickHouse).
- Stripe connector (postpaid).
- Generic CSV / S3 export.
- Idempotency, 24h late-arrival window.
- Per-tenant price book with overrides.
- Spend caps with alerting at 50/80/100%.

### Catalog

- Curated catalog of 8–10 models with shipped tuning profiles, benchmark
  sheets, and model cards. Quarterly refresh.

### Developer console

- Sign-up / login / project switch.
- API keys, scoping, rotation.
- Usage dashboard (tokens, latency p50/p95, spend).
- Catalog browser with pricing.
- Playground for chat / completions / embeddings.
- Dedicated Endpoint create / list / edit / delete UI.
- Billing portal (Stripe).
- Branding hooks (logo, primary color, custom domain).

### Operator console

- Tenants list + drill-in.
- Capacity / utilization per pool and per accelerator.
- Catalog management (publish / hide / promote SKU).
- Per-SKU SLO dashboards.
- Revenue dashboard.
- Tenant kill-switch.
- Audit log viewer.

### Productization

- Helm chart + Operator for full install.
- Argo CD reference application.
- Supported on OpenShift 4.19+ and vanilla Kubernetes 1.30+.
- Documented Day-0 install, Day-1 config, Day-2 upgrade.
- Air-gapped install profile (partial — full air-gap GA target).
- Backup / restore for control-plane state.

### Observability

- Prom and OTel metrics from every Neo Stack service.
- Pre-built Grafana dashboards.
- Datadog integration (metrics + logs).
- Status-page generator wired to per-SKU SLOs.

## What MVP excludes (and why)

| Excluded | Why |
|---|---|
| BYO model upload | Verification pipeline + license scanning add 1+ quarter; ship at GA. |
| Fine-tuning service | Different infra, different SRE story; v2. |
| Batch / async inference API | Upstream llm-d batch gateway still experimental in May 2026 (v0.7); ride upstream to GA. |
| Vision / multimodal | Catalog scope discipline. GA target. |
| Speech endpoints | Different runtime + different latency contract; v2. |
| Responses API | OpenAI-compatible Chat is enough for MVP; Responses is GA. |
| Multi-region active/active | Active/passive with manual failover for MVP; v2. |
| Per-GPU tenant pinning | GA. Until then DE replicas are per-pod isolated. |
| HIPAA / FedRAMP-ready packaging | Not realistic in MVP timeframe. |
| Metronome / Orb connectors | Stripe is enough for the first three design partners. |
| Whitelabel custom domain at full TLS termination | Stretch goal. Subdomain delegation OK for MVP. |

## Success criteria

A design-partner neocloud can:

- Stand up Neo Stack on their cluster in **under one engineering-week**
  with our SE assist.
- Onboard their first three internal-customer accounts inside two weeks.
- Process a measurable volume of MaaS tokens **and** at least one
  Dedicated Endpoint per design partner, with invoices generated through
  Stripe.
- Run the system on a 1-deep on-call rotation without paging Red Hat
  more than once per week on average across the closed-beta period.
- Demo the developer console to their own customers without seeing the
  Red Hat or Neo Stack name anywhere outside the support contract.

If those five hold, MVP is done.

## Out-of-band dependencies

- **llm-d v0.7+** in production. Already shipped (May 2026).
- **`opendatahub-io/models-as-a-service`** at a release we declare
  supportable. Today the upstream is explicit it is pre-production; we
  need a coordinated "first supportable release" with that team.
- **OpenShift AI** carrying compatible versions of KServe and the GPU
  operators.
- **Curated catalog**: legal sign-off on the 8–10 model licenses, and
  benchmarking capacity to publish per-SKU sheets.
