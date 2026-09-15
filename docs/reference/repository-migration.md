# Repository migration

`qwts/qwts-agent-sop` owns the `qwts` procedures, fleet governance, and shared
operational tooling. `qwts/agent-sop` develops the configurable public Agent SOP
framework. The migration is tracked in
[qwts-agent-sop#1](https://github.com/qwts/qwts-agent-sop/issues/1), following
[the source discussion](https://github.com/qwts/agent-sop/discussions/365).

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

Current `qwts` procedure and tooling references use this repository. The
discovery projection migrates old `main` and `master` governance links while
preserving immutable revision links and historical issue references.

`qwts-agent-org` is the organization configuration home. Its eventual profile
should select the applicable sources here; the Agent SOP TOML contract is still
under discussion, so this migration does not invent an operational `org.toml`.
Other organizations choose their own sources. They do not inherit the `qwts`
fleet, bot identities, runners, or mandatory procedures by opening Agent SOP.

## Compatibility and cutover

The original repository retains its existing operational paths while consumer
changes are reviewed. A Git copy creates no redirect from old URLs to new URLs.
Existing pinned consumers keep using their exact revisions. The copied `v1`
also retains its original embedded tooling references; it is historical
compatibility, not proof of a completed source cutover. New consumer migrations
must select a reviewed revision containing the updated references.

Do not remove compatibility paths or advance the destination `v1` until the
affected consumers have been inspected and the replacement revision passes
the required validation. Running the discovery projection is an explicit
consumer change, not permission to distribute all harness files automatically.

## CI policy bootstrap

Both workflow policy pins resolve to this repository's imported revision,
`qwts/qwts-agent-sop@41af9d7917a418810b3277f2cec969e4e003b0b3`, which is reachable
from its `main`. CI has no policy dependency on the framework repository.

The imported revision does not contain a lifecycle entry for
`qwts/qwts-agent-sop`. The migration adds that entry, but the pinned action
continues to read the catalog at its own revision. Full CI therefore remains
blocked on a reviewed policy bootstrap in this repository. Actor authorization,
missing-repository validation, and pin reachability must remain enforced.

## Repository services

All four Agent SOP project repositories are public. The destination inherits
the source's default-branch review rules and Actions policy through explicit
settings updates. Self-hosted runners, secrets, and external integrations do
not follow Git history. At import, the destination had no registered runners;
its self-hosted CI requires provisioning before it can validate a release.
Local checks and remote CI results must be reported separately.
