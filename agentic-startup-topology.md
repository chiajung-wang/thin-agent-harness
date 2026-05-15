# Agentic Startup Topology

Flat, low-bureaucracy agentic system designed using a startup company as the framing metaphor. Optimized for speed, single-stakeholder intent, and 0→1 iteration.

## Topology

```
                  ┌─────────────────┐
                  │   Founder/CEO   │  ← root orchestrator
                  │  (intent owner) │
                  └────────┬────────┘
                           │
   ┌────────────┬──────────┼──────────┬────────────┬──────────┐
   ▼            ▼          ▼          ▼            ▼          ▼
[Builder]  [Designer] [Researcher] [Marketer]  [Critic]   (spawn-on-demand)
   │            │          │          │            │
   └────────────┴────┬─────┴──────────┴────────────┘
                     ▼
             [shared blackboard]
             spec, decisions, memory, positioning
```

## What is cut vs a corporate org

| Killed | Why |
|---|---|
| Department sub-orchestrators | Founder routes direct to ICs. No middle layer = no telephone game. |
| Typed envelope schemas | Loose JSON `{task, context, output}`. Speed over shape. |
| DAG planner | Founder picks next step inline. Replan after each result. |
| Legal / compliance agent | Folded into Critic. Spawn dedicated guardrail only when domain demands it (fintech, health, regulated data). |
| Multi-round mediation | One conflict pass, founder decides, move on. |
| Per-deliverable performance review | Critic runs once at the end, not on every artifact. |
| Stakeholder matrix | Single stakeholder = founder's intent. Add more once the product hits users. |

## Operating rules

1. **Founder wears three hats**: planner + router + synthesizer. One context window holds intent end-to-end.
2. **ICs are generalists, not specialists**. `Builder` covers code + config + infra. `Researcher` covers docs + web + codebase grep. `Marketer` covers positioning + copy + launch comms + channel selection. Spawn a specialist only when a generalist fails twice on the same domain.
3. **Blackboard = single markdown file**. Spec, decisions, and open questions live in one place. No database, no message bus. ICs read, append, done.
4. **Async fan-out, sync synthesis**. Founder dispatches 2–3 ICs in parallel when work is independent. Waits for all. Synthesizes inline.
5. **Critic runs last-mile only**. Gate before ship, not before every step. Tradeoff: faster iteration, occasional rework.
6. **Human-in-loop = founder is human**. No approval gates between agents. Founder approves at the end of the loop.
7. **Lean memory**. Append decisions and reversals to the blackboard. Skip full ADRs until repeated pain shows up.

## Accepted tradeoffs

- **Founder bottleneck**: scales poorly past ~5 parallel ICs. Acceptable for 0→1.
- **No audit trail**: blackboard overwrites. Add git on the blackboard when stakes rise.
- **Critic runs too late**: a bad direction is caught at the end and may force a full redo. Mitigation: founder spot-checks mid-flight with cheap calls.
- **Generalist ICs hallucinate domain detail**: hire a specialist after the second miss.

## Growth triggers (when to add bureaucracy back)

| Signal | Add |
|---|---|
| Founder context window saturated | Sub-orchestrator per domain |
| Same conflict recurs 3× | Mediation protocol |
| More than one stakeholder | Synthesis checklist |
| Irreversible actions in scope | Guardrail agent + approval gate |
| Audit or regulated workload | Typed envelopes + ADR log |

Default: start flat, layer only on observed pain. A premature org chart is premature optimization.

## Marketer IC scope

Single agent, multi-surface. Reads blackboard for product truth, writes positioning artifacts back.

| Sub-task | Output |
|---|---|
| Positioning | One-liner, ICP, value prop, top 3 objections |
| Copy | Landing headlines, CTA variants, feature → benefit rewrites |
| Launch comms | Tweet/X thread, LinkedIn post, HN/Reddit pitch, changelog entry |
| Channel pick | Rank 2–3 channels by ICP fit + cost, not all of them |
| Narrative QA | Flag spec changes that break existing positioning |

**Coordination rules**

- Runs **parallel to Builder** once spec stabilizes. No need to wait for shipped code — works from blackboard spec.
- **Re-runs on spec delta**. Founder marks positioning-affecting changes; Marketer re-syncs only those, not full rewrite.
- **Hands off to Critic** for tone + claim accuracy before ship. Critic checks: claim provable from spec, no overpromise, on-voice.
- **Does not own distribution execution**. Drafts artifacts; founder or Builder posts. Keeps Marketer stateless and cheap.

**Failure modes**

- Drifts from product truth → enforce: every claim must cite a blackboard line.
- Generic AI-voice copy → seed with founder voice samples in blackboard.
- Premature channel commitment → cap to 2 channels at 0→1.

## Minimal contract (loose JSON)

```json
{
  "task": "string — what to do",
  "context": "string | object — relevant blackboard slice",
  "output": "string | object — deliverable",
  "notes": "string — assumptions, risks, open questions"
}
```

No schema validation. Founder reads `notes` to decide replan vs accept.

## Founder loop (pseudocode)

```
load blackboard
while not done:
    next = pick_next_step(intent, blackboard)
    if next is parallel_batch:
        results = fan_out(next.tasks)        # spawn ICs in parallel
    else:
        results = [dispatch(next.task)]
    append(blackboard, results)
    if conflict(results):
        resolve_inline(results)              # one pass, no mediation rounds
    done = intent_satisfied(blackboard)
critic_review(blackboard)
ship_or_revise()
```
