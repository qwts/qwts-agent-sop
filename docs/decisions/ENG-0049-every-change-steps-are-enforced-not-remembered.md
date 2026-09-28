# ENG-0049: Steps on every change are enforced, not remembered

**Status:** Proposed
**Date:** 2026-09-27
**Issue:** qwts/qwts-agent-sop#49

## Context

Every session is told the same things before it publishes: sign the commits,
open the pull request as the bot
([ENG-0016](ENG-0016-agent-pr-bot-identity.md)), add the `Agent-Identity`
trailer ([ENG-0081](ENG-0081-transcript-bound-agent-execution-identities.md)
decision 6). Models forget. The steps sit in a long `SKILL.md` that an agent
must find and then read before it acts, and that does not reliably happen.

A run on 2026-09-27 shows it. The agent followed `agentsop.ai` to
`governance/agents.json` in the org repository, which names
`qwts-grok-agent`. Nothing on that path said how to obtain that identity on
the machine. `agent-bot` was not installed, so `agent-bot signed-commit` was
unavailable, and the `Agent-Identity` trailer was missed. On the router,
identity appears only in the `comms` zone, after the agent has started work.

## Decision

The principle: a step that happens on every change is enforced
deterministically, not remembered. Instructions are kept for judgment.

1. **Identity is its own router zone, right after `start`.** Identity is a
   prerequisite for every other rule, so it is read first. The zone names only
   the role. Today it resolves through the existing tool-keyed `agent-bot`
   capability in `org.json`, which pins the implementation repository
   (`qwts/agent-bot-identity`) at a commit. The site still names no
   repository, and replacing the implementation stays an `org.json` change, as
   [ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md)
   decisions 3 and 4 require; accepted, this adds `identity` to that record's
   zone list (amendment of 2026-09-16). The zone says how an agent obtains its
   identity: run `agent-bot`, which takes the App's credentials from the
   reviewed `pass-cli` store (ENG-0016 amendment of 2026-08-13, decision 6).
   A key file under `~/.config/<slug>` is only a stopgap on a machine where
   `agent-bot` is not set up yet, such as the shared box this record was
   written on. It is not the sanctioned location, and nothing here points
   agents to it.
2. **Signing and the trailer are enforced in layers.**
   - Server: `required_signatures` on the default branch of every governed
     repository, through the baseline. Beside it, a required status check from
     `qwts-agent-ci` blocks merge of a pull request containing a commit
     authored by a rostered App (`<app-slug>[bot]`) that lacks a well-formed
     `Agent-Identity` trailer. The check reads the commit body, not git's
     trailer parser, and cannot see the private registry (ENG-0081 decision
     6), so it checks presence and form; resolving the Agent ID stays with the
     local guard. ENG-0081 binds only bot-attributed agent commits, so a
     commit a human account authors needs no trailer and passes.
   - Machine: bootstrap (`managed-machine`) configures each bot account so a
     plain `git commit` is signed as that App and carries the trailer.
   - CLI fallback: where bootstrap did not run (cloud VMs, shared boxes),
     `agent-bot` detects missing signing and either fixes it or blocks with
     the fix, and every commit it creates carries the trailer.
3. **`agent-bot` teaches itself by progressive disclosure, as `git` and `gh`
   do.** Bare `agent-bot` prints about ten lines: who you are (harness and App
   slug), setup health, and the next few commands. `agent-bot help <topic>`
   goes one level deeper. Every command's output ends with the next step.
   `agent-bot doctor` checks that credentials resolve from `pass-cli`,
   signing, runtime version, and trailer configuration, and prints the fix
   for each failure. It stays advisory. The `agent-bot` `SKILL.md` shrinks to
   "run `agent-bot`".
4. **The SOP stays the rule; the capability makes it true.** This repository
   says what must hold (bot identity, signed commits, the trailer).
   `agent-bot-identity` is the implementation that makes it hold
   ([ENG-0128](ENG-0128-agent-bot-runtime-ownership.md)). The template and
   implementation split is unchanged.

Bare `agent-bot` is orientation, not a clock-in. The ENG-0081 amendment of
2026-08-13 (decision 5) still holds that no skill may introduce a required
first tool call, and no layer in decision 2 depends on the agent running it.

## Current state

Nothing in this record is implemented yet. At the `org.json` pin
([`52f55b1`](https://github.com/qwts/agent-bot-identity/tree/52f55b171b41a2cf7ae80e5318246363729ccddf)),
bare `agent-bot` prints a static list of about twenty commands, there is no
`help <topic>`, and `doctor` checks installation and identity readiness but
not signing or the trailer. Its `prepare-commit-msg` hook adds the trailer
only in a worktree with a registered identity, and `qwts-agent-ci` has no
trailer check. The [router](https://github.com/qwts/agentsop.ai/tree/75f45e2c23e6ce3317ba4efb56b388df7d7a2cd5)
has no identity zone. This repository's default-branch ruleset already
requires signatures; the [baseline SOP](../sop/repo-baseline-files.md) does
not state it for every governed repository.

## Consequences

- Shorter skills and fewer SOP misses: the every-change steps stop depending
  on what the model read.
- Cost: CLI work in `agent-bot-identity`, bootstrap work in `managed-machine`,
  a trailer check in `qwts-agent-ci`, a router change in `qwts/agentsop.ai`,
  and baseline and `org.json` entries in `qwts-agent-org`. Each lands by
  reviewed PR in its own repository.
- The server layer rejects an unsigned or untrailered commit late, at merge.
  The machine and CLI layers exist so that is rare.
- A bot commit made outside `agent-bot` and bootstrap, such as one created
  through the Git Data API, fails the trailer check until it is recommitted
  with one.
- The CLI fallback acts inside a session, which
  [ENG-0384](ENG-0384-harness-config-lives-in-the-user-directory.md) decision
  4 avoids for hooks. It runs only inside `agent-bot` commands the agent
  already runs, and only where bootstrap did not; it is not a session step.
- The router gains a zone, which grows the site's surface; ENG-0355's size
  limits for `llms.txt` still apply.

## Alternatives

- **Keep identity under `comms`:** rejected. It is read too late, after the
  agent has already committed.
- **A bigger `SKILL.md`:** rejected. It must be found and read, which is the
  step that fails; it is not deterministic.

## Open questions

- Keying `org.json` capabilities by role (`identity`) instead of tool name
  (`agent-bot`), so the zone and its entry share a name.
- Naming: `agent-bot-*` versus the `agent-`/`qwts-agent-` pattern, possibly an
  `agent-identity` template with a `qwts-agent-identity` implementation.
- A dev zone pointing at the existing `qwts-agent-sdlc` rather than a new
  `agent-bot-dev` repository.
- Whether the implementation repositories move to the true-line organization.

## References

- [ENG-0016](ENG-0016-agent-pr-bot-identity.md),
  [ENG-0081](ENG-0081-transcript-bound-agent-execution-identities.md),
  [ENG-0128](ENG-0128-agent-bot-runtime-ownership.md),
  [ENG-0282](ENG-0282-immutable-pins-recorded-selection-no-aligner.md),
  [ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md),
  [ENG-0384](ENG-0384-harness-config-lives-in-the-user-directory.md)
- [Agent bot identity governance](../reference/agent-bot-identity.md)
- [Branch, PR, and review SOP](../sop/branch-pr-review.md) — the signing rule
  this record enforces
