---
name: author-cli-skill
description: Create or change a qwts CLI released for repeated use, including its bundled agent skill and release checks.
---

# Author a CLI skill

Use this skill when creating or changing a CLI that qwts releases for repeated
use. Read
[ENG-0055](../../docs/decisions/ENG-0055-every-cli-ships-its-agent-skill.md)
and [ENG-0064](../../docs/decisions/ENG-0064-cli-skill-command-family.md),
and follow the [CLI skill contract](../../docs/reference/cli-skill-contract.md)
and [CLI skill command contract](../../docs/reference/cli-skill-command-contract.md)
for the exact metadata, bundle, command, export, compatibility, and release-gate
requirements. Those decisions and
contracts remain normative.

1. **Choose the workflows.** Start from supported CLI behavior and the tasks an
   agent needs to complete. Keep one bundled skill per CLI; make its `SKILL.md`
   a short router and put each feature's task workflow in
   `references/<feature>.md`, so an agent loads one task at a time. Describe
   supported behavior rather than proposed commands.
2. **Make the feature map authoritative.** In CLI source, create one catalog
   mapping each feature ID to its reference file and summary. Link every
   cataloged feature from the router; have list, show, and packaging use the
   same catalog. The command contract defines the identifier and file rules.
3. **Expose and test the bundle.** Implement `skill`, `skill list`,
   `skill show <feature>`, and `skill path` as offline reads. Check that router
   and references print packaged bytes exactly, and that unknown features fail
   clearly. Implement `skill export --target-dir` for the complete bundle only:
   require an explicit target, refuse an existing destination by default, and
   permit replacement only through the contract's guarded path. Refuse known
   harness-discovery paths and preserve the target when a guarded export fails.
   Do not add install-to-harness behavior.
4. **Make the checks observable.** Test that catalog/router disagreement fails;
   printed router and feature bytes match their packaged files; unknown features
   and path traversal are rejected; and a refused or failed guarded export
   leaves its target unchanged. Exercise the documented replacement path and
   verify the release artifact contains the complete bundle beside its
   executable. Follow the command contract's broader export cases as applicable.
5. **Validate the release pair.** Add the contract metadata and a fail-closed
   gate that checks the packaged executable's version against the skill range,
   validates the bundle, and runs representative workflows against that
   packaged executable. At point of use, check the version of the executable
   actually being run; handle a missing, invalid, out-of-range, or pin-inconsistent
   version as the ENG-0055 mismatch case before any mutation.

Update the CLI implementation, bundle, and tests together. Name this authoring
skill in the CLI repository's `AGENTS.md`, and add the bundled skill's reviewed
revision to the [shared skills catalog](../README.md). A skill grants no
permission and does not widen the task.
