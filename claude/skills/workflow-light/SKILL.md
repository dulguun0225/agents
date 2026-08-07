---
name: workflow-light
description: Ultracode-style multi-agent orchestration with per-stage cost routing. Authors a Workflow with the same structure ultracode would use (decompose, fan out, adversarially verify, synthesize) but picks each agent's model and effort from the task type instead of running everything on the session model. Use when the user invokes /workflow-light <task> or asks for a light/cheap/budget workflow. Not for maximum-quality runs - that is ultracode, which must stay unrouted; not for web research presets - /research-lite already covers that.
---

# workflow-light

Invoking this skill is the user's explicit opt-in to the Workflow tool for the given task.

Author and run a Workflow exactly as you would under ultracode — same decomposition, same fan-out width, same adversarial verification depth, same synthesis stage. The only difference is routing: every `agent()` call gets the cheapest model+effort that holds quality for its stage. Savings come from routing, never from fewer agents or shallower verification.

## Routing

Pick per stage, by the kind of work the agent does:

| Stage kind | Route | `agentType` |
|---|---|---|
| Locate code, files, symbols, usages | haiku / low | `scout` |
| Web search, claim extraction, per-source reading | haiku / low | — |
| Dedup, format conversion, counting, list merging | haiku / low | — |
| Well-scoped edit with a decided approach | sonnet / medium | `coder` |
| Prose artifacts: docs, changelogs, comments | sonnet / medium | `docs-writer` |
| Verify one claim against its cited source | sonnet / medium | — |
| Evidence gathering with dated claims, registry/version checks | sonnet / high | `researcher` |
| Design, planning, approach choice | inherit / high | `architect` |
| Hard bugs, concurrency, cross-cutting changes | inherit / high | `deep-worker` |
| Diff review, defect hunting, commit verdicts | inherit / high | `reviewer` |
| Refutation votes, adversarial judging | inherit / high | `refuter` |
| Lifecycle docs under specs/** | inherit / high | `spec-author` |
| Final synthesis, cross-agent conclusions | inherit / high | — |

Rules:

- When a stage matches a defined agent, pass `agentType` — it pins the definition's model, effort, tools, and system prompt in one field. Otherwise set `model`/`effort` explicitly per the table.
- Judgment stages (the inherit rows) omit `model` so they run on the session model. Never pin them lower; the routed agents (`architect`, `deep-worker`, `reviewer`, `refuter`, `spec-author`) already inherit.
- A stage that fits two rows gets the higher one. Unsure which bucket: the higher one. Quality first, token efficiency second.
- Findings from cheap finders still get session-model verification when they gate a conclusion (commit verdict, decision vote, final report claim).
- In the final report, state the routing used: agents per stage and their models, so the user can judge the trade.

The routing table above mirrors the agent table in the dulguun0225/agents README; when they disagree, the README is the source of truth.
