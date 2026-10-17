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

1. **Find the bundled skill.** Run `<cli> skill list`. If the CLI has no
   `skill` command family, say so and use `<cli> --help`; do not invent a
   bundle or fetch one from a repository.
2. **Check the version pair before relying on guidance.** Run
   `<cli> --version` and compare it with the range the bundled skill prints
   (`<cli> skill` shows it). A missing, invalid, out-of-range, or
   pin-inconsistent version is a mismatch: stop before any mutation, report
   the two versions, and let the mismatch be resolved. Reads may continue
   only where the skill itself permits them.
3. **Load one workflow.** From the list in step 1, run
   `<cli> skill show <feature>` for the one feature the task needs and follow
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
