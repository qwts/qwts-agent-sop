# ENG-0055: Every CLI we build ships its own agent skill, versioned with it

**Status:** Proposed
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

1. **Every CLI built in the organization ships a skill in its own
   repository,** at `skills/<cli>/SKILL.md`. The skill is released with the
   CLI and changes in the same pull request as the behavior it describes.
   The skill catalog lists it at a pinned commit, as it lists every shared
   skill. Its body is never copied into this repository.
2. **The skill's frontmatter carries a contract** beside `name` and
   `description`:

   | Field | Holds |
   | --- | --- |
   | `versions` | The supported range, bounded (`>=1.4 <2`), never open-ended |
   | `version-command` | The least-invasive command that prints the version, normally `<cli> --version` |
   | `validated` | The version the workflows and examples were last run against |
   | `side-effects` | The strongest class any workflow reaches: `read-only`, `local-write`, `remote-write`, or `destructive` |
   | `owner` | A named owner, matching `.github/CODEOWNERS` |

3. **The body covers workflows, not reference.** It covers preferred
   end-to-end workflows, output handling (the machine-readable mode, exit
   codes, and which human-readable output is unstable), failure signatures
   with retry limits and recovery steps, and known pitfalls. Each workflow is
   labeled with its side-effect class. The body does not contain a command
   catalog, a copy of the man page, secrets, or live identifiers. It stays
   within the `perDoc` token budget; a skill that needs more is two skills.
4. **The CLI makes the skill checkable.** It prints a parseable version from
   `--version` with no side effects. Any output a workflow parses has a
   machine-readable mode (JSON, for example), and that mode's shape changes
   only with the major version.
5. **Loading is the ENG-0006 amendment's rule.** A procedure names the skill.
   The CLI's own `AGENTS.md` names it for work on that CLI, and a consuming
   repository or SOP names it where that work uses the CLI. The skill is not
   installed into a harness. A rule every session needs goes in the root
   instruction file (ENG-0051), not in an always-loaded skill.
6. **Version check before use.** Before following a workflow, the agent runs
   `version-command`. If the version is missing, cannot be parsed, or is
   outside `versions`, the skill counts only as a hint. The agent works from
   `--help` and man pages, and it reports the mismatch in its result.
7. **Enforced at release, not remembered.** Following
   [ENG-0049](ENG-0049-every-change-steps-are-enforced-not-remembered.md), the
   CLI's release workflow fails if the version it releases is outside the
   skill's `versions`. A breaking release therefore cannot ship until the
   skill has been reviewed.

A skill is executable advice. ENG-0006 decision 3 already applies to it:
it is reviewed as code, pinned, contains no secrets, and is a
prompt-injection surface. A skill explains a command. It grants no
permission and does not widen the task.

## Consequences

- **An authoring cost on every CLI.** A new CLI is not done until its skill
  exists, and a breaking release is not done until the skill is updated.
  That cost is the point, and it is still a cost.
- **A skill can be wrong inside its declared range.** The version check
  catches stale skills, not incorrect ones. Correctness still depends on
  review and, where a repository has them, ENG-0006 golden tasks.
- **Tooling work.** The release check in decision 7 is shared CI work for
  `qwts-agent-ci`. Until it exists, each CLI checks its own release, or
  review catches the problem. After acceptance, a follow-up PR adds the
  frontmatter contract to the catalog's "Adding a skill" steps.
- **Existing CLIs converge.** `managed-machine` and `zsh-functions` already
  ship skills. They gain the frontmatter contract and the release check.
  `agent-bot` gains a skill scoped to its CLI. Each gets an alignment
  issue, as ENG-0006 decision 6 requires.
- **One version-command run per load.** It is cheap, and it prevents
  following a stale skill.

## Alternatives

- **The draft ADR's three tiers with session-start metadata:** rejected. It
  puts every skill's name and description in every session, which the
  ENG-0006 amendment removed. Its Tier 1 duplicates the root instruction
  file.
- **A new organization registry:** rejected; the skill catalog is that
  registry, pinned by commit.
- **`--help` and man pages only:** rejected as the default and kept as the
  fallback. They document syntax, not how to run the workflows.
- **Persisting learned workflows automatically:** out of scope. A learned
  workflow reaches a skill through a reviewed pull request, as any change
  does.
- **Third-party CLIs and repository overlays:** out of scope; a later record
  if the need appears.

## References

- [ENG-0006](ENG-0006-agentic-primitives-governance.md) and its amendment of
  2026-09-23, [ENG-0049](ENG-0049-every-change-steps-are-enforced-not-remembered.md),
  [ENG-0051](ENG-0051-one-root-instruction-file-harness-files-point-to-it.md)
- [Skill catalog](../../skills/README.md)
