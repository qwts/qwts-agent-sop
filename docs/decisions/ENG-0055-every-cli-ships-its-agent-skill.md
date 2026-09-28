# ENG-0055: Every CLI we build ships its own agent skill, versioned with it

**Status:** Accepted
**Date:** 2026-09-28
**Issue:** qwts/qwts-agent-sop#55

## Context

Agents run the CLIs this organization builds. `--help` and man pages say
which commands and flags exist. They do not say which commands to chain,
which output is stable enough to parse, what an exit code means, or how to
recover from a failure. Each session works that out again, and what it
learns is lost when the session ends.

Two repositories already ship a skill with their tool, `managed-machine` and
`zsh-functions`, and the [skill catalog](../../skills/README.md) lists them at
pinned commits. Nothing requires a new CLI to do the same. Nothing says what
such a skill must declare so that an agent can tell whether it matches the
installed binary. A skill written for version 1 and followed against
version 2 gives confident, wrong instructions.

The owner's draft ADR, "Tiered CLI Skills for AI Agents" (2026-09-28),
proposed a three-tier loading model with an organization registry. This
record takes the parts that bind the CLIs we build. The loading model
already exists: the [ENG-0006](ENG-0006-agentic-primitives-governance.md)
amendment of 2026-09-23 and [ENG-0051](ENG-0051-one-root-instruction-file-harness-files-point-to-it.md).

## Decision

The principle: **the CLI enforces behavior, the skill explains how to use
it well, and release checks establish that the two agree.** The contract
details (metadata keys, version grammar, output conventions, gate checks)
are in the [CLI skill contract](../reference/cli-skill-contract.md).

1. **Scope.** Every CLI the organization releases for repeated use ships a
   skill: anything installed from a formula, package, or release. Scripts
   that are never installed are exempt. A package with several executables,
   or a shell-function collection, may ship one skill that covers them all.
2. **The skill ships with the release.** It lives at `skills/<cli>/` in the
   CLI's repository and changes in the same pull request as the behavior it
   describes. Each release bundles that directory from the same source
   revision, and the installed CLI prints the bundle's location and source
   commit through a read-only command. The catalog lists the skill at a
   pinned commit, and its body is never copied into this repository.
3. **The contract lives under `metadata`.** The Agent Skills reference
   validator rejects unknown top-level frontmatter fields. So the contract
   uses `qwts-` keys inside `metadata`: contract version, CLI names,
   supported version range, validated version, and side-effect class.
   Ownership comes from `.github/CODEOWNERS`, not from a frontmatter field,
   because a fact stated twice is a bug (ENG-0006 decision 1).
4. **The CLI owns what is mechanical; the skill owns judgment.** The CLI
   enforces its safeguards and explains itself through help and diagnostics
   ([ENG-0049](ENG-0049-every-change-steps-are-enforced-not-remembered.md)
   decision 3). The skill covers which task needs which workflow, how
   commands combine when that is not obvious, and how to recover from a
   failure. It may be thin and delegate to the version-matched help. Every
   mutating workflow says whether a retry is safe and how to inspect the
   result before retrying; a retry limit is not a retry-safety guarantee.
   Detail goes in `references/`. A skill is split by task, not by length.
5. **The CLI makes the skill checkable.** `<cli> --version` prints a bare
   SemVer version and has no side effects. There is no separate version
   command to declare. Documented machine-readable output stays backward
   compatible within a major version. Additions are allowed. Incompatible
   changes to field names, types, meanings, requiredness, or exit codes
   need a major release.
6. **Loading follows the ENG-0006 amendment.** A procedure names the skill.
   The CLI's own `AGENTS.md` names it for work on that CLI, and a consuming
   repository or SOP names it where that work uses the CLI. The skill is not
   installed into a harness. A rule every session needs goes in the root
   instruction file (ENG-0051). The agent uses the bundle of the installed
   release when that release is the approved pin. If the two disagree, it
   reports the inconsistency and handles it as a mismatch. It never fetches
   another revision because that revision's range happens to fit.
7. **Check the version at the point of use.** Before following a workflow,
   the agent runs `<cli> --version` against the executable it will actually
   use. It checks again after an upgrade, after an environment change, or
   when the command resolves to a different executable. This is not a
   required first call for the session: the ENG-0081 amendment of
   2026-08-13, decision 5, still holds.
8. **A mismatch voids the compatibility claim.** A mismatch is a version
   that is missing, unparseable, outside the range, or inconsistent with the
   pin. The agent may continue read-only inspection using the installed CLI's
   documentation. Before any mutation, it establishes the command's
   semantics and safeguards from documentation that matches the installed
   version. If it cannot, it stops that operation and reports the mismatch
   at that point. A skill grants no permission and does not widen the task.
9. **The release gate fails closed.** Following ENG-0049, the CLI's release
   workflow fails if the contract is missing or invalid, or if the packaged
   executable's version is outside the range. It also fails if the range
   extends past the next major version after `validated`, or if the bundle
   is absent. It runs the skill's representative workflows and output
   contract against the packaged executable. The range check shows that two
   declarations agree, the tests show that the workflows run, and the
   repository's review controls show that the change was reviewed.

ENG-0006 decision 3 still applies: a skill is reviewed as code, pinned,
free of secrets, and treated as a prompt-injection surface.

## Consequences

- **An authoring cost on every CLI.** A new CLI is not done until its skill,
  bundle, and gate exist. A major release is not done until its skill has
  been revalidated.
- **A skill can still be wrong within its range.** The gate's tests cover
  representative workflows, not every workflow. Review and, where they
  exist, ENG-0006 golden tasks cover the rest.
- **Tooling work.** A shared gate belongs in `qwts-agent-ci`. Until it
  exists, each CLI runs a local gate that implements the contract. A CLI
  that relies on review alone has a tracked exception issue and does not
  conform. After acceptance, a follow-up PR adds the contract to the
  catalog's "Adding a skill" steps.
- **Existing CLIs converge.** `managed-machine` and `zsh-functions` add the
  metadata, the bundle, and the gate. `agent-bot`'s skill stays as thin as
  ENG-0049 requires. Each gets an alignment issue, as ENG-0006 decision 6
  requires.
- **One version check per use, and a read-only bundle command on every
  CLI.** Both are cheap, and together they let an agent find a stale or
  mispaired skill.

## Alternatives

- **The draft ADR's three tiers with session-start metadata:** rejected. It
  puts every skill's name and description in every session, which the
  ENG-0006 amendment removed. Its Tier 1 duplicates the root instruction
  file.
- **A new organization registry:** rejected; the skill catalog is that
  registry, pinned by commit.
- **`--help` and man pages only:** rejected as the default and kept as the
  fallback. They document syntax, not how to run the workflows.
- **Contract fields at the top level of the frontmatter:** rejected; the
  Agent Skills validator refuses them.
- **Persisting learned workflows automatically:** out of scope. A learned
  workflow reaches a skill through a reviewed pull request, as any change
  does.
- **Third-party CLIs and repository overlays:** out of scope; a later record
  if the need appears.

## References

- [CLI skill contract](../reference/cli-skill-contract.md)
- [ENG-0006](ENG-0006-agentic-primitives-governance.md) and its amendment of
  2026-09-23, [ENG-0049](ENG-0049-every-change-steps-are-enforced-not-remembered.md),
  [ENG-0051](ENG-0051-one-root-instruction-file-harness-files-point-to-it.md),
  [ENG-0081](ENG-0081-transcript-bound-agent-execution-identities.md)
- [Skill catalog](../../skills/README.md) ·
  [Agent Skills specification](https://agentskills.io/specification) ·
  [Semantic Versioning 2.0.0](https://semver.org/)
