# ENG-0282: Versioning is immutable pins with recorded selection; there is no aligner

**Status:** Proposed
**Date:** 2026-09-15
**Issue:** qwts/agent-sop#282

## Context

[ENG-0279](ENG-0279-immutable-releases-and-repo-lockfiles.md) proposed
signed, immutable releases with a hash manifest, a per-repository lockfile
(`.playbook/lock.json`), and a nightly aligner that opens one align PR per
repository, under the rule that the fleet runs one version, the latest.
[ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md) then
retired push distribution: nothing is copied into a repository any more, so
there are no distributed files to hash, no lockfile to write, and no aligner
to run. What ENG-0279 got right, that a consumed reference must be immutable
and auditable, still needs a home. Issue #282 is that home.

## Decision

1. **Every consumed reference is a commit.** `uses:` lines, the entries of an
   org repository's `org.json`, and the router's sources name a 40-hex commit.
   Tags may be published for people; nothing consumes them. A branch or a
   moving tag in any of these places is a validation error.
2. **The pin is the record.** Selection is recorded where it is consumed: the
   consumer's workflow file, the org repository's `org.json`, the router's
   `routes.json`. There is no separate lockfile, because a second record of
   the same choice can only disagree with the first.
3. **Pins are verified reachable.** A pinned commit must be an ancestor of the
   pinned repository's default branch, so a squash merge never leaves a
   consumer on a dangling object. The pin-reachability check that enforced
   this here moves to `qwts-agent-ci` and applies to every consumer's
   workflows and to `org.json`.
4. **Capability repositories forbid moving history.** Tag deletion and tag
   updates are refused by a tag ruleset, and the default branch is
   protected, so a pinned commit stays reachable. This is an owner setting
   per repository, recorded in the fleet manifest.
5. **Updating is a reviewed pull request in the consumer**, opened by whoever
   needs the change: a person, or a bot on request. There is no scheduled
   aligner and no fleet-wide latest rule. A repository behind a pin holds a
   recorded choice, not a drift state; a dashboard may show which pin each
   repository carries and never fails one for it.
6. **Rollback is a pin change in the same place.** A defective commit is
   superseded by a fixed one; consumers move when they choose. Nothing is
   yanked, because nothing was pushed.

## Consequences

- ENG-0279 is superseded. ENG-0004's amendment of 2026-08-22, which pointed at
  immutable releases, is itself superseded by the amendment that points here:
  consumers pin exact commits directly, and no release is required.
- Nothing to build: no release manifest, no hash tooling, no lockfile schema,
  no aligner workflow, no staleness view. The tag ruleset and the reachability
  check are the whole integrity surface.
- A security-critical fix propagates only as fast as consumers merge their
  bump. The mitigation is a bot-authored bump PR opened across consumers on
  request, a fan-out someone triggers, not a scheduler.
- Auditing "what is this repository running" is reading its own files: the
  commit in each `uses:` line and in `org.json`, all reachable from a
  protected default branch.
- Version churn in a consumer is visible as pull requests in that consumer,
  which is the review point ENG-0279 wanted and the fleet keeps.

## Alternatives

- **Keep the aligner but make it opt-in:** rejected. An opt-in scheduler is a
  push channel with an off switch, and every repository would carry the stub.
- **Floating major tags (`@v1`):** rejected, as ENG-0279 already found. A
  mutable reference cannot be audited and propagates silently.
- **A lockfile in each repository:** rejected. It duplicates the pin and can
  disagree with the `uses:` line it claims to describe.
- **Fleet-wide latest as a gate:** rejected. It turns every upstream commit
  into a fleet-wide obligation, which is the propagation model ENG-0355 ended.

## References

- [ENG-0004](ENG-0004-centralize-shared-cicd.md), [ENG-0279](ENG-0279-immutable-releases-and-repo-lockfiles.md),
  [ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md).

## Amendment, 2026-10-09 — governance freshness without mutable execution pins

**Scope:** This amendment changes the promotion and freshness policy for
authoritative organization governance (the organization-specific SOP and agent
conventions). It does **not** introduce an aligner for ordinary capability
dependencies, CI `uses:` pins, release inputs, or repository baseline files.

### Decision

1. **Separate discovery from consumption.** An agent resolving the organization's
   authoritative SOP discovers the latest *approved* commit on its protected
   default branch at task entry, resolves that ref to an immutable 40-hex SHA,
   and consumes only that SHA for the task. Record the resolved source repository
   and SHA in the task's execution provenance. A moving branch is a discovery
   selector, never an execution pin. Historical replay uses the recorded SHA,
   not today's branch tip.
2. **Do not let the recorded baseline silently become policy.** The
   `sources.sop.ref` entry in the org repository's `org.json` remains an
   immutable, reviewed baseline and reproducibility record. It is **not** a
   ceiling on approved governance changes. The agent must not mistake an older
   baseline for the currently approved instructions after a newer revision is
   known. Explicit owner-selected `repos.sop` overrides remain available for
   testing and requested version selection; their selected commits are recorded.
3. **Promote governance on events, not a timer.** A merge to the protected SOP
   default branch emits a trusted repository event carrying the repository
   identity and merged SHA. The authorized consumer workflow verifies that SHA,
   coalesces or supersedes outstanding promotion work, and opens/updates one
   reviewed PR in `qwts/qwts-agent-org` to advance `sources.sop.ref`. The
   event is a notification to fetch and verify Git state, **not** instructions
   copied from a commit message, PR body, or webhook payload. Delivery must be
   idempotent and scoped to the approved governance source.
4. **Keep promotion reviewable and auditable.** The new baseline SHA, old SHA,
   merge provenance, validation results, and changed governance sections are
   visible in the promotion PR. Do not directly push an unreviewed governance
   pin or bypass the consumer's existing merge rules. An owner-approved
   auto-promotion policy may be introduced separately, with explicit gate and
   rollback semantics; this amendment does not silently enable auto-merge.
5. **Expose and recover from missed events.** Promotion failure, an invalid
   source SHA, or a behind-baseline state must be surfaced with the pinned SHA,
   the newer verified SHA (when available), and the failed action. Agents do an
   **on-demand freshness check at task entry**, not a continuous or scheduled
   poll; this is also the recovery path for lost delivery. If current authority
   cannot be established, report that limitation and do not silently assert
   stale guidance is current. Where an unverified instruction would authorize a
   sensitive write, do not infer authorization from that stale snapshot.
6. **Retain owner control.** The owner's explicit instructions prevail over
   governance defaults, per
   [the agent conventions](../reference/agent-conventions.md#explicit-user-instructions).
   Freshness machinery provides reliable context; it is not another permission
   ceremony or a reason to challenge an already authorized task.

### Supersession and boundaries

- Decision 1 (immutable consumed commits), decision 2 (a single recorded pin
  per consumer), decision 3 (reachability), decision 4 (protected history), and
  decision 6 (rollback by pin change) remain in force.
- Decision 5's "bot on request" and "behind a pin is not drift" language, and
  the no-staleness-view/no-event-propagation consequences and alternatives,
  are **superseded only for authoritative governance**. They continue to apply
  to other capabilities and dependency pins.
- ENG-0355's "nothing is pushed" means no distribution of files or rules into
  consumer worktrees. A bounded, event-triggered pull request that advances
  the org repository's *one existing pointer* is not file distribution.
- This is a **governance design decision**, not a claim that the event workflow,
  freshness gate, or promotion PR machinery is already deployed. Implementation
  and integration tests must follow in their owning repositories.

### Acceptance criteria for implementation

- A protected SOP `main` merge causes an authenticated, idempotent proposal
  to advance `qwts-agent-org/org.json`, without a scheduler or repeated
  polling and without creating per-repository copies.
- A second merge supersedes or updates pending promotion work instead of
  producing competing PRs; reordered/duplicate events cannot roll back pins.
- At task entry, resolved governance provenance includes the authoritative
  verified SHA; a stale baseline is observable rather than silently accepted.
- Source spoofing, malformed event payloads, unreachable SHAs, failed gates,
  or lost event delivery never grant authority or create an unreviewed update.
- Capability and CI dependencies continue to resolve at explicitly selected
  immutable SHAs. An approved rollback selects and records a specific SHA.
