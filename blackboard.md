# Blackboard

## Intent

Design an agentic system framed as a startup company structure — flat, low-bureaucracy — where a founder orchestrator delegates work to IC agents and synthesizes results back into a coherent deliverable that respects user intent.

## Spec

- **Topology**: one root orchestrator (Founder) + IC agents (Builder, Designer, Researcher, Marketer, Critic). No middle managers.
- **Memory**: single shared `blackboard.md` with Intent · Spec · Decisions · Open questions · Positioning.
- **Contracts**: loose JSON between Founder and ICs (`task`, `context`, `output_expected`, `constraints` in; `deliverable`, `assumptions`, `risks`, `open_questions` out).
- **Concurrency**: fan-out parallel when IC outputs are independent; serial when dependent.
- **Conflict resolution**: one mediation pass by Founder, then escalate to user.
- **Reporting**: at every stage boundary, Founder invokes the `stage-report` skill, which writes synced HTML + JSON + chat-Markdown artifacts using a fixed four-layer template.
- **Human-in-loop**: reports build the human's mental model and muscle memory; same shape every stage.

## Decisions

- 2026-05-15 [decision] Use the startup (flat) variant of the company-as-harness framing rather than the corporate (multi-layer) variant. Optimizes for 0→1 speed; will add layers only on observed pain.
- 2026-05-15 [decision] Add `Marketer` IC alongside Builder, Designer, Researcher, Critic. Owns positioning, copy, launch comms, channel pick.
- 2026-05-15 [decision] Implement stage reporting as a **project-level skill** (`stage-report`), not as a founder-owned skill or a dedicated agent. Reasons: format consistency for human muscle memory, reuse across orchestrator + critic + future reporter-agent fallback.
- 2026-05-15 [decision] Stage report emits three artifacts in sync from one render pass: chat Markdown (narrative), HTML (human, Mermaid diagram via CDN, no build step), JSON (agent, structured twin). All three share a fixed four-layer template: TL;DR · Mental model · State delta · Open questions.
- 2026-05-15 [decision] Skill is versioned (`v1`). Future shape changes ship as sibling skills, not in-place mutations, to preserve trained human reading pattern.

## Open questions

1. Which IC agent file to write next (Builder, Designer, Researcher, Marketer, Critic) — default if silent: Builder, since it is the highest-traffic IC for a code-producing project.
2. Where the runtime `blackboard.md` lives when the system is deployed against a real project — default if silent: project root, same level as `.claude/`.
3. Whether to implement the fallback `reporter` agent now or defer until context pressure is observed — default if silent: defer.
4. Whether to add automatic growth-trigger detection (e.g., context saturation warning) — default if silent: defer, rely on manual founder judgement.

## Positioning

Not yet authored. Marketer IC will populate when product framing stabilizes beyond architecture-only stage.
