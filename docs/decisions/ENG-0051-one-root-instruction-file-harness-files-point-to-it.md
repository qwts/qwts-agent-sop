# ENG-0051: One generated root instruction file per account; harness global files point to it

**Status:** Accepted
**Date:** 2026-09-27
**Issue:** qwts/qwts-agent-sop#51

## Context

Each harness reads a global instruction file from its user directory
([ENG-0384](ENG-0384-harness-config-lives-in-the-user-directory.md)
decision 1). Today every session keeps that file current by hand: the
[org-wide agent conventions](../reference/agent-conventions.md), section
"Global instruction file" (#19, #20), tell the agent to make the file match an
`agentsop:global` block. The section names paths for four harnesses: Grok,
Claude Code, Codex, and Cursor. The org roster
([`governance/agents.json`](https://github.com/qwts/qwts-agent-org/blob/62517e731de2d98def5fdcab8d4179adf5773187/governance/agents.json))
lists 21 active harness Apps.

This is the kind of step
[ENG-0049](ENG-0049-every-change-steps-are-enforced-not-remembered.md) says is
enforced, not remembered: it happens in every session, a model performs it,
and nothing checks the result. The files drift and differ per harness.

## Decision

This record applies ENG-0049's principle. Decision 5 is an acknowledged
exception to it.

1. **One root instruction file per account.** It sits beside `config.toml` in
   the organization's own directory, since no cross-harness standard exists:
   on Linux and macOS `$XDG_CONFIG_HOME/agent-sop/` when that is set, else
   `~/.config/agent-sop/`; on Windows `%USERPROFILE%\.config\agent-sop\`.
   The router names one literal path, `~/.config/agent-sop/config.toml`; on
   Windows `~` resolves to `%USERPROFILE%`, so that path holds unchanged,
   while `%APPDATA%` would add a second convention. Like `config.toml`, the
   file holds no secrets. `~` is the OS account's home, and with one account
   per harness ([ENG-0339](ENG-0339-os-account-determines-persona.md)
   decision 2) there is one file per account, not per machine.
2. **`agent-bot` generates it from one pinned source.** It builds the content
   from `qwts-agent-sop` at the commit `org.json` pins and writes the file
   where it is missing, or prints exactly what the agent should write. The
   file carries a stamp naming that commit. Every `agent-bot` run compares
   the stamp with the current pin and regenerates the file when they differ,
   so updates arrive deterministically and no copy drifts silently;
   `agent-bot doctor` reports a stale or missing file. There may be many
   copies, but there is one source of truth. This works on cloud VMs, shared
   boxes, and single-account machines. `agent-bot`, and every org CLI from
   now on, runs on Linux, macOS, and Windows.
3. **Each harness's global file holds a pointer, not a copy.** After an agent
   has read the root file once, it writes a pointer of two or three lines
   into its own harness's global instruction location (`agent-bot` writes the
   resolved path), for example:

   ```text
   <!-- agentsop:global -->
   Read ~/.config/agent-sop/AGENTS.md before acting; if it is missing, start at https://agentsop.ai.
   <!-- /agentsop:global -->
   ```

   These are today's `agentsop:global` markers, so the pointer replaces
   today's block in place and a checker can find it. The locations on record
   are those the conventions name: Grok `~/.grok/AGENTS.md`, Claude Code
   `~/.claude/CLAUDE.md`, Codex `~/.codex/AGENTS.md`, and Cursor
   `~/.cursor/rules/agentsop.mdc` with `alwaysApply: true` in its
   frontmatter. `hook-dialects.mjs` in `agent-bot-identity` lists per-harness
   user directories (`.claude`, `.codex`, `.cursor`, `.copilot`, `.windsurf`)
   for hook files only, so no other instruction path is named here; one is
   added to that table, with evidence, before tooling uses it.
4. **Rules change in one place.** Pointers are not copies, and every root
   file is generated from the pin, never edited by hand. That keeps
   [ENG-0006](ENG-0006-agentic-primitives-governance.md) decision 1 (a fact
   stated in two agent files is a bug): a stamped copy is build output, not a
   second statement.
5. **Writing the pointer is a one-time model action, an acknowledged
   exception.** ENG-0049 enforces steps that happen on every change; the
   pointer is written once per harness. `agent-bot doctor` verifies each
   installed harness's pointer and prints how to fix a miss, and `agent-bot`
   may write it wherever the table knows the path. Other content in a global
   file is left as it is.
6. **Precedence: the SOP's mandatory sections are the floor; proximity wins
   above it (a policy change).** Nothing overrides a section a shared SOP
   marks mandatory. A non-mandatory section stays overridable by a recorded
   repository delta under [ENG-0008](ENG-0008-shared-sop-inheritance.md)
   decision 3, which this record preserves, not amends. Within what the SOP
   leaves open, repository `AGENTS.md` overrides the root file, which
   overrides the harness pointer. A repository states a rule only when it is
   critical to that repository and not repeated across repositories; a
   repeated rule is promoted (ENG-0008 decision 4). So "open pull requests as
   the owner" in a repository `AGENTS.md` does not escape the bot-identity
   rule ([ENG-0016](ENG-0016-agent-pr-bot-identity.md)): it would make the
   merge bar's human review, a mandatory section, unsatisfiable. Today's
   conventions block says instead that rules already in the harness file
   "still win".
7. **"Write the pointer" replaces the rewrite rule.** Once this record is
   accepted and implemented, a separate PR replaces the conventions rule that
   asks each harness to rewrite its global file. This record does not change
   the conventions. Their "call `take_inbox`" rule is out of scope here: push
   delivery replaces it separately.

## Current state

Nothing here is implemented. The router at
[`75f45e2`](https://github.com/qwts/agentsop.ai/tree/75f45e2c23e6ce3317ba4efb56b388df7d7a2cd5)
says `config.toml` "is the only thing on the machine", and
[ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md)'s
amendment of 2026-09-16 says "Nothing else is local." Its `start` zone has
the agent create `config.toml`, then "make memory" of agentsop.ai. At the
`org.json` pin
[`agent-bot-identity@52f55b1`](https://github.com/qwts/agent-bot-identity/tree/52f55b171b41a2cf7ae80e5318246363729ccddf),
`agent-bot` has no command that generates, stamps, or refreshes a root file;
`sync-hooks.mjs` writes hooks, not instruction files; `doctor` checks no
instruction file; the daemon supervisor supports only Linux and macOS. The
`managed-machine` setup scripts at `3276571` write neither file.

## Consequences

- **Router change.** The `start` flow leads agents to the root file and the
  pointer, replacing "make memory", and names the XDG and Windows paths. That
  lands by PR in `qwts/agentsop.ai`.
- **A second local file.** Accepted, this amends ENG-0355 decision 2 as
  amended ("Nothing else is local"). ENG-0355 rejected a copy on every machine
  as a drift source; here every copy is stamped and refreshed from the pin.
- **Work lands in `agent-bot-identity`:** the generator and stamp check, the
  `doctor` checks, per-OS path resolution, the pointer writer, and the
  instruction-path table. `managed-machine` may call `agent-bot` but is not
  required.
- ENG-0384 decision 4 keeps hook installation out of sessions. The pointer
  is not a hook; it is written once, and `doctor` reports a missing one.
- The ENG-0081 amendment of 2026-08-13 (decision 5) bars a skill from adding
  a required first tool call. The pointer is not a skill, and no enforcement
  may depend on it.
- [ENG-0178](ENG-0178-evidence-bound-harness-projections.md) is related by
  name only: it classifies harness-sync pull requests, not user-directory
  files.

## Alternatives

- **A generated stub in every harness's global file** (the previous draft):
  rejected. It put the rules in about 21 files in several formats, which
  ENG-0006 decision 1 calls a bug.
- **`managed-machine` owns the root file:** rejected; it is not available on
  cloud VMs. A machine-wide path outside the account homes would likewise
  need a privileged installer.
- **Keep per-harness hand edits:** rejected. It is the remembered step, and
  the files already drift.
- **Symlink each harness file to the root file:** noted, not taken.
  Formats differ (Cursor needs `.mdc` frontmatter), and a harness that writes
  to its global file would write the root file. Tooling may still link where
  the format allows.

## Open questions

- Whether the pointer carries identity (the slug, where credentials live) or
  defers to the root file and `agent-bot`. In the owner's account a harness is
  the delegate (ENG-0339 decision 3), so a fixed slug there would be wrong.
- What a session does without `agent-bot`: the pointer's fallback sends it to
  agentsop.ai, and a cloud job that sees only the clone has only the
  repository `AGENTS.md`.
- Whether the instruction-path table lives beside `hook-dialects.mjs`
  ([ENG-0128](ENG-0128-agent-bot-runtime-ownership.md)) or in the
  `managed-machine` app catalog.

## References

- [ENG-0049](ENG-0049-every-change-steps-are-enforced-not-remembered.md)
  (Accepted, merged in #50)
- [ENG-0006](ENG-0006-agentic-primitives-governance.md),
  [ENG-0008](ENG-0008-shared-sop-inheritance.md),
  [ENG-0016](ENG-0016-agent-pr-bot-identity.md),
  [ENG-0081](ENG-0081-transcript-bound-agent-execution-identities.md),
  [ENG-0128](ENG-0128-agent-bot-runtime-ownership.md),
  [ENG-0178](ENG-0178-evidence-bound-harness-projections.md),
  [ENG-0339](ENG-0339-os-account-determines-persona.md),
  [ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md),
  [ENG-0384](ENG-0384-harness-config-lives-in-the-user-directory.md)
- [Org-wide agent conventions](../reference/agent-conventions.md): the
  "Global instruction file" rule that decision 7 replaces.
