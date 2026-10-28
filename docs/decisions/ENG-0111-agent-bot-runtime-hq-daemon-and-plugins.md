# ENG-0111: The agent-bot runtime is an HQ daemon plus plugins

**Status:** Proposed
**Date:** 2026-10-27
**Issue:** qwts/qwts-agent-sop#111

## Context

[ENG-0128](ENG-0128-agent-bot-runtime-ownership.md)'s amendment of 2026-10-10
planned to extract identity into `qwts/agent-identity`. Soul and harness work
would stay in agent-bot-identity as the `agent-bot` facade
([ADR-0645](https://github.com/qwts/agent-bot-identity/blob/main/docs/decisions/ADR-0645-one-runtime-repository-four-owned-modules.md)).

The *Soul Agent Platform* draft 0.4 then made HQ a trusted local host service,
separate from the Souls it runs. The agent-bot server became the identity
service, responsible only for credential custody and issuance. The owner
decided the mapping on 2026-10-27, as decision D1 of
[the agent-bot-daemon split plan](https://github.com/qwts/agent-bot-daemon/blob/1780433/docs/daemon-plugin-split.md).

## Decision

This record replaces decisions 1 to 3 of ENG-0128's 2026-10-10 amendment.
ENG-0128's other decisions and amendments stand, including the governance CLI
boundary (decision 3).

1. **Four runtime repositories.**
   - **agent-bot-daemon** is HQ. It owns the `agent-bot` command and the
     runtime release.
   - **agent-bot-identity** is the identity plugin.
   - **agent-bot-comms** is agent-comms renamed, in its broker, client and
     relay roles.
   - **agent-bot-soul-runtime** holds the soul package, render and admission
     logic, plus the Soul's entry point.
2. **managed-machine stays the provisioner.** It creates HQ's account and the
   member accounts.
3. **The agent-bot server is the identity service:** custody and
   per-operation, short-lived issuance only.
4. **`qwts/agent-identity` is not created.**
   [ADR-1183](https://github.com/qwts/agent-bot-identity/pull/1194) retires
   the extraction and carries ADR-0645's module rules forward.
5. **Governance consumers keep invoking `agent-bot`.** The CLI contract,
   `AGENT_BOT_BIN` and the fail-closed behaviour do not change when the
   executable moves to agent-bot-daemon.

## Why

The extraction drew the line around identity alone. Draft 0.4's line runs
between the trusted host and the untrusted Souls. Moving HQ out matches the
trust boundary. A strangler migration keeps the CLI and MCP surfaces working
throughout.

## Consequences

- Code ships from the repository that owns it, which changes as modules
  move. Until a module moves, it ships from agent-bot-identity.
- This is more churn than the extraction would have caused. It is accepted
  because the boundary matches.
- [qwts-agent-org#32](https://github.com/qwts/qwts-agent-org/issues/32), the
  agent-identity pin, is not needed in that form. The four repositories get
  capability pins when they first release.
- Credential custody follows draft 0.4: no long-lived secret on the host.
  `legacy-local-custody` is off by default, and is removed at stage 3 exit or
  on 2027-01-31, whichever comes first.
