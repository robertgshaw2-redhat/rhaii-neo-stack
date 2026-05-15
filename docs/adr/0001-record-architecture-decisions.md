# ADR-0001: Record architecture decisions

## Status

Accepted.

## Context

Neo Stack is a multi-team, multi-quarter product. Many decisions about
its shape will be made early, lose context fast, and be questioned
months later. We want a lightweight, durable record that explains
*why* — not just *what* — to readers who weren't in the room.

## Decision

We use Architecture Decision Records (ADRs) in this repo, under
`docs/adr/`, numbered, in the format Michael Nygard popularized:

- Title
- Status (Proposed | Accepted | Superseded by ADR-XXXX | Deprecated)
- Context
- Decision
- Consequences

ADRs are immutable once Accepted; superseding decisions get a new ADR
that references the old one. ADRs are short — one page when possible,
two when necessary, never longer.

## Consequences

- Anyone joining the project later can read the ADRs in order to
  reconstruct the design conversation.
- Decision authors can be lazy about restating the context in design
  docs and instead link.
- We will inevitably accumulate ADRs that become inaccurate but
  haven't been Superseded. That's OK — they record what we thought at
  the time.
