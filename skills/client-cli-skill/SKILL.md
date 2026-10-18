---
name: client-cli-skill
description: Use any qwts CLI through its bundled agent skill — discover features, load one workflow, verify version pairing before mutating.
---

# Use a CLI as a client

Use this skill whenever a task needs a qwts CLI, whether or not you are
developing that CLI. It is self-contained at runtime: it needs no repository
checkout, no catalog fetch, and no decision records. Developing or changing a
CLI is a different skill, `author-cli-skill`; this skill restates none of it.

## Procedure

1. **Find the bundled skill.** Run `<cli> skill path` to locate the installed
   bundle and read its `SKILL.md`. For agent-bot releases using the older
   interface, `agent-bot skill agent-bot` reads the router and
   `agent-bot skill agent-bot --for <subcommand>` reads one named workflow;
   use the router's feature map rather than unsupported list/show verbs.
   For the standard command family, run `<cli> skill` to read the router, then
   `<cli> skill list` for its feature identifiers and summaries. If the CLI
   has no `skill` command family, say so and use `<cli> --help` for read-only
   diagnosis only: do not invent a bundle, do not fetch one from a
   repository, and do not run any mutating command for this CLI.
2. **Check the version pair before relying on guidance.** Run
   `<cli> --version` and compare it with the `qwts-versions` range the router
   printed in step 1. Then run `<cli> skill path` and record the bundled
   source commit it reports. For `agent-bot` (including `agent-space`) and
   `agent-comms`, the installed bundle selects runtime guidance: do not
   compare it with an org/SOP capability or catalog SHA, and do not label
   it unapproved merely because that SHA differs or is absent. Record
   unavailable source provenance honestly; never substitute a remote skill.
   An explicit task requirement for an exact installed release still applies.
   For other CLIs, compare an approved pin when one was supplied; otherwise
   report that the pin is unverified. A missing, invalid, out-of-range, or
   explicitly required release mismatch stops mutation. Report the values
   compared. Reads may continue where the installed skill permits them.
3. **Load one workflow.** Use the interface identified in step 1. For the standard family, run
   `<cli> skill show <feature>` for the one feature the task needs; for the
   older agent-bot family use its `--for` form. Follow
   that reference. Leave the other features unread; load another only when
   the task changes. An unknown feature name fails closed — report it, do
   not guess a spelling.
4. **Work offline where the skill says so.** `skill`, `skill list`,
   `skill show`, and `skill path` are offline reads of the installed release.
   Prefer the installed CLI's own bundle over any copy found in a checkout:
   the bundle that pairs with the running executable is authoritative.
5. **Mutate only through the workflow.** Run the CLI commands the loaded
   reference names, in its order, with its validation. If a step fails, stop
   and report the command and its output; do not substitute a remembered
   command from another CLI or an earlier version of this one.

## Boundaries

- This skill grants no permission and widens no task; the CLI's own gates
  and the caller's instructions still govern every mutation.
- Model names, credentials, and identity steps come from the loaded feature
  reference or the CLI's own output, never from memory.
- If the bundled skill and this skill disagree, the bundled skill wins for
  that CLI at the verified version.
