# Repository migration

`qwts/qwts-agent-sop` is the `qwts` instance of the Agent SOP template:
the procedures, decisions, and guides the organization follows.
`qwts/agent-sop` develops the configurable public template. The migration is
tracked in [qwts-agent-sop#1](https://github.com/qwts/qwts-agent-sop/issues/1),
following [the source discussion](https://github.com/qwts/agent-sop/discussions/365);
the capability split that followed it is
[ENG-0355](../decisions/ENG-0355-static-router-one-pointer-pinned-capabilities.md).

## Preserved history

The destination was initialized at source commit
`41af9d7917a418810b3277f2cec969e4e003b0b3`, with `v1` still pointing to
`0ab1396c527536627981b27473d38d5bbdef3138`. Existing commits, signatures,
and ancestry are unchanged. This import does not move the source repository's
other branches, issues, pull requests, discussions, releases, or settings.

Historical issue and PR URLs remain at their original home. Existing ENG
records retain their issue identifiers and text. Their issue-number allocation
still belongs to the original `agent-sop` tracker; the destination has an
independent counter, so changing `docs-gov.config.json`'s decision home would
not make that counter compatible. A new numbering contract needs a separate
decision before creating ENG records from destination issue numbers.

## Organization sources

Current `qwts` procedure references use this repository.
[qwts-agent-org](https://github.com/qwts/qwts-agent-org) is the organization
configuration home: its `org.json` pins this repository as the SOP source and
every capability repository at a commit, and its `governance/` holds the
governed-repos manifest, the App roster, and the organization profile that
used to live here. The revision this instance aligned with is recorded in
[README.md](../../README.md), together with the deltas it carries. Other
organizations choose their own sources. They do not inherit the `qwts`
fleet, bot identities, or mandatory procedures by opening Agent SOP.

## Compatibility and cutover

The original repository's historical revisions remain addressable by commit.
A Git copy creates no redirect from old URLs to new URLs. Consumers pin the
capability repositories directly at a commit (ENG-0282); the copied `v1` tag
is historical compatibility only — it still embeds the pre-split tooling
references and is not a consumption path for anything.

## CI policy bootstrap

The Action Policy job pins `qwts/qwts-agent-ci`'s `ci-policy` action at
`3a5617b287d922e37f262210a1d8750d8217b56d`. That revision's
`governance/release-lifecycles.json` has no entry for `qwts/qwts-agent-sop`,
so its classifier would fail closed here. CI therefore invokes the action in
`authorization-only` mode, which still rejects unauthorized actors and public
forks before checkout, and a repository-scoped step then selects full
validation for every enabled event. Exact-SHA PR preflight reuse stays
available; post-merge smoke and release-projection shortcuts are not selected.
All validation jobs and the final success gate remain required.

This is a temporary scheduling delta from the
[CI execution policy](https://github.com/qwts/qwts-agent-ci/blob/3a5617b287d922e37f262210a1d8750d8217b56d/docs/ci-execution-policy.md).
After a `qwts-agent-ci` revision carrying this repository's lifecycle entry is
pinned here, remove the bootstrap output step in a reviewed follow-up.

## Repository services

The repository is public and runs its CI on GitHub-hosted runners
([ENG-0269](../decisions/ENG-0269-trusted-dependency-reuse.md), 2026-09-15
amendment). Default-branch review rules and the Actions policy are explicit
settings, not Git history. Local checks and remote CI results are reported
separately.
