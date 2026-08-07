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

Exception — ultracode on: ultracode buys maximum quality and
declares token cost no constraint, so this rule must not shrink it.
Do not down-route to save tokens: every stage inherits the session
model unless model tier provably cannot change output quality
(dedup, format conversion, counting). `agentType` pins the agent's
frontmatter model (e.g. scout = haiku); there is no `model: inherit`
override — under ultracode either pass `agentType` with `model` set
explicitly to the session tier, or skip `agentType` and inline the
agent's rules in the prompt. Never
reduce ultracode's default fan-out, verification depth, or agent
count because of this table.
