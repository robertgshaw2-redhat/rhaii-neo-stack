# ADR-0002: OpenAI-compatible API as the end-customer surface

## Status

Accepted.

## Context

The end-customer API surface for both MaaS and Dedicated Endpoints can
take one of several shapes:

1. OpenAI-compatible (chat/completions/embeddings/etc).
2. A neutral, model-agnostic shape we design (e.g., the
   serving-protocol-neutral one llm-d exposes internally).
3. Each neocloud's own shape, with us as a router.
4. Multiple shapes simultaneously.

OpenAI's API surface is the de-facto standard for inference. Every
major SDK targets it; every popular framework speaks it. The Responses
API extends it in a direction the rest of the industry is following.

Anyscale, Together, Fireworks, Groq, Bedrock, Vertex, Azure AI Foundry,
and Cerebras all expose OpenAI-compatible surfaces for their
non-proprietary models. Customers expect it.

## Decision

The end-customer inference API surface is **OpenAI-compatible**. We
target current OpenAI semantics and version with their public surface.
We add the Responses API at GA. We do not implement OpenAI-specific
endpoints that don't apply (Assistants, image generation).

We do **not** expose a competing neutral shape. The Admin API has its
own shape (per [`api/admin-api.md`](../api/admin-api.md)); that's
different — it's not inference.

## Consequences

- Lowest possible adoption friction. Drop-in for OpenAI customers.
- We are downstream of OpenAI's API decisions. We don't get to design
  the verbs. When OpenAI ships a new mechanic, we have to decide
  whether to follow.
- We have a stable target — the OpenAI public API is well-documented
  and slow-moving in its core verbs.
- Where vLLM behavior differs from OpenAI's, we either patch the
  difference at the gateway (preferred) or document it explicitly.
- Customers who want a neutral shape can call vLLM directly through
  their endpoint URL — vLLM has its own OpenAI-compatible surface and
  some extensions.

## Notes

- We track the OpenAI public surface quarterly and decide which new
  endpoints / parameters / behaviors to absorb.
- Customers can pin `system_fingerprint` for reproducibility (we
  populate it from SKU + vLLM/llm-d versions).
