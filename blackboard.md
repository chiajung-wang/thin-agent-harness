# Blackboard

## Intent

Design an agentic system framed as a startup company structure — flat, low-bureaucracy — where a founder orchestrator delegates work to IC agents and synthesizes results back into a coherent deliverable that respects user intent.

## Spec

### Topology (v1, frozen)

- **Topology**: one root orchestrator (Founder) + IC agents (Builder, Designer, Researcher, Marketer, Critic). No middle managers.
- **Memory**: single shared `blackboard.md` with Intent · Spec · Decisions · Open questions · Positioning.
- **Contracts**: loose JSON between Founder and ICs (`task`, `context`, `output_expected`, `constraints` in; `deliverable`, `assumptions`, `risks`, `open_questions` out).
- **Concurrency**: fan-out parallel when IC outputs are independent; serial when dependent.
- **Conflict resolution**: one mediation pass by Founder, then escalate to user.
- **Reporting**: at every stage boundary, Founder invokes the `stage-report` skill, which writes synced HTML + JSON + chat-Markdown artifacts using a fixed four-layer template.
- **Human-in-loop**: reports build the human's mental model and muscle memory; same shape every stage.

### Active spec slice: landing page (dry-run intent B)

- **Goal**: 1-page HTML landing for `google-agent-skill` repo that exercises the founder loop end-to-end (dry-run validation).
- **Audience**: hiring / portfolio surface — recruiters + engineers evaluating the author. Tone: polished, emphasize design thinking + tradeoffs.
- **Primary CTA**: Star on GitHub. Secondary: read the topology doc.
- **Hosting**: `docs/index.html` + GitHub Pages (needs Pages enabled in repo settings; out of scope for this dry-run, doc the step).
- **Content blocks (proposed, designer to finalize)**:
  1. Hero — one-line value prop + star CTA.
  2. Topology diagram — reuse / restyle the Mermaid graph from `reports/2026-05-15-1645-ic-roster-complete.html`.
  3. Mental model — 3–5 design-thinking bullets (write asymmetry, severity ladder, verification ladder, stage-report cadence, growth triggers).
  4. Tradeoffs — what was deliberately NOT built (no DAG, no middle managers, no per-IC blackboard writes) and why.
  5. Tech footer — built with Claude Code + addy-agent-skills + Mermaid; link to repo, CLAUDE.md, topology doc.
- **Non-goals**: no signup, no analytics, no backend, no JS framework. Single static HTML + inlined or CDN CSS.
- **Acceptance**: page renders locally in browser; Critic verdict `go` or `conditional`; both stage reports (spec-frozen + ship-gate) on disk.

## Decisions

- 2026-05-15 [decision] Use the startup (flat) variant of the company-as-harness framing rather than the corporate (multi-layer) variant. Optimizes for 0→1 speed; will add layers only on observed pain.
- 2026-05-15 [decision] Add `Marketer` IC alongside Builder, Designer, Researcher, Critic. Owns positioning, copy, launch comms, channel pick.
- 2026-05-15 [decision] Implement stage reporting as a **project-level skill** (`stage-report`), not as a founder-owned skill or a dedicated agent. Reasons: format consistency for human muscle memory, reuse across orchestrator + critic + future reporter-agent fallback.
- 2026-05-15 [decision] Stage report emits three artifacts in sync from one render pass: chat Markdown (narrative), HTML (human, Mermaid diagram via CDN, no build step), JSON (agent, structured twin). All three share a fixed four-layer template: TL;DR · Mental model · State delta · Open questions.
- 2026-05-15 [decision] Skill is versioned (`v1`). Future shape changes ship as sibling skills, not in-place mutations, to preserve trained human reading pattern.
- 2026-05-15 [decision] Full IC roster landed in commit `946c8c7`: builder, researcher, designer, marketer, critic. Same template (frontmatter + responsibilities + JSON in/out + operating rules + skill-reference table + anti-patterns + escalation + blackboard rules).
- 2026-05-15 [decision] Every IC + founder wired to `addy-agent-skills` plugin via a Skill references table mapping triggers to specific skills. Return JSON gains `skills_invoked` field for founder audit.
- 2026-05-15 [decision] Marketer is the only IC permitted to append to the blackboard (`## Positioning` section). All other ICs are read-only; founder owns writes to `## Decisions` and `## Spec`.
- 2026-05-15 [decision] Verification ladder for Builder fixed: typecheck → lint → targeted test → smoke run. Stop at first real signal.
- 2026-05-15 [decision] Critic is a one-pass ship gate, not per-deliverable review. Returns severity-tagged findings (`blocker|major|minor|nit`) with verdict `go|no-go|conditional`; never writes fixes.
- 2026-05-18 [decision] Dry-run intent picked: landing page for `google-agent-skill` repo (intent B from menu). Exercises all 5 ICs, both stage-report boundaries, and the only-Marketer blackboard-write path. Audience = hiring/portfolio; CTA = star on GitHub; host = `docs/index.html` + GitHub Pages.

## Open questions

1. Where the runtime `blackboard.md` lives when the system is deployed against a real project — default if silent: project root, same level as `.claude/`.
2. Whether to implement the fallback `reporter` agent now or defer until context pressure is observed — default if silent: defer.
3. Whether to add automatic growth-trigger detection (e.g., context saturation warning) — default if silent: defer, rely on manual founder judgement.
4. Whether to dry-run the topology against a sample project intent to validate the founder loop end-to-end before declaring v1 done — default if silent: dry-run on a small bounded task next.
5. Whether to add `CLAUDE.md` at project root to document how to use this topology — default if silent: write it once dry-run validates the loop.

## Positioning

Not yet authored. Marketer IC will populate when product framing stabilizes beyond architecture-only stage.
