# Test coverage matrix

This repository is docs and governance only, so its "tests" are the checks its
own CI runs. Each row maps a governed property to the check that enforces it —
`docs-gov` (the `qwts-agent-docs-gov` workflow at `67db7dc`, configured by
`docs-gov.config.json`), `markdownlint` (`npm run lint:markdown`,
`.markdownlint.json`), or a `qwts-agent-ci` composite action (at `f651d5e`).
Priority: **P0** protects governance integrity itself, **P1** should be closed
before the check is relied on fleet-wide, **P2** as time allows.

| Governed property | Enforcing check | Gaps | Priority |
| ----------------- | --------------- | ---- | -------- |
| ENG record fields: `Status`, `Date`, `Issue` on every `docs/decisions/ENG-*.md` | docs-gov `structure.requiredFields` | — | P0 |
| ENG numbering: issue-derived, duplicate-free, home repo and template aliases | docs-gov `decisionSeries` | Which home's counter a locally authored record must draw on is enforced by review, not the gate (decisions README, qwts-agent-docs-gov#1) | P1 |
| Index completeness: a doc unlisted by README, the decision index, the SOP index, or the skills catalog is unreachable | docs-gov `integrity.orphans` (`indexes: ["README.md"]`) | Index row *quality* (scope summaries) is review-only | P0 |
| Relative link integrity | docs-gov `integrity.links` | External URL liveness is not fetched; a pinned commit that disappears upstream is caught only by pin-reachability for `uses:` pins | P1 |
| Stale path references | docs-gov `integrity.stalePaths` (roots `docs/`, `tools/`) | `tools/` does not exist here; the root is kept for template parity and silently matches nothing | P2 |
| Per-doc token budget (2600; named overrides each carrying a reason) | docs-gov `tokenBudgets` | Overrides only accrete; no scheduled re-examination | P2 |
| Structure limits: heading depth ≤ 4, ≤ 250-token summary | docs-gov `structure` | — | P2 |
| Anti-patterns: positional references, unresolved placeholders, duplicate paragraphs | docs-gov `antiPatterns` | `duplicateParagraphs` threshold (160 chars) catches verbatim copies only; paraphrased drift is review-only | P1 |
| Markdown style | `npm run lint:markdown` | Covers syntax only; prose quality is the docs-eval loop and review | P1 |
| SOP edit changelog (every SOP edit updates its footer changelog) | Review only | No machine check; the convention lives in AGENTS.md | P1 |
| Skills catalog pins: catalogued entries at immutable commits | `integrity.links`, `integrity.orphans`, review | The pin-reachability guard checks `uses:` pins in workflows, not skill pins inside `skills/README.md` | P1 |
| Action and capability pins at 40-hex, reachable from the default branch | `ci-runtime-check` + `pin-reachability.mjs` | Freshness (a newer approved commit) is review; reachability only proves the pin exists | P0 |
| Actor/trigger authorization, public-fork policy | `ci-policy` (Action Policy job) | — | P0 |
| Dependency & tooling inventory coverage | `qwts-agent-inventory` workflow + `dependency-inventory.config.json` | Report-only: a new manifest type or tool config is catalogued, never fails the lane | P1 |
| `governance/release-lifecycles.json` consistency | None, by design: the file is unread here; the live catalog is the pinned `qwts-agent-ci` copy (README, Deltas) | The local copy can drift silently; nothing flags divergence | P2 |
| Static analysis | `.github/workflows/codeql.yml` | Runs only when the repo is public; a private flip disables it silently | P2 |
| ENG-0006 §6 conformance | `agentic-primitives-conformance-checklist.md`, self-attested | No machine check that a governed repo's claimed items hold | P1 |
| Governed-repos manifest | `governance/repos.json` in qwts-agent-org; nothing here reads or validates it (AGENTS.md) | Correct by design; a repo missing from the manifest is caught there, not here | P2 |
