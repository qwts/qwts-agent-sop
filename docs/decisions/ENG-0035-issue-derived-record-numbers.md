# ENG-0035: ENG record numbers are the originating issue number — allocation cannot collide

**Status:** Proposed
**Date:** 2026-07-23
**Issue:** qwts/playbook-engineering#35

## Context

ENG numbers were allocated by taking the next free number at authoring time.
That counter — the highest number in [the index](README.md) — is only
*reserved* when a PR merges, so two open PRs both read the same number as free
and collide when the second lands.

On 2026-07-23 [PR #27](https://github.com/qwts/playbook-engineering/pull/27)
collided twice in one afternoon: it claimed ENG-0011, then PR #25 merged
(taking ENG-0011–0014), forcing a renumber to ENG-0015; then PR #24 merged
(taking ENG-0015), forcing a second renumber to ENG-0016. Each collision is a
same-line conflict in `README.md`, and a conflicted PR silently stops its
`pull_request` CI runs — so the cost is renumber toil *plus* a quiet loss of
the checks that gate the change.

The repo already has an atomic, monotonic, never-reused counter it was not
using for this: GitHub's issue/PR number sequence. And every record must
already cite an originating issue before it lands
([ENG-0013](ENG-0013-issue-first-provenance.md)) — so the atomic allocation
has already happened by authoring time.

## Decision

1. **A new ENG record's number *is* its originating issue's number**,
   zero-padded to four digits. Issue #35 → `ENG-0035`. The originating issue
   lives in the decision-home repo, `qwts/playbook-engineering`
   ([ENG-0001](ENG-0001-cross-repo-decision-home.md)), so a single counter
   backs the whole series.
2. **Allocation is therefore collision-free by construction.** GitHub hands out
   issue numbers atomically and never reuses them, and the number is fixed the
   instant `gh issue create` returns — before any file is written. Two records
   can never derive the same number because two records can never share an
   issue, and no two open PRs can claim one number.
3. **docs-gov enforces it** (`decision-number` rule, folding in the duplicate
   check that was candidate (c) on the issue). It fails when two records share
   a number, and — for records under this convention — when the `ENG-NNNN`
   filename number does not equal the `**Issue:**` field's issue number, or the
   issue is not in `qwts/playbook-engineering`. So a manual slip fails loudly
   and deterministically in the gate rather than racing at merge.
4. **ENG-0001 through ENG-0016 are grandfathered.** Their numbers predate this
   rule and do not equal their issues; docs-gov exempts them by exclusion glob,
   exactly as ENG-0013 scoped its own Issue field. They stay as written —
   records are never renumbered after the fact.

## Why not the alternatives

- **Reserve the number by merging a stub index row first** moves the race
  earlier instead of removing it: two contributors can still stub the same
  number concurrently, and it adds a mandatory pre-merge round trip to every
  record. Issue-derivation reuses a counter that is *already* atomic and
  *already* allocated, at no extra step.
- **A duplicate-number check alone** is detection, not prevention: collisions
  still happen and still block CI; the check only makes them fail loudly. It is
  worth having, so it is folded in as enforcement (item 3) rather than adopted
  as the convention.

## Consequences

- **Numbering goes sparse and non-consecutive.** The next record jumps from
  ENG-0016 to ENG-0035, and gaps are permanent. "Allocated in order" becomes
  "allocated in issue order" — still monotonic, so ordering survives, but the
  highest number no longer counts the decisions, and PRs share the counter, so
  numbers are consumed by non-records too.
- **The series is coupled to GitHub's counter.** If issues ever migrate off
  GitHub, the derivation rationale goes with them (the same distant risk
  ENG-0013 already carries); existing numbers stay valid as bare identifiers.
- **The grandfather carve-out is permanent** — one more escape glob a reader
  must know means "predates the rule," not "unchecked."
- **One issue backs at most one new record.** An amendment to an existing
  record does not take a new number; a genuinely new decision needs its own
  issue. This matches ENG-0013's one-issue-per-record shape.
- **Four-digit ceiling.** Past issue #9999 the four-digit `ENG-NNNN` field
  overflows; the gate will flag it as unparseable, which is the signal to widen
  the field — a problem this repo is thousands of issues from having.

## Amendment — 2026-09-27: two counters after the template/instance split

Decision 1 assumed one decision-home repository and one issue counter.
[ENG-0355](ENG-0355-static-router-one-pointer-pinned-capabilities.md) split
the series into a template (`qwts/agent-sop`, formerly `playbook-engineering`
and `dev-steward`) and instances (`qwts/qwts-agent-sop` for qwts). An instance
imports the template's records and files its own. What applies in an instance
([qwts/qwts-agent-sop#36](https://github.com/qwts/qwts-agent-sop/issues/36)):

1. **An imported record keeps its template number and cites its template
   issue.** Its number is history, never re-derived.
2. **An instance-authored record takes the instance's own issue number and
   cites that issue.** The template's counter is not borrowed: a template
   issue opened only to allocate a number for an instance decision is the
   ceremony [ENG-0013](ENG-0013-issue-first-provenance.md) exists to prevent,
   and it spends a public counter on a decision the template never carries.
3. **Collision between an imported and an instance-authored record is
   possible, and the gate refuses it.** The two counters are independent.
   Decision 3's duplicate check fails the second record to land, whichever
   kind it is, and the record already landed keeps its number. When the
   instance record lands second, it is filed from a fresh instance issue,
   linked to the first. When the template record lands second, the alignment
   adopts it the same way: a fresh instance issue, and a record numbered from
   it whose body names the template record it adopts and that record's
   original number. Renumbering a landed record is never the resolution.
4. **docs-gov's `decisionSeries.homeRepo` in an instance is the instance
   repository; the template and its former names are aliases.** At the pinned
   docs-gov the aliases apply to every record, so an instance-authored record
   that cites a template issue passes the gate; review, not the gate, holds
   item 2 until docs-gov can distinguish imported records from authored ones
   ([qwts/qwts-agent-docs-gov#1](https://github.com/qwts/qwts-agent-docs-gov/issues/1)).
5. **ENG-0384 keeps its number by exception.** It was filed from template issue
   agent-sop#384 before this amendment, its body links the instance request
   ([qwts/qwts-agent-sop#25](https://github.com/qwts/qwts-agent-sop/issues/25)),
   and it is cited by the baseline SOP. It is the last record numbered that
   way. ENG-0029, which cited the template while its issue was here, was
   withdrawn and removed
   ([qwts/qwts-agent-sop#31](https://github.com/qwts/qwts-agent-sop/issues/31),
   [#43](https://github.com/qwts/qwts-agent-sop/pull/43)).

Consequences: the instance's numbers interleave with the template's, so a
reader cannot tell an imported record from a local one by number alone; the
`**Issue:**` field says which. Decision 4's grandfather carve-out is unchanged.

## References

- qwts/playbook-engineering#35 — the originating issue
- [ENG-0013](ENG-0013-issue-first-provenance.md) — issue-first provenance; this record derives the number from the issue that rule already requires
- [ENG-0001](ENG-0001-cross-repo-decision-home.md) — the decision home whose issue counter backs the series
- [Decision index](README.md) — the Numbering section documents the rule
