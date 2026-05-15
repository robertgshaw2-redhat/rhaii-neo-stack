# Personas

Neo Stack has two distinct buyer-and-user surfaces. The first is the
**neocloud itself** — the company that buys Neo Stack and runs it. The
second is the **neocloud's customers** — developers and ML teams who never
know Red Hat is in the picture. We design for both.

## Buyer (at the neocloud)

### "Priya" — VP of Product, Tier-2 Neocloud

- Owns the P&L for the tokens / endpoints line of business.
- Has board-level pressure to show non-GPU-hour revenue inside 12 months.
- Will buy Neo Stack if it shaves 12+ months off launch and the per-token
  uplift is defensible against in-house build math.
- **What she needs from us**: a credible ship date, a margin-friendly
  pricing model, public reference customers, and a roadmap she can show
  her board.

### "Marcus" — Head of Platform Engineering, Tier-2 Neocloud

- Owns Kubernetes, networking, storage. Already deploys vLLM. Probably has
  llm-d in pre-prod.
- Skeptical of any platform that sits between his team and the metal.
- Will block a deal if Neo Stack adds operational complexity without
  giving him an exit path.
- **What he needs from us**: GitOps install, plain-K8s and OpenShift
  parity, no proprietary CRDs we don't upstream, working escape hatches,
  runbooks, and a 24×7 phone number.

## Users at the neocloud

### "Sam" — Site Reliability Engineer, Neocloud

- On-call for the inference plane.
- Cares about clear failure modes, useful dashboards, and the ability to
  drain a tenant or a model without paging product.
- **What he needs**: operator console, multi-tenant SLO dashboards, blast
  radius controls, audit logs, alert recipes that come pre-tuned.

### "Lin" — Solutions Engineer, Neocloud

- Front-line to the neocloud's enterprise customers asking about Dedicated
  Endpoints.
- Demos the developer console weekly.
- Routes feature requests back to product.
- **What she needs**: a clean, brandable console; an honest model catalog
  with benchmarks she can show; a way to provision a sandbox endpoint in
  60 seconds for a demo; templates for common workloads.

### "Daniel" — Finance / RevOps, Neocloud

- Books revenue, reconciles usage against invoices, manages credit limits.
- Has to answer auditors and the CFO.
- **What he needs**: an export pipeline he can trust into Stripe /
  Metronome / Orb / NetSuite. Idempotent. Auditable. Late-arriving usage
  handled correctly. No "trust me, the dashboard says so."

## End-customer users (the neocloud's customers)

### "Alex" — Application Developer at a startup

- Building an agent / chatbot / copilot.
- Picks an inference provider based on price, model availability, latency,
  and how fast they can sign up and get a key.
- Will absolutely leave if rate limits are surprising or billing is opaque.
- **What he needs**: a great developer console, OpenAI-compatible API,
  fast key issuance, predictable rate limits, a usage dashboard, and a
  playground.

### "Maya" — Staff ML Engineer at a mid-market AI company

- Deploys models for her company's product.
- Wants a Dedicated Endpoint for a fine-tune, with predictable latency
  and the ability to autoscale.
- Cares about LoRA support, GPU SKU choice, and a real SLO.
- **What she needs**: self-service endpoint provisioning, a YAML or
  Terraform interface, custom-model upload, LoRA hot-swap, deployment
  history, rollback, and real per-endpoint observability.

### "Jordan" — CTO of a regulated mid-market company

- Cannot send data to OpenAI for compliance reasons.
- Wants a single-tenant endpoint in a specific region, with a BAA / DPA
  / sovereignty story.
- Will pay a premium for control and contractual clarity.
- **What he needs**: data-residency guarantees from the neocloud, a clear
  isolation model from us, an audit log we can defend, and a model
  catalog that includes the open models his team has standardized on.

## Non-personas (explicitly out of scope)

- **The frontier-model researcher.** Not our user. Training is not in
  scope; this is an inference product.
- **The consumer marketplace renter.** Different buying motion.
- **The hyperscaler enterprise.** Already buying Bedrock / Vertex /
  Foundry. We may pick them up later via sovereignty plays but they are
  not the design center.
