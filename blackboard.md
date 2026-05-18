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
- 2026-05-18 [decision] T3 fan-out complete (designer ‖ marketer). Synthesis decisions: (a) tradeoff section labels use Marketer's noun forms ("DAG planner", "Sub-orchestrator middle managers", "Per-IC blackboard writes") over Designer's earlier wording; (b) hero subhead capped at original ≤20 words (Marketer's 17 stands), not Designer's self-imposed ≤12; (c) accent color bumped #1A56DB → #1445C0 for ~5.1:1 contrast margin (was 4.56:1, 0.06 above AA floor); (d) primary CTA links to repo root, not `/stargazers` (avoids login wall); (e) repo owner slug = `chiajungwang`; (f) Mermaid CDN-failure fallback text required in TopologySection per Designer risk; (g) voice samples deferred — neutral-technical register accepted unless user overrides at Critic gate.
- 2026-05-18 [decision] Critic ship-gate verdict `conditional`: 1 major + 3 minors + 1 nit. Founder rulings: (1) major (hero subhead word count mismatch) **dismissed as false positive** — page text matches Marketer's `subhead` field byte-for-byte; actual count = 16 (Marketer self-miscounted as 17). (2) Minor SRI hash on Mermaid CDN **deferred** — low risk for portfolio page, logged as known. (3) Minor eyebrow `aria-hidden` **fixed inline by founder** — removed attribute so SR users hear "Open-source" qualifier. (4) Minor skip-nav anchor (WCAG 2.4.1) **fixed inline by founder** — added `<a class="skip-nav">` + CSS + `id="main-content"` on `<main>`. (5) Two nits skipped (redundant `mermaid.initialize`, fallback text placement non-issue). Builder re-dispatch not needed; founder edits were 2-line scope.
- 2026-05-18 [decision] Post-T7 Mermaid render bugs caught by user eyeball after pre-ship report; 3 inline fixes by founder: (a) **fallback-text-in-pre bug** — Designer's "fallback as initial textContent" pattern broke parsing (Mermaid read "Diagram unavailable — view source on GitHub" as graph syntax); moved fallback to sibling `<noscript>`; stripped non-standard `rx:N` from `classDef` lines. (b) **white-on-white edge labels** — Mermaid `base` theme defaulted edge label text to white; added `textColor: "#111111"` to themeVariables. (c) **node label clipping** — global `* { box-sizing: border-box; margin: 0; padding: 0 }` cascaded into Mermaid's hidden HTML text-measurement DOM (created in `document.body`, outside `pre.mermaid` scope); switched flowchart config to `htmlLabels: false` so labels render as SVG `<text>` with intrinsic sizing; then added SVG-targeted `fill` overrides for `.edgeLabel text/tspan` since SVG ignores CSS `color`. Lesson: Critic's render-pass acceptance was Builder-self-reported and not founder-verified; for visual artifacts in future, founder must eyeball the artifact even when Critic returns `go`. Critic's a11y axis caught text-not-visible-only-to-SR; missed text-not-visible-to-anyone — gap in review scope.
- 2026-05-18 [decision] Retire blackboard root open question q4 (dry-run topology end-to-end) — **done** via intent B landing-page exercise; all founder-loop edges fired (restate, parallel fan-out, marketer-only blackboard write, founder synthesis with conflict, serial dispatch, critic gate with conditional → go transition, two stage-reports, inline mediation). Topology v1 validated.
- 2026-05-18 [decision] Retire blackboard root open question q5 (CLAUDE.md) — **done** via commit prior to dry-run; project memory in place at project root.

## Open questions

1. Where the runtime `blackboard.md` lives when the system is deployed against a real project — default if silent: project root, same level as `.claude/`.
2. Whether to implement the fallback `reporter` agent now or defer until context pressure is observed — default if silent: defer.
3. Whether to add automatic growth-trigger detection (e.g., context saturation warning) — default if silent: defer, rely on manual founder judgement.
4. Whether to enable GitHub Pages now (repo Settings → Pages → branch: main, folder: /docs) — default if silent: defer to user; instructions live in page footer.
5. Whether to add SRI hash to Mermaid CDN script before public link-out — default if silent: defer; revisit if page is shared on LinkedIn or similar professional channel.
6. Whether to extend Critic's review scope to include visual-render verification (eyeball check, not just claims/a11y/security) — default if silent: leave as founder responsibility; Critic stays artifact-claim-focused.

## Positioning

_Authored by Marketer IC — 2026-05-18. All claims cite blackboard ## Spec or ## Decisions._

**One-liner**: A reference implementation of a flat agentic startup topology, built for engineers who think in tradeoffs.

**ICP**: Recruiters and engineers evaluating the author's engineering judgment — people who read a portfolio page to understand how someone thinks, not just what they shipped.

**Value prop**: Readers leave with a concrete mental model of how to structure multi-agent systems without bureaucratic overhead — and evidence the author made deliberate tradeoffs, not just default choices.

**Objections**

| Objection | Response | Cites |
|---|---|---|
| "This is just a prompt collection, not real engineering." | The topology encodes specific write-asymmetry rules, a severity ladder, a verification ladder, and a stage-report cadence — each a design decision with a stated reason. | ## Decisions: 2026-05-15 (verification ladder, Critic gate, Marketer-only write) |
| "Why not use an existing orchestration framework?" | The flat topology deliberately cuts the DAG planner, sub-orchestrators, and typed envelopes. That's the thesis — not an oversight. | ## Spec: Topology (v1), agentic-startup-topology.md "What is cut vs a corporate org" |
| "How does this scale beyond a demo?" | Growth triggers are explicit: add a layer only when context saturation, recurring conflict, or irreversible actions are observed. The point is to stay flat until pain shows up. | ## Decisions: 2026-05-15 (flat variant); agentic-startup-topology.md "Growth triggers" |
