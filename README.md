# Global Claude Code subagents

Reusable subagent definitions with per-task model and reasoning-effort routing: cheap models for cheap work, expensive ones only where quality depends on them. Claude Code picks an agent by matching the task against each `description` field.

## Routing table

| Agent | Model | Effort | Use for |
|---|---|---|---|
| `scout` | haiku | low | Locating files, symbols, usages ("where is X") |
| `docs-writer` | sonnet | medium | README, changelogs, comments, docstrings |
| `coder` | sonnet | medium | Well-scoped features, known-cause fixes, mechanical refactors, tests |
| `reviewer` | inherit | high | Read-only diff review before committing |
| `deep-worker` | inherit | high | Gnarly bugs, concurrency, performance, cross-cutting refactors |
| `architect` | inherit | high | Read-only design/planning when the approach is not obvious |
| `spec-author` | inherit | high | Drafting spec.md / plan.md / tasks.md under `specs/**`, rule authoring |
| `researcher` | sonnet | high | Web evidence gathering: dated claims, registry/version checks, candidate surveys |
| `refuter` | inherit | high | Adversarial panelist: steelman, then refutation vote on a proposed decision |

`deep-worker`, `architect`, `spec-author`, `refuter`, and `reviewer` omit `model:` so they inherit the session model — pinning them would cap quality below the session tier (e.g. Fable) exactly where being wrong is expensive. Review is the last gate before commit; a pinned reviewer would judge the session model's work with a weaker model.

Escalation path: `scout` → `coder` → `deep-worker`; plan with `architect` first when the approach is unclear; run `reviewer` after any non-trivial change.

## Install

Agents are global when they live in `~/.claude/agents/`. Everything Claude-specific sits under `claude/` in this repo; link the subdirectories there instead of copying, so `git pull` updates them in place.

Windows (junction, no admin needed):

```powershell
git clone <repo-url> D:\repos\dulguun0225\agents
New-Item -ItemType Junction -Path "$HOME\.claude\agents" -Target D:\repos\dulguun0225\agents\claude\agents
Copy-Item D:\repos\dulguun0225\agents\claude\CLAUDE.md "$HOME\.claude\CLAUDE.md"
```

If `~/.claude/agents` already exists, move its contents into the clone first, then delete the directory and create the junction.

`claude/CLAUDE.md` is the global user CLAUDE.md — it carries the Workflow-script routing rule that makes ultracode/Workflow scripts use this repo's agent definitions (`agentType`) and per-stage `model`/`effort` instead of defaulting every agent to the session model. It must be a copy, not a junction: junctions are directory-only, and a file symlink needs admin or Developer Mode. After editing it in the repo, re-run the `Copy-Item`. With Developer Mode on, a symlink removes that step:

```powershell
New-Item -ItemType SymbolicLink -Path "$HOME\.claude\CLAUDE.md" -Target D:\repos\dulguun0225\agents\claude\CLAUDE.md
```

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

Workflow-tool scripts do **not** consult the routing table automatically — `agent()` calls default to the session model. Each call must pass `agentType: '<agent>'` (pulls the definition's model/effort/tools/prompt) or explicit `model`/`effort`. That is deliberate for ultracode: it buys maximum quality, so its workflows stay unrouted (the "Workflow-script routing" rule in `claude/CLAUDE.md`, copied to `~/.claude/CLAUDE.md` on install, says exactly that). Cost-routed orchestration is opt-in via the `/workflow-light` skill below. The `Agent` tool needs neither — it auto-routes by `description` match.

## Skills

`claude/skills/` holds skills, junctioned per skill directory into `~/.claude/skills/` (which also holds skills installed by other means, so the whole directory cannot be junctioned):

| Skill | What it does |
|---|---|
| `/workflow-light` | Ultracode-style orchestration (same decomposition, fan-out, adversarial verification, synthesis) with per-stage cost routing: each `agent()` call gets the cheapest model+effort that holds quality, via `agentType` for stages matching a defined agent or explicit `model`/`effort` otherwise. Judgment stages always inherit the session model. Routing table in the skill mirrors the agent table above; the README is the source of truth on conflict. |

```powershell
New-Item -ItemType Junction -Path "$HOME\.claude\skills\workflow-light" -Target D:\repos\dulguun0225\agents\claude\skills\workflow-light
```

## Frontmatter fields used

- `model`: `haiku` | `sonnet` | `opus` | `fable` | full model ID | `inherit` (default)
- `effort`: `low` | `medium` | `high` | `xhigh` | `max`
- `tools`: comma-separated allowlist; read-only agents (`scout`, `reviewer`, `architect`, `refuter`) get no `Edit`/`Write`; `scout` also gets no `Bash` — Glob/Grep cover search, and no Bash means no shell escape from read-only
- `color`: task-list display color

Full field reference: https://code.claude.com/docs/en/sub-agents.md

## Editing

Change a file, commit, push. Sessions pick up changes on next agent spawn; no reinstall step.
