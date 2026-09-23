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

## CI policy

The Action Policy job pins `qwts/qwts-agent-ci`'s `ci-policy` action at
`f651d5e6aa17ebee3d5956b45c0378cbcf9c29fa` and runs it in classification
mode. That revision's release-lifecycle catalog has one entry for
`qwts/qwts-agent-sop`, `metadataSystem: none`. Actor and fork enforcement,
lane selection, and the post-merge shortcut all come from that action.
The contract is the
[CI execution policy](https://github.com/qwts/qwts-agent-ci/blob/f651d5e6aa17ebee3d5956b45c0378cbcf9c29fa/docs/ci-execution-policy.md).

## Repository services

The repository is public and runs its CI on GitHub-hosted runners
([ENG-0269](../decisions/ENG-0269-trusted-dependency-reuse.md), 2026-09-15
amendment). Default-branch review rules and the Actions policy are explicit
settings, not Git history. Local checks and remote CI results are reported
separately.
