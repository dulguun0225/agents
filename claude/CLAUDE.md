# User instructions

## Writing style

Always: concise first, precise second, simple third. Keep technical terms when the everyday word is less exact. No business-speak or figurative
speech; say what actually happens.

The wording rules apply everywhere. Coverage defaults to complete —
every edge case. The one exemption is chat and terminal session
replies: answer what was asked; include an edge case only when it
changes the answer. Anything used outside the session — a file, a
spec, a commit message, a code comment — is complete even when
drafted inside a reply.

## Workflow-script routing

In Workflow-tool scripts (ultracode included), route each `agent()`
call per the routing table in ~/.claude/agents (repo
dulguun0225/agents, README): pass `agentType` when the stage matches
a defined agent; otherwise set `model`/`effort` per stage. Mechanical
stages (locate, search, extract, dedup, mechanical edits) get
haiku/low or sonnet/medium; judgment stages (verify, judge, review,
design, synthesize) omit `model` so they inherit the session model.
Never pin a judgment stage lower to save tokens — quality first,
token efficiency second.
