# Eval results

Newest first. Format: date, session model, per-eval verdict, notes worth acting on.

## 2026-08-07 — session model Fable 5 (first full run)

| Eval | Verdict | Notes |
|---|---|---|
| routing | **Pass** 20/20 | Judge: sonnet, no tools. |
| reviewer | **Pass** 3/3 | Zero false positives, correct "blocked" verdict, 17s / ~9k tokens. |
| refuter | **Pass** | Steelman cited real claude-code GitHub issues; refuted the leaning on its false premise (junctions ≠ symlinks, no privilege needed) plus drift cost; found the 14 foreign junctions in `~/.claude/skills` unprompted. |
| scout | **Pass** | All 3 correct, `path:line` format; also caught `effort:` keys inside research-lite.js — thorough, not noise. |
| prober | **Pass** (minor) | Facts exact, error quoted verbatim. Minor: added root-cause analysis and a "working alternative" beyond what was asked — borderline vs "reports observed state only"; watch whether it ever recommends state changes. |
| coder | **Pass** | One-line fix, test actually run, output `ok` reported. |
| researcher | **Pass** | Correct (Active LTS = 24, Current = 26), canonical source (nodejs/Release schedule.json), explicitly separated inference from quoted fact, flagged a WebFetch summarizer artifact as unverified. |

Static validator: clean on the repo; negative test with seeded drift (bad effort value, write tools on scout, README/table mismatch) produced 6 failures and exit 1.

Incidents worth keeping:
- `Workflow {name: 'research-lite'}` was rejected: "script contains control characters" — the working-tree file had CRLF line endings (git autocrlf). Fixed with `.gitattributes` (`claude/workflows/*.js text eol=lf`) + renormalize; by-name invocation still failed in the same session (script cached at session start), `scriptPath` invocation worked. New sessions pick up the LF file.
- mise's `node` shim is broken on this machine (default version resolution); evals that run node need an explicit `mise x node@<version> -- node`.

Live integration runs (same day):
- `/research-lite` on "PostgreSQL supported versions + EOL dates as of 2026-08-07": **pass** — 13 agents, 0 errors, 87s, ~319k subagent tokens; answer correct (14–18 supported, 14 EOL 2026-11-12), verified vs unverified claims kept separate, all cited.
- `/workflow-light` on a docs-vs-reality audit: **pass** — the authored workflow routed per the SKILL.md table (prober+scout haiku/low → reviewer ×3 inherit/high → refuter per finding → inherit synthesis), 8 agents, ~55k subagent tokens, and correctly found the 2 real drifts (CLAUDE.md's stale "no test step" claim and the editing workflow omitting the validator) with 0 false findings. Both drifts fixed in the same change.
