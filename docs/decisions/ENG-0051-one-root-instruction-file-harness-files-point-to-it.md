# ENG-0051: One root instruction file per machine; harness global files point to it

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
`agentsop:global` block, or replaces a stale one. The section names paths for
four harnesses: Grok, Claude Code, Codex, and Cursor. The org roster
([`governance/agents.json`](https://github.com/qwts/qwts-agent-org/blob/62517e731de2d98def5fdcab8d4179adf5773187/governance/agents.json))
lists 21 active harness Apps.

This is the step that
[ENG-0049](https://github.com/qwts/qwts-agent-sop/pull/50) (Proposed, PR #50,
not merged) says must be enforced, not remembered: it happens in every
session, a model performs it, and nothing checks the result. The files drift
and differ per harness.

There is no cross-harness standard path for a global instruction file. Each
harness chooses its own: `~/.codex/AGENTS.md`, `~/.claude/CLAUDE.md`, a rule
file under `~/.cursor/rules/`. The one local path every agentsop.ai machine
already has is `~/.config/agent-sop/`, which holds `config.toml`
([ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md),
amendment of 2026-09-16, point 2).

## Decision

This record builds on ENG-0049's principle: a step that happens on every
change is enforced deterministically, not remembered. Decision 4 is an
acknowledged exception to it.

1. **One canonical root instruction file per machine.** It lives at
   `~/.config/agent-sop/AGENTS.md`, next to `config.toml`: the
   organization's own path, since no standard exists. Like `config.toml`, it
   holds no secrets. The agentsop.ai start flow leads agents to it (a router
   change; see Consequences).
2. **Each harness's global file holds a pointer, not a copy.** After an agent
   has read the root file once, it writes a pointer of two or three lines
   into its own harness's global instruction location, for example:

   ```text
   <!-- agentsop:global -->
   Read ~/.config/agent-sop/AGENTS.md before acting; if it is missing, start at https://agentsop.ai.
   <!-- /agentsop:global -->
   ```

   The markers are today's `agentsop:global` markers, so the pointer replaces
   today's block in place and a checker can find it. The locations on record
   are those the conventions section names: Grok `~/.grok/AGENTS.md`,
   Claude Code `~/.claude/CLAUDE.md`, Codex `~/.codex/AGENTS.md`, and Cursor
   `~/.cursor/rules/agentsop.mdc` with `alwaysApply: true` in its
   frontmatter. `hook-dialects.mjs` in `agent-bot-identity`, the one table of
   per-harness user directories (`.claude`, also serving Devin CLI; `.codex`;
   `.cursor`; `.copilot`; `.windsurf` for Devin Desktop), records hook files,
   not instruction files, so no other instruction path is named here. Another
   harness's path is added to the table, with evidence, before tooling writes
   or checks a pointer there. With the pointer in place, a later session
   starts from the root file without being told to follow agentsop.ai.
3. **One source of truth.** A pointer is not a copy. Rule changes land in one
   file, the root file, and harness files rarely change. This removes the
   conflict with [ENG-0006](ENG-0006-agentic-primitives-governance.md)
   decision 1 (a fact stated in two agent files is a bug): ENG-0384's
   amendment of 2026-09-27 gives that rule no copy exception, and this design
   needs none.
4. **Writing the pointer is a one-time model action, an acknowledged
   exception.** ENG-0049 enforces steps that happen on every change. The
   pointer is written once per harness, not on every change, so a model may
   write it. Machinery bounds the exception: `agent-bot doctor` verifies that
   the root file exists and that each installed harness has the pointer, and
   prints how to fix each miss. Bootstrap (`managed-machine`) or `agent-bot`
   may also write the pointer deterministically wherever the table knows the
   path. Where a global file holds other content, the pointer is added and
   nothing else is changed or discarded.
5. **Precedence (proposed; a policy change).** A repository's `AGENTS.md`
   overrides both global layers, and the root file overrides a harness's
   global file. This keeps the earlier draft's position that repository
   instructions override global ones. Today's conventions block says the
   opposite for the harness file: "Rules already in this file still win when
   they conflict."
6. **"Write the pointer" replaces the rewrite rule.** Once this record is
   accepted and implemented, a separate PR replaces the conventions rule that
   asks each harness to rewrite its global file. This record does not change
   the conventions.

## Current state

Nothing in this record is implemented, and no root instruction file exists
anywhere by design today. The router at
[`75f45e2`](https://github.com/qwts/agentsop.ai/tree/75f45e2c23e6ce3317ba4efb56b388df7d7a2cd5)
says `config.toml` "is the only thing on the machine", and ENG-0355's
amendment says "Nothing else is local." The router's `start` zone checks for
`config.toml`, has the agent create it if missing, and then asks the agent to
"make memory for future references to agentsop.ai". No step names a root
instruction file or a pointer. At the `org.json` pin
[`agent-bot-identity@52f55b1`](https://github.com/qwts/agent-bot-identity/tree/52f55b171b41a2cf7ae80e5318246363729ccddf),
`sync-hooks.mjs` writes hook adapters, not instruction files, for the five
dialects above (with a `--check` mode); `doctor` checks hooks but no
instruction file; nothing writes a root file or a pointer. Neither do the
`managed-machine` setup scripts at `3276571`.

## Consequences

- **Router change.** The agentsop.ai `start` flow leads agents to the root
  file and to writing the pointer. Its step 3, a memory pointing at
  agentsop.ai, is the step the pointer replaces with a file. That change lands
  by reviewed PR in `qwts/agentsop.ai`, within ENG-0355's `llms.txt` size
  limits; this PR does not edit the site.
- **A second local file.** Accepted, this amends ENG-0355 decision 2 as
  amended ("Nothing else is local") to allow `~/.config/agent-sop/AGENTS.md`
  beside `config.toml`. ENG-0355 also rejected a copy installed on every
  machine as a drift source. Whether the root file is such a copy depends on
  how it is produced (see Open questions).
- `~` is the account's home. With one macOS account per harness
  ([ENG-0339](ENG-0339-os-account-determines-persona.md) decision 2), "one per
  machine" means one per account, as with `config.toml` today.
- Cost: an instruction-file path column for every active harness (today's
  table covers hook files for five dialects), the `doctor` check, and the
  optional pointer writer in `agent-bot-identity`, plus whatever installs the
  root file.
- ENG-0384 decision 4 keeps hook installation out of sessions. The pointer
  is not a hook; like decision 4's exception to ENG-0049, it is written once,
  and `doctor` makes a missing one visible.
- The ENG-0081 amendment of 2026-08-13 (decision 5) says no skill may
  introduce a required first tool call. The pointer is not a skill. It asks
  what today's block already asks, to read before acting, and no enforcement
  may depend on the agent doing it.
- [ENG-0178](ENG-0178-evidence-bound-harness-projections.md) is related by
  name only: it classifies harness-sync pull requests into repositories, not
  user-directory files. This record neither builds on it nor changes it.

## Alternatives

- **A generated stub in every harness's global file** (the previous draft of
  this record): rejected. It duplicates text: the same stub in about 21 files
  is what ENG-0006 decision 1 calls a bug, and it needed a copy exception that
  ENG-0384's amendment does not give.
- **Keep per-harness hand edits:** rejected. It is the remembered step, and
  the files already drift.
- **Symlink each harness file to the root file:** noted, not taken.
  Formats differ (Cursor needs an `.mdc` with frontmatter), and a harness
  that writes to its global file would write the root file. Tooling may still
  link where the format allows.

## Open questions

- Who owns the root file's content, and how does it get onto a machine:
  `managed-machine` bootstrap, `agent-bot`, or the agent on its first visit
  through the start flow? Is it itself generated from `qwts-agent-sop` at the
  `org.json` pin, which keeps one source, or written per machine, which makes
  it a copy that can drift?
- Whether the pointer carries identity (the slug, where credentials live) or
  defers to the root file and `agent-bot`. In the owner's account a harness is
  the delegate (ENG-0339 decision 3), so a fixed slug there would be wrong.
- How a cloud VM or shared box without the root file behaves. The pointer's
  fallback sends the agent to agentsop.ai; does the start flow then write the
  root file there, or does the session proceed from the router alone? A
  cloud job that sees only the clone has only the repository `AGENTS.md`.
- Who owns the per-harness instruction-path table: `agent-bot-identity`
  beside `hook-dialects.mjs`
  ([ENG-0128](ENG-0128-agent-bot-runtime-ownership.md) decision 1), or the
  `managed-machine` app catalog?

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
- [Org-wide agent conventions](../reference/agent-conventions.md): the rule
  decision 6 replaces
