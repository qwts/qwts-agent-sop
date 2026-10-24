---
name: governance-bootstrap
description: Discover current organization and SOP guidance at task entry and use the current client skill before running a qwts CLI.
---

# Discover current governance

Install only after the runtime supports SOP-local catalog entries and
`agent-bot skill client-cli-skill --json` succeeds for the selected SOP.
This installed file is a small router, not a policy snapshot. At task entry
and again before using a qwts CLI:

1. Run `agent-bot sop --json`. Inspect the selection source and record the
   resolved org and SOP repository names and exact commits. Follow the
   selected organization's current guidance. If no SOP is selected, ask for
   the organization; do not infer it from a remote or use an archived template.
2. Run `agent-bot skill client-cli-skill --json`. Read `text` from that
   result and follow it. Record its `catalog` and source commit. It resolves
   fresh guidance on each use; do not reuse a previously copied client skill.
   This is a separate resolution from step 1: its returned catalog commit is
   authoritative for this skill read, even if the SOP head has since advanced.
3. For another named SOP skill, use `agent-bot skill <name> --json` and read
   only that skill. The resolver keeps catalog and SOP-owned skill content
   on one immutable snapshot. CLI bundles come from the installed executable.

## Repository and GitHub work routes through session setup first

This router discovers guidance; it does not establish the session. Before
repository work — and always before a GitHub or publishing operation —
load `agent-bot skill agent-space` and follow the trait-routed startup in
the branch/PR SOP: observed placement, worktree binding, and, for GitHub
operations, a live session binding. A CLI feature workflow (including
`signed-commit`) never substitutes for that setup, and neither a minted
worktree token nor a ready `doctor` proves the live session binding.

Route on what is observed, and disclose only the matching branch: a
session already in a bound soul worktree skips check-in remediation; a
session that cannot establish the live session binding (for example, no
MCP `bind` tool in the harness) stops and reports that exact blocker. No
alternate identity, real-binary path, or override variable replaces it.

Failure to refresh blocks the dependent operation. Report the failed command
and error; do not present a cache as current. Reads can be retried after the
cause is repaired. Explicit `@ref` selections are intentional snapshots until
the owner migrates them. Fresh discovery does not activate policy, personas,
launch state, credentials, or grant authority. Existing trust gates remain.
