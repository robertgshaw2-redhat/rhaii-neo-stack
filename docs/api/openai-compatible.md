# OpenAI-Compatible API

The end-customer inference API is **OpenAI-compatible** by design. The
goal is drop-in replacement of `api.openai.com` for the supported
verbs. If a customer's SDK works against OpenAI, it works against the
neocloud's Neo Stack deployment.

## Scope at each release

### MVP

- `POST /v1/chat/completions` (streaming + non-streaming)
- `POST /v1/completions` (legacy)
- `POST /v1/embeddings`
- `GET  /v1/models`
- Tool calling / function calling
- Structured outputs (JSON schema)
- Standard request/response shape, including `usage` block

### GA

- `POST /v1/responses` (parity with OpenAI Responses API)
- `POST /v1/moderations`
- `POST /v1/batches` + `POST /v1/files` (async batch)
- Reranking endpoint (vendor-neutral path; OpenAI-style not standardized)
- Vision / multimodal inputs (image inputs in chat)

### v2

- Speech-to-text (`POST /v1/audio/transcriptions`)
- Text-to-speech (`POST /v1/audio/speech`)
- Realtime audio API (where applicable)

## Authentication

```
Authorization: Bearer sk-...
```

API keys are issued through the developer console or maas-api. The
gateway strips this header before forwarding upstream.

## Headers we honor and emit

Inbound (honored if sent by the client):

- `X-Request-Id` — passed through; we generate one if absent.
- `OpenAI-Beta` — accepted for parity; specific betas honored per release notes.
- `Idempotency-Key` — GA; deduplicates retries.
- `X-Stainless-*` and other OpenAI SDK telemetry headers — accepted,
  not acted on, never logged.

Outbound (we add):

- `X-Request-Id`
- `X-Model-Sku` — exact SKU served (helpful for debugging).
- `X-Pool` — which pool served (debug; suppressed in production
  responses unless tenant has opted into debug mode).
- `X-RateLimit-Remaining-Requests` / `-Tokens` (OpenAI parity).
- `X-RateLimit-Reset-Requests` / `-Tokens`.

## Differences from OpenAI

We document these explicitly because customers will notice:

| Difference | Why |
|---|---|
| Model names | Use the neocloud's catalog model IDs, not `gpt-*`. Mapping shims are out of scope. |
| `usage.prompt_tokens_details.cached_tokens` | We populate this from llm-d KV cache hits. Semantics may differ slightly from OpenAI's. |
| `system_fingerprint` | We set to `<sku>@<vllm_version>@<llmd_version>` so customers can pin reproducibility. |
| Tool-call streaming order | Matches OpenAI's; vLLM differences from OpenAI are smoothed at the gateway. |
| Rate-limit error shape | Matches OpenAI 429 with `error.type: "rate_limit_exceeded"`. |
| Free-tier daily reset | Tier policy chosen by the neocloud; documented per tier. |

## Model addressing

Customers send a `model` field. We accept:

- A **catalog SKU ID** (`llama-3.1-70b-instruct`) — the catalog-current
  version for that SKU.
- A **versioned SKU** (`llama-3.1-70b-instruct@2026-04-12`) — pinned.
- A **base@adapter** form for LoRA on Dedicated Endpoints
  (`base-sku@adapter-id`).
- An **endpoint alias** for Dedicated Endpoints (`@endpoint:my-endpoint`).

Unknown / disabled / unsubscribed models return `404` with an error
type the customer can disambiguate.

## Streaming

Server-Sent Events, OpenAI-compatible:

- `data: {...}\n\n` per chunk.
- `data: [DONE]\n\n` terminator.
- Heartbeats every 10s for long-running responses (configurable).
- Cancellation: closing the connection cancels the upstream request
  (best-effort; some KV state may already be committed).

## Errors

We use OpenAI's error envelope:

```json
{
  "error": {
    "message": "Rate limit exceeded for project proj_… on model …",
    "type": "rate_limit_exceeded",
    "param": null,
    "code": null
  }
}
```

We add a `request_id` field at the top level for support correlation:

```json
{
  "error": { ... },
  "request_id": "req_…"
}
```

Status codes are OpenAI-equivalent: 400, 401, 403, 404, 409, 413, 422,
429, 500, 502, 503, 504.

## Stability guarantees

- **Major versions in the URL path** (`/v1`). We keep `/v1` stable; new
  surfaces under `/v2` if needed.
- **Backward-compatible additions** are minor-release events.
- **Breaking changes** require a 6-month deprecation window with
  parallel paths.

## What we will not absorb

- OpenAI-specific endpoints that don't apply (e.g., `/v1/assistants`,
  `/v1/threads`, `/v1/runs`). The Assistants API has been superseded by
  Responses; we follow Responses.
- OpenAI's organization-billing endpoints (we have our own billing).
- Image-generation endpoints (no diffusion in catalog at MVP).
