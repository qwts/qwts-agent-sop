# ENG-0029: Credential minting is authorized by a worktree-bound grant, not by the soul

**Status:** Proposed
**Date:** 2026-09-27
**Issue:** qwts/agent-sop#29

## Context

The agent identity runtime holds one GitHub App private key per rostered App and
mints installation tokens on demand. Credential access today is authorized by
**filesystem position, not identity**: any process running as the workstation
user, from any directory it can reach, can invoke the credential helper, which
reads the key from `~/.config/<slug>/` and mints. The gate is "can you read the
key file", not "are you the identity this belongs to". The App slug comes from
that worktree's git config, which is a file the same caller can write.

The pieces a credential broker needs are already built in
`qwts/agent-bot-identity` — deny-by-default per-principal authorization,
per-operation scoping, immutable proposals bound to operation digests, and audit
receipts, all pointed at sessions and jobs. The credentials were the part left
out, so what exists is a credential broker with the credentials left out.

The obvious design is to authorize the mint by the calling soul. It contradicts
two ratified records, not one:

- [ENG-0081](ENG-0081-transcript-bound-agent-execution-identities.md) req 1: the
  transcript-bound Agent ID "grants no authority."
- [ENG-0172](ENG-0172-agent-space-is-durable-per-soul-storage.md) req 1: the
  Agent ID "is a lookup key and grants no authority"; req 5 keeps Agent Space
  outside "commit-authority boundary, credential store, or `gh` identity
  boundary."

For `git push` from an agent worktree there is no human principal in the loop —
the requester *is* the soul — so authorizing by principal instead does not
resolve it.

## Decision

1. **A worktree-bound grant is the mint authority.** The daemon issues a grant at
   bind time, scoped to one worktree and an explicit operation set. The mint
   boundary checks that grant. It never checks the Agent ID for authority.
2. **The Agent ID stays non-authoritative.** ENG-0081 req 1 and ENG-0172 req 1
   stand as written. This record does not amend or supersede either.
3. **A grant is named for its scope, not for its holder.** It carries the App it
   may mint for, the operations it permits, and the worktree it is bound to. It
   is not a capability URL and conveys no authority beyond that scope.
4. **Binding is fail-closed.** A worktree with no live grant is refused. An
   absent, revoked, expired, or scope-mismatched grant refuses before any mint is
   attempted. An unreachable authorization store refuses; it does not fall
   through to an unscoped mint.
5. **Every grant issue, use, and refusal is auditable, and the receipt carries no
   secret material** — no token, key, or private path. This extends the existing
   audit sink rather than adding a second one.
6. **The grant's lifetime is the worktree's.** It is issued at bind and revoked at
   unbind. A grant that outlives its worktree is a defect, not a cache.
7. **A grant is stored secret-free.** Whatever material backs it — a signed
   artifact, a daemon-side table, or both — leaves no key material in identity
   records, space markers, population rows, principal records, job or event
   records, export packs, or logs.
8. **The local key-file path is retired when the broker serves every rostered
   App**, and not before. Until then it remains the fallback, so no agent loses
   the ability to push mid-transition. Its retirement is a separate recorded
   change, not an implication of this one.
9. **The broker is a separate authority domain from sessions and jobs.** It is
   audited separately and calls one minting implementation.

## Why

Reviewed against [ENG-0012](ENG-0012-decision-priority-order.md), which puts
security first. The choice is between admitting a narrow authority-bearing use
of the Agent ID and introducing a second object. Direction 1 is simpler; this
record takes the more complex option because of what the Agent ID *is*.

The Agent ID is the key for Agent Space paths, population rows, and transcript
locators. Those records are durable and are built to travel — export packs and
handoff transports inherit the boundary. So the same identifier is deliberately
inert in one design and authority-bearing in the other, and its leak changes
character accordingly: a non-authoritative ID escaping a durable record is a
bookkeeping problem, while an authoritative one escaping the same record is a
credential problem. Keeping the identifier inert means the blast radius of every
place it is written stays the smaller one.

A grant is also a better fit for the failure mode that matters. A grant is bound
to one worktree and an operation set, so a leaked grant is a scoped, revocable,
short-lived thing that says which App and which operations. An authority-bearing
soul ID is not scoped at all.

## Consequences

- **A new object and a lifecycle.** Grants must be issued, scoped, revoked, and
  expired, and their failure modes designed. That is real work the broker did not
  previously need, and it is the cost of not weakening two records.
- **The broker cannot be a thin wrapper over the credential helper.** The helper
  has to ask the daemon rather than read the key itself, which changes the `git
  push` path and its failure modes.
- **An unbind must revoke.** A grant that survives its worktree silently widens
  access, so revocation belongs in the unbind path and needs its own test.
- **Nothing here defends against a co-located process.** A caller on the same
  account that can read the daemon's state is outside this control. The binding
  is against the *ambient* credential path — a helper that reads a key file
  because of where it runs — not against a determined local attacker. Claims
  stronger than that would be false, and
  [ENG-0339](ENG-0339-os-account-determines-persona.md) already places the
  persona boundary at the OS account.
- **Machine-versus-account trust is still open.**
  [agent-bot-identity#191](https://github.com/qwts/agent-bot-identity/issues/191)
  records that a multi-account host has one boundary per account rather than one
  per machine. This record does not settle that, and a grant does not help with
  it.
- **Migration is ordered, not instant.** Requirement 8 keeps the key-file path
  authoritative until the broker is complete, so there is a window where both
  exist and the weaker one still works. That window is the accepted cost of not
  breaking `git push` for agents mid-transition.
- **Bearer credentials the runtime holds remain a separate weakness.** Moving
  secrets behind the runtime does not make them unforgeable or unobservable to a
  caller that can reach the process. That is tracked in
  [agent-bot-identity#229](https://github.com/qwts/agent-bot-identity/issues/229)
  and is not addressed here.

## Related

- [ENG-0001](ENG-0001-cross-repo-decision-home.md) — this repository is the
  decision home
- [ENG-0012](ENG-0012-decision-priority-order.md) — the review lens used above
- [ENG-0013](ENG-0013-issue-first-provenance.md) — issue-first provenance
- [ENG-0035](ENG-0035-issue-derived-record-numbers.md) — this record's number is
  its issue's number
- [ENG-0081](ENG-0081-transcript-bound-agent-execution-identities.md) — req 1
  stands unchanged
- [ENG-0172](ENG-0172-agent-space-is-durable-per-soul-storage.md) — reqs 1 and 5
  stand unchanged
- [ENG-0339](ENG-0339-os-account-determines-persona.md) — the persona boundary
  this record does not move
