# SOP: Trait-routed agent startup

How an agent establishes a session before repository work, routing on
observable traits instead of loading the full setup corpus and inferring
which parts apply. This is the startup path referenced by the
[branch, PR, and review workflow](branch-pr-review.md#agent-work-area-mandatory--extend-dont-drop).

> **Guidance class: frontier-comparable** (the class named as
> muse-wonder / GPT-5.6-Terra-comparable). The sequence needs no
> inference: observe the traits, take exactly the matching path, load no
> other branch. Capability changes verbosity only; it never skips a gate.

## Two bindings, two facts

- **Worktree binding** — the worktree's own `agentBot.agentId` pin and
  `agent-binding.json`, written by `setup-worktree`. It says which App
  this checkout is attributed to.
- **Live session binding** — the daemon's live binding for this
  session's transcript, established through the MCP `bind` tool. It is
  what lets a soul-bound caller obtain GitHub credentials.

Confusing them is the failure this SOP exists to prevent:
`setup-worktree` reporting a minted token, and `agent-bot doctor`
reporting the machine ready, neither prove the live session binding.

## The routing

Observe in order; stop at the first trait that fails.

1. **Operation.** Read-only local work needs no GitHub identity. Any
   GitHub or publishing operation requires traits 2–5 to pass first.
2. **Placement.** In a linked worktree under
   `<soulDir>/worktrees/<name>` → proceed. Anywhere else → stop editing;
   check in with `agent-bot join` and create that worktree
   (`agent-space`), then resume the task there. Never edit, branch, or
   commit in the owner's primary checkout.
3. **Worktree binding.** Present and matching the session's soul →
   proceed. Missing or conflicting → run `agent-bot setup-worktree` in
   that worktree. A conflicting pin fails closed; do not overwrite it.
4. **Live session binding** (GitHub operations only). Established →
   proceed. Missing → establish it with MCP `bind` before any `gh`,
   remote, or publishing command. If the harness does not expose the
   MCP `bind` tool, stop and report that exact blocker: the harness,
   the missing tool, and the observed error. There is no alternate
   path — no other identity, no real-binary path, no override
   variable, and no human credential substitutes for it.
5. **Verify, then work.** In the worktree, run `agent-bot doctor` and
   confirm the worktree identity is the soul's App. Only then use
   GitHub operations or qwts CLI mutations.

A session that passes every trait loads none of the remediation for
the others. A session that fails one trait takes only that trait's
path. Loading all of startup "to be safe" and inferring applicability
is the context cost — and the misrouting — this replaces.

## What does not satisfy startup

- Reading a skill, including this one. Skills route; the runtime state
  above is what must exist.
- A soul created with an unregistered harness (`--harness custom`).
  It may proceed for non-GitHub work only; it has no App assignment
  and therefore no GitHub credential path.
- Another agent's soul, worktree, or binding. Identities are never
  borrowed to get past a gate.

## Changelog

- 2026-10-24 — initial version: trait-routed startup, two-bindings
  terminology, unconditional live-session-binding gate with a
  report-the-blocker path (#108).
