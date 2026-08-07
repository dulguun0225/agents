# Global Claude Code subagents

Reusable subagent definitions with per-task model and reasoning-effort routing: cheap models for cheap work, expensive ones only where quality depends on them. Claude Code picks an agent by matching the task against each `description` field.

## Routing table

| Agent | Model | Effort | Use for |
|---|---|---|---|
| `scout` | haiku | low | Locating files, symbols, usages ("where is X") |
| `docs-writer` | sonnet | medium | README, changelogs, comments, docstrings |
| `coder` | sonnet | medium | Well-scoped features, known-cause fixes, mechanical refactors, tests |
| `reviewer` | opus | high | Read-only diff review before committing |
| `deep-worker` | inherit | high | Gnarly bugs, concurrency, performance, cross-cutting refactors |
| `architect` | inherit | high | Read-only design/planning when the approach is not obvious |
| `spec-author` | inherit | high | Drafting spec.md / plan.md / tasks.md under `specs/**`, rule authoring |
| `researcher` | sonnet | high | Web evidence gathering: dated claims, registry/version checks, candidate surveys |
| `refuter` | inherit | high | Adversarial panelist: steelman, then refutation vote on a proposed decision |

`deep-worker`, `architect`, `spec-author`, and `refuter` omit `model:` so they inherit the session model — pinning them would cap quality below the session tier (e.g. Fable) exactly where being wrong is expensive.

Escalation path: `scout` → `coder` → `deep-worker`; plan with `architect` first when the approach is unclear; run `reviewer` after any non-trivial change.

### ASDLC skill mapping

The set covers the ASDLC skill collection by work shape, not one agent per skill. Lifecycle documents (`asdlc-spec`, `asdlc-plan`, `asdlc-tasks`) run in the main session (they use AskUserQuestion); `spec-author` drafts sections on direction, `architect` does the read-only design exploration feeding the plan. `asdlc-implement` defaults to `deep-worker` — tests derive from signed requirements and the domain rule corpora (money, caching, async-handoff) are correctness-critical — with `coder` for purely mechanical tasks: `*-java` gate wiring, catalog entries, tasks.md bookkeeping. `tech-decision-research` fans out `researcher` for evidence gathering and spawns `refuter` for the panel and the independent refutation votes. `reviewer` audits diffs against whatever domain rule skills are installed. Rule-corpus skills themselves get no agent: they are context loaded by whichever agent edits or reviews.

## Install

Agents are global when they live in `~/.claude/agents/`. Everything Claude-specific sits under `claude/` in this repo; link the subdirectories there instead of copying, so `git pull` updates them in place.

Windows (junction, no admin needed):

```powershell
git clone <repo-url> D:\repos\dulguun0225\agents
New-Item -ItemType Junction -Path "$HOME\.claude\agents" -Target D:\repos\dulguun0225\agents\claude\agents
```

If `~/.claude/agents` already exists, move its contents into the clone first, then delete the directory and create the junction.

macOS/Linux:

```sh
git clone <repo-url> ~/repos/agents
ln -s ~/repos/agents/claude/agents ~/.claude/agents
```

## Workflows

`claude/workflows/` holds saved multi-agent workflows, junctioned to `~/.claude/workflows/` the same way (available in every project, invoked as `/<name>`):

| Workflow | Stages (model/effort) | Use for |
|---|---|---|
| `/research-lite` | plan (sonnet/medium) → search (haiku/low, one per angle, ≤4 sources each) → verify load-bearing claims (sonnet/medium, ≤6) → report (opus/high) | Web research that needs citations but not the full `/deep-research` fan-out |

```powershell
New-Item -ItemType Junction -Path "$HOME\.claude\workflows" -Target D:\repos\dulguun0225\agents\claude\workflows
```

The built-in `/deep-research` offers no per-stage model/effort control (only session-wide `effortLevel` and the advisory `workflowSizeGuideline`); `/research-lite` exists to route each stage to the cheapest model that holds quality.

## Frontmatter fields used

- `model`: `haiku` | `sonnet` | `opus` | `fable` | full model ID | `inherit` (default)
- `effort`: `low` | `medium` | `high` | `xhigh` | `max`
- `tools`: comma-separated allowlist; read-only agents (`scout`, `reviewer`, `architect`) get no `Edit`/`Write`
- `color`: task-list display color

Full field reference: https://code.claude.com/docs/en/sub-agents.md

## Editing

Change a file, commit, push. Sessions pick up changes on next agent spawn; no reinstall step.
