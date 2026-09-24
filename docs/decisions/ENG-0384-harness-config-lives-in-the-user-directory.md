# ENG-0384: Harness config lives in the user directory

**Status:** Proposed
**Date:** 2026-09-24
**Issue:** qwts/agent-sop#384

## Context

The repository baseline requires `.codex/` and `.claude/settings.json` in every
repository. [ENG-0128](ENG-0128-agent-bot-runtime-ownership.md) allows this
repository to keep governed harness adapters such as `.codex/hooks.json`.
[ENG-0006](ENG-0006-agentic-primitives-governance.md) treats hooks and vendor
settings files as primitives that live in the repo as thin adapters.

Each harness reads its rules, hooks, and other primitives from its user
directory, `~/.<harness>` or `~/.config/<harness>`. Copies committed into
every repository drift, and a session that rechecks them spends context on
every turn. The fleet request is
[qwts/qwts-agent-sop#25](https://github.com/qwts/qwts-agent-sop/issues/25).

## Decision

1. **The user directory is the home.** Harness rules, hooks, and the other
   harness primitives live in that harness's user directory. `AGENTS.md`
   stays the one canonical vendor-neutral file in the repository.
2. **A repository file is an exception.** It exists only when a named harness
   cannot read that primitive from the user directory. This record lists no
   such exception.
3. **A repository hook is additional.** It runs beside the user-level hook
   for the same event. It does not replace that hook, disable it, or win
   when the two disagree. Where a harness gives the project file the
   conflict, the repository file does not subscribe to an event the user
   file already handles.
4. **Install once, with the catalog.** User-level hooks for a harness are
   installed when that harness is added to the managed-machine app catalog
   and machine setup runs. An agent session does not discover, check, or
   install hooks.
5. **Skills stay catalog bodies.** A skill is read when a procedure names it.
   This record does not move those bodies and does not copy them into a
   harness directory.

This supersedes the harness-in-repo clauses of ENG-0128 (the permission to
keep governed adapters such as `.codex/hooks.json` in the repository) and of
ENG-0006 (hooks and vendor settings files as in-repo thin adapters). Those
records are not rewritten. Their other clauses stand.

## Consequences

- `docs/sop/repo-baseline-files.md` drops `.codex/` and
  `.claude/settings.json` from the required set.
- The hook-composition survey remains a historical audit. It is not the list
  of managed copies.
- A harness that is not in the app catalog has no hooks until it is added
  and machine setup runs. That delay is accepted. A per-session reminder is
  not.
- A cloud job that only sees the clone does not see the user directory.
  Until a harness is named here as an exception, that job runs without these
  hooks. Recording the exception is a later amendment, not a reason to put
  the files back in every repository.
- Cursor gives the project file the win when a user hook and a project hook
  disagree. Decision 3 exists so a repository file cannot use that precedence
  to replace the user hook.
