# Live governance discovery

Live instructions follow the selected active org and SOP repositories'
protected default branches. Each operation resolves those heads once and
reads at the exact resulting SHAs. Capabilities remain immutable pins and
CLI bundles remain paired with their installed executables. This is the
2026-10-17 amendment to
[ENG-0282](../decisions/ENG-0282-immutable-pins-recorded-selection-no-aligner.md)
and [ENG-0355](../decisions/ENG-0355-static-router-one-pointer-pinned-capabilities.md),
implementing [org issue 66](https://github.com/qwts/qwts-agent-org/issues/66).

## Selection and migration

The owner selects `org = "qwts/qwts-agent-org"` in the existing local
`~/.config/agent-sop/config.toml` (schema version 1, `[repos]`). The org's
`sources.sop.repo` selects `qwts/qwts-agent-sop`. A missing SOP ref means live
discovery; a legacy SOP manifest SHA is provenance only. An explicit local
`repos.sop` overrides that repository. Bare names follow remote `HEAD`;
explicit `@ref` selections retain their meaning, including historical SHAs.
Full replay sets both org and SOP SHAs; an org-only snapshot still follows
the declared SOP's live head.

Deploy the supporting runtime first: [agent-bot PR 1152](https://github.com/qwts/agent-bot-identity/pull/1152)
adds SOP-local relative catalog resolution. Before installing the bootstrap,
run `agent-bot skill client-cli-skill --json` with the selected SOP containing
this catalog and confirm it returns `text` and the catalog's exact commit.
A shipped runtime that rejects the relative entry is not ready for migration. Back up old configs; remove a frozen
`@<sha>` when migrating to live behavior and remove an unnecessary SOP
override. Do not automatically rewrite intentional rollback selections.
Run `agent-bot sop --json` and verify both resolved commits. Then remove the
legacy manifest SOP ref. No recurring promotion or pin-bump PR is needed.
On first selection, or an intentional change of selection, review the
selected SOP's `persona.toml` or confirm that it is absent. Then explicitly
record that reviewed snapshot with `agent-bot sop persona --json` and verify
`agent-bot doctor --machine-only --json`. A selected SOP without a recorded
snapshot refuses soul launches, wakes, tasks, and turns. An absent file still
needs an absence record; it leaves the user's existing launch settings in
charge. Recording a mapping can affect launch routing and must be intentional.
This initial migration step is separate from routine discovery: do not
re-record personas or activate policy hooks on each freshness refresh.

The active instances are `qwts-agent-org` and `qwts-agent-sop`; archived
`agent-org` and `agent-sop` are templates and remain untouched.

## Shared bootstrap and skills

Install only [governance-bootstrap](../../skills/governance-bootstrap/SKILL.md)
into `~/.agents/skills/governance-bootstrap/SKILL.md`. Back up and remove an
older copied `client-cli-skill` from shared discovery once the bootstrap
works, to avoid conflicting instructions. The bootstrap calls
`agent-bot skill client-cli-skill --json` at use; the current client workflow
is read from the resolved SOP, so updating it needs no manual reinstallation.
Other domain skills stay on demand. This small router is an explicit
exception to the harness-install prohibition in ENG-0006, ENG-0055, and
ENG-0064; it does not install CLI bundles into harness discovery.

SOP-local relative catalog links use the catalog's resolved SHA. External
capability skill links require full immutable commits. The installed CLI's
own skill remains authoritative for its commands and version pairing.

## Failure and activation boundaries

Record resolved org/SOP SHAs for each operation. A branch moving during a
read cannot change that operation's snapshot; the next operation refreshes.
Unavailable remotes, absent catalogs, invalid manifests, and failed trust
checks stop dependent work. Cached documents are historical evidence and
must never be described as current without successful refresh.

Protected default branches and repository review controls establish approval;
the resolver does not create or change those controls. Foreign soul selections
retain exact repository-and-commit trust. A new head can therefore require a
new explicit trust decision. Policy hooks, persona mappings, and daemon
launch caches keep their explicit activation contracts. Discovery alone
never reactivates or authorizes them.
