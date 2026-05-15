---
name: stage-report
description: Render a stage-boundary report that humans can read at a glance and agents can re-ingest losslessly. Use at planned stage boundaries (after a spec lands, after a build phase, before ship, after a major decision) — not for ad-hoc summaries. Produces three artifacts in sync: a Markdown narrative for the conversation, an HTML diagram for the human, and a JSON twin for agents. Goal is human-agent shared mental model with consistent shape so the human builds muscle memory across stages.
---

# stage-report

Stage-boundary report. Same underlying state model rendered three ways: Markdown (in-chat), HTML (human, with diagram), JSON (agent, machine-readable). All three must come from one render pass — never hand-edit one without the others.

## When to invoke

**Fire on:**
- Stage boundary declared by founder (spec frozen, build phase done, pre-ship gate, post-decision checkpoint)
- Explicit user request (`/stage-report` or "give me a report")
- Critic gate before ship

**Do NOT fire on:**
- Mid-task progress check (founder summarizes inline instead)
- Single-file edit completion
- Inside a Builder or Researcher loop

If called outside a stage boundary, return early with one line: `stage-report skipped: no stage boundary; founder should summarize inline`.

## Inputs

Read these in order. Stop if `blackboard.md` is missing — report cannot be rendered without it.

1. `./blackboard.md` — full file (intent, spec, decisions, open questions, positioning)
2. `./reports/latest.json` — prior report if it exists (for state delta)
3. Recent agent outputs in current conversation context (within ~2000 tokens of the boundary)

## Outputs

Write all three. Same content, three shapes.

```
./reports/<YYYY-MM-DD-HHmm>-<slug>.html
./reports/<YYYY-MM-DD-HHmm>-<slug>.json
./reports/latest.html      ← overwrite, same content as newest dated html
./reports/latest.json      ← overwrite, same content as newest dated json
```

`<slug>` = kebab-case stage label, e.g. `spec-frozen`, `build-phase-1`, `pre-ship`.

Markdown narrative is printed to the conversation directly, not written to disk.

## The four layers (fixed order, fixed names)

Every report — Markdown, HTML, JSON — uses these four sections in this order. Skip a section if it has no content; never rename or reorder.

1. **TL;DR** — one sentence: where we are, what changed since last stage
2. **Mental model** — 3–5 bullets naming the *concepts* in play right now (not tasks). Each bullet is a noun phrase + one clarifying clause. This is what builds the human's intuition.
3. **State delta** — what moved since last report: decisions made, assumptions added, risks surfaced, scope changes. Bulleted, each item tagged `[decision]`, `[assumption]`, `[risk]`, `[scope]`.
4. **Open questions** — what the human must decide before the next stage. Numbered. Each has a default if the human stays silent.

## The diagram model

Single graph. Same nodes and edges in HTML and JSON. The HTML renders it via Mermaid; the JSON is the structured twin.

### Node types

| type | meaning |
|---|---|
| `intent` | The single root goal. Exactly one per report. |
| `component` | A part of the system being built (module, page, service). |
| `decision` | A choice made and logged. |
| `question` | An open question awaiting human input. |
| `risk` | A surfaced risk. |
| `milestone` | A stage boundary marker. |

### Node status

`pending` · `in_progress` · `done` · `blocked` · `deferred`

### Edge kinds

| kind | meaning |
|---|---|
| `depends_on` | Target must exist before source can complete. |
| `decided_by` | Source component shape is fixed by target decision. |
| `blocks` | Source is blocked by target question or risk. |
| `supersedes` | Source replaces target (old decision retired). |
| `produces` | Source component produces target artifact. |

Keep the graph small: aim for ≤ 15 nodes per report. If more, collapse by component group.

## JSON schema

```json
{
  "schema_version": "1",
  "stage": "string — kebab-case label",
  "timestamp": "ISO8601",
  "intent": "one-sentence goal, copied verbatim from blackboard",
  "tldr": "one sentence",
  "mental_model": ["string", "string"],
  "state_delta": [
    {"kind": "decision|assumption|risk|scope", "text": "string"}
  ],
  "open_questions": [
    {"id": "q1", "text": "string", "default_if_silent": "string"}
  ],
  "diagram": {
    "nodes": [
      {"id": "string", "label": "string", "type": "intent|component|decision|question|risk|milestone", "status": "pending|in_progress|done|blocked|deferred"}
    ],
    "edges": [
      {"from": "node_id", "to": "node_id", "kind": "depends_on|decided_by|blocks|supersedes|produces"}
    ]
  },
  "prior_report": "filename of last report or null"
}
```

## HTML template

Single self-contained file. Mermaid via CDN. No build step. Open in any browser.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Stage Report — {{stage}} — {{timestamp}}</title>
<style>
  body { font: 15px/1.5 ui-sans-serif, system-ui, sans-serif; max-width: 900px; margin: 2rem auto; padding: 0 1rem; color: #222; }
  h1 { font-size: 1.4rem; margin-bottom: 0.2rem; }
  .meta { color: #888; font-size: 0.9rem; margin-bottom: 1.5rem; }
  h2 { font-size: 1.1rem; margin-top: 2rem; border-bottom: 1px solid #eee; padding-bottom: 0.3rem; }
  .tldr { font-size: 1.05rem; background: #f6f8fa; padding: 0.8rem 1rem; border-left: 3px solid #0969da; }
  .tag { display: inline-block; font-size: 0.75rem; padding: 0.1rem 0.5rem; border-radius: 3px; margin-right: 0.4rem; }
  .tag-decision { background: #dcfce7; color: #166534; }
  .tag-assumption { background: #fef3c7; color: #92400e; }
  .tag-risk { background: #fee2e2; color: #991b1b; }
  .tag-scope { background: #e0e7ff; color: #3730a3; }
  ol li { margin-bottom: 0.6rem; }
  .default { color: #666; font-size: 0.9rem; font-style: italic; }
  .mermaid { background: #fafbfc; padding: 1rem; border-radius: 6px; }
</style>
<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>mermaid.initialize({ startOnLoad: true, theme: 'default' });</script>
</head>
<body>
<h1>Stage Report — {{stage}}</h1>
<div class="meta">{{timestamp}} · intent: <em>{{intent}}</em></div>

<h2>TL;DR</h2>
<div class="tldr">{{tldr}}</div>

<h2>Mental model</h2>
<ul>{{#mental_model}}<li>{{.}}</li>{{/mental_model}}</ul>

<h2>State delta</h2>
<ul>
{{#state_delta}}<li><span class="tag tag-{{kind}}">{{kind}}</span>{{text}}</li>{{/state_delta}}
</ul>

<h2>Open questions</h2>
<ol>
{{#open_questions}}<li>{{text}}<br><span class="default">default if silent: {{default_if_silent}}</span></li>{{/open_questions}}
</ol>

<h2>Diagram</h2>
<div class="mermaid">
graph TD
{{#diagram.nodes}}  {{id}}["{{label}}"]:::s{{status}}
{{/diagram.nodes}}
{{#diagram.edges}}  {{from}} -->|{{kind}}| {{to}}
{{/diagram.edges}}
  classDef spending fill:#f6f8fa,stroke:#888;
  classDef sin_progress fill:#dbeafe,stroke:#0969da;
  classDef sdone fill:#dcfce7,stroke:#166534;
  classDef sblocked fill:#fee2e2,stroke:#991b1b;
  classDef sdeferred fill:#f3f4f6,stroke:#666,stroke-dasharray: 4 2;
</div>
</body>
</html>
```

Render by string substitution. Mustache-style markers above are illustrative — when writing the HTML, just interpolate the values from the JSON directly.

## Markdown narrative (printed to chat)

```
## Stage Report — <stage>
<timestamp>

**TL;DR.** <one sentence>

**Mental model**
- <bullet>
- <bullet>

**State delta**
- [decision] <text>
- [risk] <text>

**Open questions**
1. <text> — default if silent: <text>

Report files: ./reports/<filename>.html · ./reports/<filename>.json
```

Keep narrative under 30 lines. Diagram lives only in HTML/JSON, not Markdown — the chat is for reading, the HTML is for seeing.

## Self-guard

Before writing anything, confirm:

- `./blackboard.md` exists and has an `## Intent` section
- A stage boundary is declared (founder said so, user invoked, or Critic called pre-ship)
- Prior `./reports/latest.json` was read if it exists, so state delta is real not invented

If any check fails, abort with one line stating which check failed. Do not write partial reports.

## Versioning

This skill is `v1`. Do not mutate the four-layer shape in place — if the project outgrows it, ship `stage-report-v2` as a sibling skill so trained human readers don't lose pattern recognition mid-project.
