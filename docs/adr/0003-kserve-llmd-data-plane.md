# ADR-0003: KServe + llm-d + vLLM as the data plane

## Status

Accepted.

## Context

Several runtimes could underpin Neo Stack:

1. Plain vLLM containers behind a custom scheduler.
2. KServe + vLLM with no distributed orchestration.
3. KServe + llm-d + vLLM (the current Red Hat AI stack).
4. Ray Serve + vLLM (the Anyscale-style approach).
5. NVIDIA Triton + TensorRT-LLM (NVIDIA-stack approach).
6. Custom runtime built in-house.

Red Hat already ships, supports, and contributes to **KServe + llm-d +
vLLM**, with predicted-latency scheduling GA in llm-d v0.7 (May 2026)
and P/D disaggregation and KV-cache-aware routing already in
production at CoreWeave and Azure.

The other options each have tradeoffs but none let us reuse the
investments Red Hat has already made — and the whole premise of Neo
Stack is to monetize those investments.

## Decision

The Neo Stack data plane is:

- **vLLM** (Red Hat AI Inference Server distribution) as the inference
  engine.
- **llm-d** as the distributed-inference orchestrator (KV-cache
  pooling, P/D disaggregation, predicted-latency scheduling, future
  batch gateway).
- **KServe** as the Kubernetes abstraction for `InferenceService`.

Neo Stack does not run a competing scheduler, does not fork vLLM, and
does not maintain a parallel data-plane offering.

## Consequences

- We benefit from upstream improvements in vLLM and llm-d without
  re-doing the work.
- Our perf engineering effort goes into **tuning profiles** per model
  × accelerator SKU, not into rebuilding the engine.
- We share an on-call surface with the vLLM and llm-d teams, which is
  acceptable given the same organization staffs both.
- We are constrained by what the upstream supports. If a customer asks
  for a model the upstream cannot serve well, we say no (or upstream
  the fix).
- AMD MI300X, Intel Gaudi3, and other accelerators land on Neo Stack
  on the upstream's schedule, not ours. We coordinate roadmaps but
  don't fork ahead.

## Notes

- llm-d's batch gateway is experimental in May 2026 (v0.7). We ride
  upstream to GA before exposing it as a customer-facing batch API.
- Per-SKU tuning profiles are a real artifact: a YAML stanza of vLLM
  flags + llm-d hints + recommended replicas per accelerator profile.
