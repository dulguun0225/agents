# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Global Claude Code subagent definitions (`claude/agents/*.md`), saved workflows (`claude/workflows/*.js`), and skills (`claude/skills/<name>/SKILL.md`). The `claude/` subdirectories are junctioned/symlinked into `~/.claude/agents`, `~/.claude/workflows`, and `~/.claude/skills/<name>` (per skill directory), so **edits here are live global config** — sessions on this machine pick up edits to existing files on the next agent spawn; a *newly added* agent file is only visible to sessions started after it exists (the agent list loads at session start). There is no build, test, or lint step; the only validation is correct frontmatter (agents and skills) and workflow-script syntax.

## Routing principle

Priorities, in order:
1. Quality of work done
2. Token efficiency

Concretely: judgment-heavy agents (`deep-worker`, `architect`, `spec-author`, `refuter`, `reviewer`) **omit `model:`** so they inherit the session model — never pin them to a lower tier to save tokens. Mechanical/lookup work gets pinned cheap (`scout`/`prober` = haiku/low, `coder`/`docs-writer` = sonnet/medium). When adding or retuning an agent, place it on this axis first; the README routing table is the source of truth and must be updated in the same change.

## Architecture

- `claude/agents/<name>.md` — one agent per file. Frontmatter fields: `name`, `description`, `tools` (allowlist; read-only agents get no `Edit`/`Write`), `model` (`haiku`|`sonnet`|`opus`|`fable`|full ID|omit = inherit), `effort` (`low`…`max`), `color`. The `description` field is what Claude Code matches tasks against for auto-routing — it must state both when to use the agent and when *not* to (escalation/boundary), since routing quality depends entirely on it.
- Body of each agent file = system prompt: one-line role statement, then a short `Rules:` list. Keep that shape.
- `claude/workflows/<name>.js` — Workflow-tool scripts (plain JS, `export const meta` first). Per-stage `model`/`effort` on each `agent()` call is the point of these workflows: route each stage to the cheapest model that holds quality (see `research-lite.js` — haiku searchers, sonnet verify, opus synthesis).
- `claude/skills/<name>/SKILL.md` — skill definitions, junctioned per skill directory into `~/.claude/skills/<name>`. `workflow-light`'s routing table mirrors the README table; the README wins on conflict and both must change together.
- Escalation path between agents: `scout` → `coder` → `deep-worker`; `architect` before implementation when the approach is unclear; `reviewer` after non-trivial changes.

## Editing workflow

Change a file, update the README routing table if routing changed, commit, push. No reinstall step.
