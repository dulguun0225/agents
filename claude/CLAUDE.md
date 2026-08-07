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

Under ultracode, Workflow scripts keep their defaults: every stage
inherits the session model; never down-route, pin `agentType`, or
shrink fan-out, verification depth, or agent count to save tokens.
Cost-routed orchestration is opt-in via /workflow-light — same
structure as ultracode, but each stage gets the cheapest
model+effort that holds quality; the routing table lives in that
skill.
