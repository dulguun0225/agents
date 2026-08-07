# Evals

Golden-task regression suite for the agent definitions. Static consistency is `checks/validate.py`; these test behavior, which the validator cannot.

## How to run

Each eval file states the exact prompt, the answer key, and a pass/partial/fail rubric. Run by spawning the named agent (Agent tool, `subagent_type` = the agent) with the prompt, then judge the report against the rubric. Independent evals run in parallel. Judge against the key, not against what sounds plausible.

When to run:
- After editing an agent's `description` or rules: that agent's eval + `routing.md`.
- After adding an agent: add a boundary task to `routing.md` first, then run it.
- Before a release-ish commit touching several agents: the whole suite.

## Suite

| Eval | Agent | What it proves |
|---|---|---|
| `routing.md` | (judge on sonnet) | `description` fields route 20 tasks to the intended agents |
| `reviewer/task.md` | reviewer | Catches 3 seeded bugs in a "pure refactor" diff, no false positives |
| `refuter.md` | refuter | Refutes a plausible-but-wrong leaning instead of endorsing it |
| `scout.md` | scout | Locates definitions/usages, terse `path:line` output |
| `prober.md` | prober | Reports link/system state exactly, including broken state, without prescribing fixes |
| `coder.md` (`coder/task.md`) | coder | Minimal known-cause fix, proves the test passes |
| `researcher.md` | researcher | Dated claims with official citations, inference flagged |

Not covered (judgment quality is hard to rubric; rely on real use + reviewer gate): `architect`, `deep-worker`, `docs-writer`, `spec-author`.

Live integration (run occasionally, they cost real tokens): `/research-lite` on a question with an official schedule as ground truth; `/workflow-light` on a small audit task — check the authored script routes stages per the SKILL.md table and judgment stages stay unrouted.

Results log: `RESULTS.md`.
