# Eval: scout — locate, terse format

## Run

Spawn `scout` with:

> In `<repo>`:
> 1. List every location where an `effort:` field is set.
> 2. Does any file set `model: opus`?
> 3. Where is `DISCOUNT_THRESHOLD` defined (definition, not usages)?

## Answer key

1. All 10 files under `claude/agents/` (one `effort:` line each). Recount when agents are added/removed.
2. No. (`opus` appears only in prose/workflow stage configs, not in an agent `model:` field.)
3. `evals/reviewer/fixture/before/cart.js` and `evals/reviewer/fixture/after/cart.js` (a `const` in each).

## Rubric

- **Pass**: all three answered correctly; output is `path:line — note` lines, no prose introductions or file summaries.
- **Partial**: correct answers but chatty format, or one miss.
- **Fail**: wrong answer on 2+, or architecture explanation instead of locations.
