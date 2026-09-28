# CLI skill contract

The machine-checkable contract behind
[ENG-0055](../decisions/ENG-0055-every-cli-ships-its-agent-skill.md): what a
CLI's `SKILL.md` declares, how versions are compared, what the CLI's output
promises, and what the release gate checks. This is contract version `1`. A
change that an existing skill would fail raises the contract version.

## Frontmatter

The standard `name` and `description` fields stay at the top level. The
contract goes under `metadata`, whose values are strings in the
[Agent Skills specification](https://agentskills.io/specification):

```yaml
---
name: example-cli
description: Run Example CLI workflows and recover from their failures.
metadata:
  qwts-contract: "1"
  qwts-cli: "example-cli"
  qwts-versions: ">=1.4.0 <2.0.0"
  qwts-validated: "1.4.2"
  qwts-side-effects: "remote-write"
---
```

| Key | Holds |
| --- | --- |
| `qwts-contract` | The contract version this skill follows |
| `qwts-cli` | The executable name, or several separated by commas |
| `qwts-versions` | The supported range, in the grammar below |
| `qwts-validated` | The version the release gate last passed with this skill revision |
| `qwts-side-effects` | The strongest class any workflow reaches |

Side-effect classes, from weakest to strongest: `read-only` (no state
changes); `local-write` (changes files or state on this machine only);
`remote-write` (changes state on a remote service); `destructive` (deletes or
overwrites something that cannot be recovered). Each workflow in the body
also carries its own class.

## Version grammar

- **Versions** are bare [SemVer 2.0.0](https://semver.org/) with no `v`
  prefix. `<cli> --version` prints one, alone or as the last
  whitespace-separated token of its first line.
- **Ranges** use the range syntax of the npm `semver` package, evaluated
  with its default rules. A prerelease satisfies a range only when the range
  names that prerelease's `major.minor.patch`.
- **A range is bounded.** It has an upper bound, and that bound is no higher
  than the next major version after `qwts-validated`. During `0.x`, where
  SemVer promises no stable API, the bound is the next minor version.
- **`qwts-validated` lies inside `qwts-versions`.**

## Output conventions

- stdout carries data only. Diagnostics, progress, and prompts go to stderr.
- A machine-readable mode emits one JSON document. A command that streams
  emits JSON Lines instead and says so in its help.
- Every failure exits non-zero. Failures that a workflow handles carry a
  stable error identifier in a `code` field of the JSON output.
- Exit codes and error identifiers are documented and follow the
  compatibility rule in ENG-0055 decision 5.

## Release gate

The release workflow fails closed. It fails when:

1. `SKILL.md` is missing, or its frontmatter fails the Agent Skills validator
   or the metadata rules above.
2. The packaged executable's `--version` does not parse, or lies outside
   `qwts-versions`.
3. `qwts-validated` is outside `qwts-versions`, or the range's upper bound
   is too high under the version grammar.
4. The bundled skill directory is missing from the release artifact, or the
   bundle-location command does not report it.
5. The skill's representative workflow and output-contract tests fail against
   the packaged executable.
