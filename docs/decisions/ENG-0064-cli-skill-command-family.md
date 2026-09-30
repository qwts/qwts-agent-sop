# ENG-0064: CLIs expose their bundled skill through a `skill` command family

**Status:** Proposed
**Date:** 2026-09-30
**Issue:** qwts/qwts-agent-sop#64

## Context

[ENG-0055](ENG-0055-every-cli-ships-its-agent-skill.md) requires every CLI
the organization releases to ship `skills/<cli>/` with each release, and a
read-only command that prints the bundle's location and source commit. It
does not say what that command looks like, how an agent reaches the workflow
for one task without reading the whole skill, or how an agent building a CLI
learns to produce all of this. Each CLI would answer differently, and an
agent that has used one CLI's skill could not rely on the next.

There are two consumers. An agent **building** a CLI needs an authoring skill
that explains how to design, package, expose, and test the CLI's skill. An
agent **using** the CLI needs the workflow for its current task, from the
bundle of the installed release. Keeping them apart means users of a CLI
never receive the much larger authoring instructions.

Loading is already settled: the ENG-0006 amendment of 2026-09-23 and ENG-0055
decision 6 say that skills are not installed into a harness, because a
harness injects every installed name and description on every turn. This
record keeps that rule and amends neither record.

## Decision

The contract details (commands, output, export guard, acceptance criteria)
are in the [CLI skill command contract](../reference/cli-skill-command-contract.md).

1. **One skill per CLI, one reference per feature.** The bundle stays at
   `skills/<cli>/` (ENG-0055 decision 2). `SKILL.md` is a thin router: when
   to use the CLI, the version check at the point of use (ENG-0055 decision
   7), and a link to each feature's reference. Each feature's workflow lives
   in `references/<feature>.md`, split by task (ENG-0055 decision 4).
2. **A `skill` command family reads the bundle.** `<cli> skill` prints the
   router; `skill list`, `skill show <feature>`, and `skill path` list
   features, print one feature's reference, and report the bundle directory
   and source commit. `skill path` is the ENG-0055 bundle-location command.
   Every operation is a verb, so a feature identifier never collides with a
   subcommand. Reads are offline, byte-exact, and change nothing.
3. **One catalog drives every command.** The CLI source maps each feature
   identifier to its reference file and summary. A test fails when the
   router's feature links and the catalog disagree
   ([ENG-0049](ENG-0049-every-change-steps-are-enforced-not-remembered.md)).
4. **Export goes to an explicit directory, never into harness discovery.**
   `skill export --target-dir` copies the complete bundle, which is how an
   agent obtains supporting files a stdout read does not provide. It refuses
   known harness discovery paths, and its replacement guard is code, not
   instructions. No read or export executes bundled code or fetches from the
   network. A skill grants no permission (ENG-0055 decision 8).
5. **Versions follow ENG-0055.** The `qwts-*` metadata, version grammar,
   point-of-use check, and mismatch handling are ENG-0055's and the
   [CLI skill contract](../reference/cli-skill-contract.md)'s. An exported copy
   can outlive its binary; on a mismatch the agent reads from the current
   binary instead. No command updates an exported copy as a side effect.
6. **An authoring skill lives in the shared skills home.** It sits under
   [`skills/`](../../skills/README.md) (ENG-0004), is cataloged, owned in
   `.github/CODEOWNERS`, kept within the `perDoc` budget, and never installed.
   The procedure for creating or changing a CLI names it, as does a CLI
   repository's `AGENTS.md`. It teaches the workflow split, the router and
   references, the catalog, the `skill` commands, the export guard, and the
   tests, and it links to ENG-0055 and this record rather than restating them.
   Where a rule needs a control, it requires the control.

## Consequences

- **Every CLI gets the same interface.** An agent that has used one CLI's
  `skill` commands can use any CLI's, and the ENG-0055 bundle-location
  requirement has one concrete shape.
- **An agent loads one workflow, not the whole skill.**
- **Supporting files need an export step.** A stdout read does not make
  `references/` or `scripts/` available as files. Self-contained references
  are the default; a workflow that needs a file says so first.
- **More surface in every CLI.** The export guard, catalog test, and
  harness-path refusal cost code in each CLI until `qwts-agent-ci` provides
  shared pieces.
- **The harness-path list needs upkeep** whenever a harness adds a discovery
  location.

## Alternatives

- **Install into harness discovery paths (`--scope`, `--harness`):**
  rejected. It contradicts ENG-0055 decision 6 and the ENG-0006 amendment.
- **Several skills per CLI:** rejected. It contradicts ENG-0055 decision 2
  and multiplies release-gate work; references already split by task.
- **`<cli> skill <feature>` with reserved words:** rejected. Each new
  subcommand would break any feature with that name.
- **Model-backed runtime decisions in this record:** proposed separately
  (qwts/qwts-agent-sop#65); they send state to a provider and need their own
  review.

## References

- [CLI skill command contract](../reference/cli-skill-command-contract.md) ·
  [CLI skill contract](../reference/cli-skill-contract.md)
- [ENG-0055](ENG-0055-every-cli-ships-its-agent-skill.md) ·
  [ENG-0006](ENG-0006-agentic-primitives-governance.md) and its amendment of
  2026-09-23 · [ENG-0049](ENG-0049-every-change-steps-are-enforced-not-remembered.md) ·
  [ENG-0004](ENG-0004-centralize-shared-cicd.md)
- [Skill catalog](../../skills/README.md) ·
  [Agent Skills specification](https://agentskills.io/specification)
