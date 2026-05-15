# Component: Developer Console

**Source**: Neo Stack.

The developer console is what the **neocloud's end customers** see. It
is not a Red Hat brand surface. Every page must be themable and
content-overridable so the neocloud's brand sits cleanly on top.

## Pages (MVP)

### Sign-up & onboarding

- Email/password or SSO (Google, GitHub at neocloud's discretion).
- Email verification.
- Org creation, default project.
- Guided first-key issuance.
- "First request in 60 seconds" embedded curl + JS + Python.

### Project switcher / settings

- Project list, switch.
- Project members, roles.
- Project settings (default model, allowed models, default region).
- Delete project.

### API keys

- List with last-used, scopes, created-by, expiry.
- Create with scoping (project, model, endpoint, expiry).
- Rotate / revoke.
- Show-once-on-creation pattern with copy-to-clipboard.

### Usage

- Tokens, requests, spend over time.
- Per-model breakdown.
- Per-key breakdown.
- Latency histograms (p50, p95, p99) per model.
- Error breakdown by status code.
- Date-range picker; export CSV.

### Catalog

- Browse models grouped by class.
- Per-model: pricing, latency targets, license, model card, recommended
  use cases.
- "Try it" → playground prefilled with the model.

### Playground

- Chat-style interface, model switcher, system prompt, tool-call
  composer.
- Toggle: streaming on/off, temperature, top-p, max tokens, JSON mode.
- Side-by-side compare across models.
- "Copy as curl / JS / Python" with the user's key (redacted by default).

### Dedicated Endpoints

- List of endpoints, status, replicas, recent requests.
- Create new endpoint: model picker → SKU picker → capacity settings →
  scale policy → review → create.
- Per-endpoint detail: dashboards, settings, adapters, deletion.
- LoRA upload + management.

### Billing

- Current invoice estimate, last paid invoice, payment method on file.
- Spend caps (per project, per key) with edit.
- Spend alerts.
- Prepaid credit balance (GA).
- Invoice history.

### Logs (GA, opt-in)

- Per-request log with input/output (redacted by default).
- Filter by date range, key, model, status.
- Search.

### Account & profile

- Profile, security settings, 2FA.
- Audit log of own actions.
- Notifications.

## Pages (Operator, beta-only)

Operator console is a separate UI; see
[`operator-console.md`](./operator-console.md).

## Branding

The console is shipped as a Next.js app with:

- A **theme bundle** loaded at runtime: logo, favicons, color tokens,
  typography overrides, optional CSS.
- A **content bundle**: per-org overrides for marketing copy, ToS,
  Privacy, help URLs.
- A **domain map**: `app.<neocloud>.com` → tenant Foo at theme Foo.
- A **custom HTML head injection** slot for analytics / pixels.

The console can be configured to **never** mention Red Hat or Neo
Stack. Default is to credit subtly in the footer; the neocloud can
remove the credit (per their license tier; Scale and Enterprise tiers
include full whitelabel).

## Component philosophy

- Built on shadcn-style primitives. Themable via design tokens.
- WCAG AA at MVP, AAA at GA where feasible.
- Internationalization-ready (i18n keys everywhere; we ship en-US and
  let the neocloud add locales).
- No analytics phoning home to Red Hat by default. Operators opt in
  to anonymous telemetry.

## API surface used

The console only talks to the **Tenant API** (control plane). It does
not bypass the gateway to call the inference plane directly; the
playground calls the same OpenAI-compatible API the customer's code
calls.

## Performance budget

- TTI < 2s on warm session, < 4s cold.
- Usage dashboards render against the last 30 days in < 1s for typical
  tenants (< 10M requests/period).
- The console is fully static + API; no SSR is required for tenant
  pages.

## Out of scope (MVP)

- Mobile app. Mobile-responsive web only.
- Chat-with-your-data / RAG UI.
- Notebook integration.
- Marketplace browsing (no marketplace yet).
- Per-user notification routing (single email per tenant for MVP).
