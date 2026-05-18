# CLAUDE.md

Project memory for the `google-agent-skill` repo. Read this on session start before touching anything.

## What this project is

A reference implementation of a **flat agentic startup topology** — a Claude Code harness where a root orchestrator (Founder) delegates to IC agents (Builder, Researcher, Designer, Marketer, Critic) over a shared blackboard, and reports back to the human at stage boundaries via a `stage-report` skill that emits synced HTML + JSON + chat-Markdown artifacts.

The repo is the topology itself, not a product on top of it.

## Topology at a glance

```
           ┌──────────────────┐
           │   Founder/CEO    │  root orchestrator
           │  (intent owner)  │
           └────────┬─────────┘
                    │
 ┌──────────┬───────┼───────┬──────────┬─────────┐
 ▼          ▼       ▼       ▼          ▼         ▼
Builder Designer Researcher Marketer Critic   (spawn on demand)
 │          │       │       │          │
 └──────────┴───┬───┴───────┴──────────┘
                ▼
        ./blackboard.md
        Intent · Spec · Decisions · Open questions · Positioning
```

## Agents (`.claude/agents/`)

| Agent | Role | Writes to blackboard? |
|---|---|---|
| `founder.md` | Plan + route + synthesize. Single intent owner. | Yes — `## Decisions`, `## Spec` |
| `builder.md` | Code, config, infra. Verification ladder. | No |
| `researcher.md` | Facts only, cited. Codebase + library docs + web. | No |
| `designer.md` | UI/UX spec for Builder to implement. | No |
| `marketer.md` | Positioning, copy, launch comms. | Yes — `## Positioning` only |
| `critic.md` | One-pass ship gate. Severity-tagged findings, never fixes. | No |

Every IC follows the same shape: frontmatter → responsibilities → JSON in/out contract → operating rules → skill-reference table → anti-patterns → escalation → blackboard rules.

## The blackboard (`./blackboard.md`)

Single shared markdown file. Sections, in fixed order:

- `## Intent` — one-sentence goal. Immutable unless user changes it.
- `## Spec` — current product/feature truth.
- `## Decisions` — append-only log, dated.
- `## Open questions` — surfaced by ICs, resolved by founder or user.
- `## Positioning` — owned by Marketer.

**Write rules**

- Founder writes `## Decisions` and `## Spec`.
- Marketer writes `## Positioning` (only IC with write access).
- All other ICs are read-only; they return through founder synthesis.
- Never overwrite `## Intent` without explicit user confirmation.

## Founder loop

```
load blackboard
restate intent in one sentence
while not done:
    next = pick_next_step(intent, blackboard)
    if next.parallel:
        results = fan_out(next.tasks)   # spawn ICs concurrently via Agent tool
    else:
        results = [dispatch(next.task)]
    append(blackboard, results)
    if conflict(results):
        resolve_inline(results)         # one mediation pass, then escalate
    done = intent_satisfied(blackboard)
dispatch(critic, blackboard)
invoke_skill("stage-report")
present_to_user()
```

## Stage reporting (`.claude/skills/stage-report/`)

Fire at stage boundaries: spec frozen · build phase done · pre-ship gate · post-decision checkpoint · explicit user request.

Do NOT fire mid-task. Founder summarizes inline instead.

Emits three artifacts from one render pass:

- chat Markdown narrative (printed to conversation)
- `./reports/<YYYY-MM-DD-HHmm>-<slug>.html` — human, Mermaid diagram via CDN
- `./reports/<YYYY-MM-DD-HHmm>-<slug>.json` — agent, machine-readable twin
- `./reports/latest.{html,json}` — overwritten mirror to newest

Fixed four-layer template (never reorder, never rename): **TL;DR · Mental model · State delta · Open questions**. Versioned `v1` — shape changes ship as sibling skill.

## addy-agent-skills wiring

Every IC + founder has a `## Skill references (addy-agent-skills)` table mapping triggers to specific skills from the `addy-agent-skills` marketplace. Invoke via the `Skill` tool. Log invocations in the return JSON `skills_invoked` field so founder can audit which playbooks shaped the output.

Common triggers:

| Situation | Skill |
|---|---|
| Vague intent | `agent-skills:interview-me`, `agent-skills:idea-refine` |
| No spec yet | `agent-skills:spec-driven-development` |
| Need to break a task down | `agent-skills:planning-and-task-breakdown` |
| Writing logic / fixing a bug | `agent-skills:test-driven-development` |
| Library or API question | `agent-skills:source-driven-development` (uses `context7`) |
| Bug / unexpected behavior | `agent-skills:debugging-and-error-recovery` |
| Production-stakes decision | `agent-skills:doubt-driven-development` |
| Pre-ship | `agent-skills:shipping-and-launch` + `critic` |

## Conventions

- **Commits**: caveman style, Conventional Commits. Use `/safe-commit here` to commit on the current branch.
- **Branches**: never commit to `main` or `master` directly. `/safe-commit` derives `feat/`, `fix/`, `chore/` etc. unless `here` is passed.
- **Reports under git**: dated `reports/*.{html,json}` are tracked; `reports/latest.*` are git-ignored (regenerable mirrors).
- **Comments**: default to none. Add only when WHY is non-obvious.
- **No new files unless asked.** Prefer editing.
- **No docs files unless asked.** Output goes to chat or report artifacts.

## File map

```
.
├── CLAUDE.md                        ← this file
├── agentic-startup-topology.md      ← topology design doc
├── blackboard.md                    ← harness-level shared memory (meta / dry-runs)
├── .gitignore                       ← includes products/
├── .claude/
│   ├── agents/
│   │   ├── founder.md
│   │   ├── builder.md
│   │   ├── researcher.md
│   │   ├── designer.md
│   │   ├── marketer.md
│   │   └── critic.md
│   ├── skills/
│   │   └── stage-report/SKILL.md
│   └── settings.local.json          ← git-ignored
├── reports/                         ← harness stage-reports only
│   ├── 2026-05-15-1620-init-foundations.{html,json}
│   ├── 2026-05-15-1645-ic-roster-complete.{html,json}
│   └── latest.{html,json}           ← git-ignored mirrors
└── products/                        ← git-ignored; each product has own .git
    └── <product-name>/
        ├── .git/
        ├── CLAUDE.md                ← product-specific memory
        ├── blackboard.md            ← product blackboard
        ├── docs/specs/              ← product specs
        ├── reports/                 ← product stage-reports
        └── ...                      ← product source
```

## Products

Products built by this harness live in `products/<name>/`, each with its own git repo, own remote, own CI. The harness `.gitignore` excludes `products/` so the harness stays product-agnostic and reusable across N products.

**Working in a product:**

- Founder cd's into `products/<name>/` before dispatching IC work on product tasks.
- Each product has its own `CLAUDE.md`, `blackboard.md`, `docs/specs/`, `reports/` — fully self-contained.
- Harness-level work (topology evolution, new IC, skill rewiring) stays at repo root with the harness `blackboard.md`.
- Convention: user names the active product in the prompt; founder cd's first, then loads the product blackboard.

**Currently scaffolded products:**

- `products/prompt-eval-workbench/` — self-hostable prompt + eval workbench for Anthropic FDE workflow. Phase 1 sliced; awaiting Builder dispatch.

## Anti-patterns

- Spawning an IC for a one-line task the founder can do inline.
- Building a multi-step DAG upfront instead of replanning after each result.
- Letting IC outputs land in context without an explicit synthesis step.
- Skipping Critic at ship gate to save a turn.
- Mutating the stage-report shape in place instead of versioning a sibling.
- Writing to the blackboard from an IC other than Marketer or Founder.
- Treating skill tables as decorative; failing to log `skills_invoked` in returns.
- Rewriting `## Intent` silently when the deliverable drifted from the goal.

## Growth triggers (when to layer back)

| Signal | Add |
|---|---|
| Founder context window saturated mid-task | Sub-orchestrator per domain |
| Same conflict recurs 3× | Mediation protocol |
| More than one external stakeholder | Synthesis checklist |
| Irreversible actions in scope | Guardrail agent + approval gate |
| Audit or regulated workload | Typed envelopes + ADR log |

Default: stay flat, layer only on observed pain.
