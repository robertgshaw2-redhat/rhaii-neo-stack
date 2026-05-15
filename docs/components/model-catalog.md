# Component: Model Catalog

**Source**: Neo Stack.

The catalog is the **product** part of the product. Without a curated,
benchmarked, supported set of models, Neo Stack is a runtime; with one,
it is something a neocloud can sell.

## What a SKU is

A SKU is `(model_family, model_size, quantization, accelerator_profile, version)`.

Examples:

- `llama-3.1-70b-instruct@fp8@h200-1x@2026-04-12`
- `qwen-3-30b-a3b-instruct@bf16@mi300x-1x@2026-04-12`
- `bge-large-en@fp16@h100-1x@2026-04-12`

Each SKU ships with:

- **Tuning profile** — vLLM config (`max_model_len`, `gpu_memory_utilization`,
  `enforce_eager`, attention impl, chunked-prefill on/off, prefix-cache
  on/off, etc.) tuned by Red Hat performance engineering.
- **Benchmark sheet** — throughput, TTFT, TPOT at three reference workload
  shapes (short Q&A, long context, generation-heavy).
- **Reference cost** — $/1M tokens at 60–75% pool utilization, used as
  guidance for the operator price book.
- **Model card** — license, training data summary, known limitations,
  safety considerations.
- **Signed artifact** — sha256, cosign signature.
- **Compatibility matrix** — accelerators, vLLM versions, llm-d versions.

## MVP catalog (illustrative)

The exact list is finalized with design partners. The shape:

- **Chat — small**: a fast, cheap 8B-class model. e.g., Llama-3.1-8B-Instruct.
- **Chat — medium**: a 30–40B-class model. e.g., a Qwen 32B or a Mistral.
- **Chat — large**: a 70B-class model. e.g., Llama-3.1-70B-Instruct.
- **Chat — frontier-open**: a frontier-open MoE. e.g., DeepSeek-V3 or
  Llama-4-class once available.
- **Reasoning**: a reasoning-tuned model. e.g., DeepSeek-R1-Distill-Llama-70B.
- **Embeddings — general**: bge-large-en or e5-large.
- **Embeddings — multilingual**: a multilingual embedding model.
- **Reranker**: bge-reranker-large.

8 SKUs at MVP. Growing to 25+ by GA.

## Catalog lifecycle

```
Proposed ──► Benchmarking ──► Legal sign-off ──► Signed artifact ──► Published
                                                                          │
                                                                          ▼
                                                            Promoted / Hidden / Deprecated
                                                                          │
                                                                          ▼
                                                                  End-of-life
```

States:

- **Proposed**: requested internally or by a design partner; in queue.
- **Benchmarking**: perf-eng has it on the bench.
- **Legal**: license review by Red Hat legal. Some models won't pass.
- **Signed**: artifact in object storage, cosigned, ready to ship.
- **Published**: available to operators to enable for their tenants.
- **Hidden**: operator-disabled SKU; existing references still resolve.
- **Deprecated**: scheduled for end-of-life; new endpoints cannot select.
- **End-of-life**: not selectable, not callable. Customers given 90 days'
  notice before this state.

## Why curation matters more than catalog size

Neoclouds will be tempted to ask for every HF Hub model in the catalog.
Resist this. Curation is the value-add:

- A neocloud cannot stand behind 10,000 models. It can stand behind 25.
- Tuning takes engineering. We cannot tune what we don't ship.
- Legal review takes weeks. We cannot review what we don't ship.
- The list is itself a recommendation — "here are the models you should
  actually use." That's table stakes for a credible MaaS.

BYO model (GA) exists precisely so customers can run the long tail
without it being in the curated catalog.

## Operator workflow

The operator console catalog page lets the operator:

- Browse the full Neo Stack catalog.
- Enable / disable SKUs for their cluster (capacity gates, business
  decision).
- Override per-SKU pricing.
- Override per-SKU rate-limit defaults.
- See per-SKU adoption (how many tenants are using it, how much revenue
  it drives).
- Schedule deprecation announcements to tenants who use a SKU.

## Catalog updates

- Catalog distribution is a Helm subchart shipped by Red Hat, refreshed
  quarterly. Operators pull the new chart and decide what to roll out.
- Critical security updates (e.g., a CVE in a model's tokenizer or a
  vLLM kernel) ship out-of-band.
- New SKUs published mid-quarter are additive, never breaking.

## Cross-accelerator coverage

For each accelerator the catalog ships at least the "small, medium,
large chat + embeddings" baseline. The frontier-open and reasoning
options may be NVIDIA-only at first if the upstream support is not yet
on AMD / Gaudi.

## Out of scope

- A model marketplace where third parties publish into the catalog with
  revenue share. Possible v3.
- Hosted closed-weight models (Claude, GPT-4-class, Gemini). We do not
  have the licensing. If a neocloud has it, they can publish privately
  via the operator console — but we do not ship them.
- Embedded RAG / agent-bound model variants. Out of scope; users
  compose at the application layer.
