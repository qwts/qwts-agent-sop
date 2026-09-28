# ENG-0051: Every harness's global instruction file is one generated stub

**Status:** Proposed
**Date:** 2026-09-27
**Issue:** qwts/qwts-agent-sop#51

## Context

Each harness reads a global instruction file from its user directory
([ENG-0384](ENG-0384-harness-config-lives-in-the-user-directory.md)
decision 1). Today every session keeps that file current by hand. The
[org-wide agent conventions](../reference/agent-conventions.md), section
"Global instruction file" (#19, #20), say: "After following
`https://agentsop.ai/llms.txt`, make this harness's global instruction file
match the block below." The agent creates the file, inserts the
`agentsop:global` block, or replaces a stale one, and leaves every other line.
The section names paths for four harnesses: Grok, Claude Code, Codex, and
Cursor. The org roster
([`governance/agents.json`](https://github.com/qwts/qwts-agent-org/blob/62517e731de2d98def5fdcab8d4179adf5773187/governance/agents.json))
lists 21 active harness Apps.

This is the step that
[ENG-0049](https://github.com/qwts/qwts-agent-sop/pull/50) (Proposed, PR #50,
not merged) says must be enforced, not remembered: it happens in every
session, a model performs it, and nothing checks the result. The files drift,
they differ per harness, and keeping about 21 of them by hand keeps coming up
as a sticking point.

## Decision

This record builds on ENG-0049's principle: a step that happens on every
change is enforced deterministically, not remembered. Here the step happens in
every session.

1. **Every global file is the same short generated stub.** It says: start at
   `agentsop.ai`, run `agent-bot`, and a repository's own instructions
   override global ones. It adds only harness-specific facts that must be
   there, such as the identity slug. Nothing else is written into it by hand.
2. **Machinery writes it, from one source.** Machine bootstrap
   (`managed-machine`) or an `agent-bot` command (for example
   `agent-bot harness sync`) writes the stub to each harness's own path and
   format. The proposed home for the stub text and the per-harness path
   table is `agent-bot-identity`, beside `hook-dialects.mjs`, which is already
   that repository's one table of per-harness user-directory paths
   ([ENG-0128](ENG-0128-agent-bot-runtime-ownership.md) decision 1); the
   table's owner stays an open question below. Where
   the file holds other content, the command reports it and does not discard
   it silently.
3. **`agent-bot doctor` reports drift.** It names each harness whose global
   file differs from the stub, or is missing, and prints the command that
   fixes it.
4. **The rewrite rule retires.** Once this is accepted and implemented, the
   conventions section that tells models to rewrite their global file is
   removed by a separate PR. This record does not change the conventions.
5. **The stub rarely changes.** It only points to where the rules live, so
   rule changes land in the SOP and the router, not in 21 files. Maintenance
   drops to near zero.

## Current state

Nothing in this record is implemented. At the `org.json` pin
[`agent-bot-identity@52f55b1`](https://github.com/qwts/agent-bot-identity/tree/52f55b171b41a2cf7ae80e5318246363729ccddf),
`sync-hooks.mjs` writes hook adapters, not instruction files, from
`hook-dialects.mjs` into the user directories of Claude Code (also serving
Devin CLI), Codex, Cursor, Copilot, and Devin Desktop, and has a `--check`
mode. `doctor` checks hook installation and coverage but no instruction file,
and there is no `harness` command. The `managed-machine` setup scripts at
`3276571` write no global instruction file. The router at
[`75f45e2`](https://github.com/qwts/agentsop.ai/tree/75f45e2c23e6ce3317ba4efb56b388df7d7a2cd5)
has no step about it; only the conventions section does.

## Consequences

- One source, drift that is visible, and no per-session edit. A harness gets
  its file when bootstrap or the command runs, the same trade ENG-0384
  decision 4 makes for hooks.
- Cost: the stub, the path table for every active harness (today's hook table
  covers five dialects), the command, and the doctor check in `agent-bot-identity`, plus
  a bootstrap call in `managed-machine`.
- Precedence changes. Today's block says rules already in the global file win;
  the stub says repository instructions win.
- Tension with ENG-0384's amendment of 2026-09-27, which leaves
  [ENG-0006](ENG-0006-agentic-primitives-governance.md) decision 1 (a fact in
  two agent files is a bug) with no copy exception. The stub is one generated,
  drift-checked text in many user files, not a repository copy. If review
  counts it as a copy, this record needs that exception stated.
- Tension with [ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md):
  it keeps one local pointer (decision 2 and its amendment) and rejected a
  copy installed on every machine as a drift source. The global files already
  exist; this record makes them generated and checked.
- "Run `agent-bot`" is orientation, not a clock-in. The ENG-0081 amendment
  of 2026-08-13 (decision 5) says no skill may introduce a required first
  tool call. The stub is not a skill, but no enforcement may depend on the
  agent running it.
- [ENG-0178](ENG-0178-evidence-bound-harness-projections.md) classifies
  harness synchronization pull requests into repositories. It does not cover
  user-directory files, so this record neither builds on it nor changes it.

## Alternatives

- **Keep per-harness hand edits:** rejected. It is the remembered step, and
  the files already drift.
- **One file, symlinked into every harness:** not taken. It gives one file and
  no copies. But harnesses differ in path and format (Cursor needs an `.mdc`
  with frontmatter; Claude Code reads `CLAUDE.md`), harness-specific facts
  cannot differ, a harness that writes to its global file writes to all of
  them, and [ENG-0339](ENG-0339-os-account-determines-persona.md) gives each
  harness its own account and home. The generator may still link where path
  and format match.

## Open questions

- Who owns the per-harness path and format table: `agent-bot-identity`
  beside the hook table, or the `managed-machine` app catalog?
- How the repository `AGENTS.md` relates: the stub defers to it, but does it
  also name it?
- Whether the stub carries identity (the slug, where credentials live) or
  defers to `agent-bot`. In the owner's account a harness is the delegate
  (ENG-0339 decision 3), so a fixed slug there would be wrong.

## References

- ENG-0049, [PR #50](https://github.com/qwts/qwts-agent-sop/pull/50)
  (Proposed)
- [ENG-0006](ENG-0006-agentic-primitives-governance.md),
  [ENG-0081](ENG-0081-transcript-bound-agent-execution-identities.md),
  [ENG-0128](ENG-0128-agent-bot-runtime-ownership.md),
  [ENG-0178](ENG-0178-evidence-bound-harness-projections.md),
  [ENG-0339](ENG-0339-os-account-determines-persona.md),
  [ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md),
  [ENG-0384](ENG-0384-harness-config-lives-in-the-user-directory.md)
- [Org-wide agent conventions](../reference/agent-conventions.md) — the rule
  decision 4 retires
