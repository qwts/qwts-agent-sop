# ENG-0071: A broker carries messages between persona accounts

**Status:** Proposed
**Date:** 2026-09-30
**Issue:** qwts/qwts-agent-sop#71

## Context

[ENG-0339](ENG-0339-os-account-determines-persona.md) runs each harness
persona in its own macOS account, so agents that need to talk to each other
sit in different accounts. The agent-bot daemon is per account. Its per-start
bearer token, in a 0600 file, exists to keep other accounts out, and its
LaunchAgent stops when the account logs out. Nothing today carries a message
from a soul in one account to a soul in another, or holds it while the
recipient's account is offline.

ENG-0339 decision 7 puts machine-scoped state in `/Users/Shared/Public` and
expects coordination to graduate to a broker. Decision 9 grants no ACLs
between accounts. The [agent-comms ADR series](https://github.com/qwts/agent-comms/blob/main/docs/decisions/README.md)
designs the messaging system; this record states the fleet-level rules it
must keep.

## Decision

1. **One broker per machine, installed by the owner.** It listens on a Unix
   socket under `/Users/Shared/Public/agent-comms/`. The directory belongs to
   the broker's account and no client account can write to it. The socket's
   group is the owner-created group of agent accounts and the owner. The
   broker's store lives in its own private state directory, never in the
   shared space.
2. **Clients are paired accounts, never other accounts' files.** Each account
   pairs once from inside itself, and the owner approves the pairing. The
   credential stays in that account's home. No account writes into another's
   home, and no private registry is exposed (ENG-0339 decision 9).
3. **The broker authenticates the account; the account vouches for the
   soul.** A sender may speak only for souls its own account joined. Until the
   account's daemon can vouch for a soul through its connection binding
   ([ENG-0081](ENG-0081-transcript-bound-agent-execution-identities.md)
   amendment of 2026-08-13), the soul is recorded as a claim and nothing
   grants authority on it.
4. **Addresses are the account plus the soul.** The account short name is the
   persona's roster slug, and the soul is its `agent_<uuid>`. Display names
   and avatars never route.
5. **The human is a separate principal.** Pairing the owner's account does
   not authenticate the owner, because delegate harnesses run there too
   (ENG-0339 decision 3). A message is attributed to the human only when sent
   with a human-principal credential that delegate processes cannot read.
6. **Message content is untrusted input.** Under ENG-0081 decision 8, a peer
   message never selects an App, widens access, approves a proposal, or
   stands in for an owner decision.
7. **Chat services are human gateways only.** Telegram and Slack reach one
   account's daemon through its principal adapters. They carry no machine
   traffic between accounts, and bot-to-bot modes stay off for fleet bots.
8. **Cross-machine routing needs its own record.** Execution identities are
   workstation-local under ENG-0081. Joining machines is a separate
   decision.

## Why

The account boundary is the one the kernel enforces, so the broker
authenticates accounts and leaves soul-level separation inside an account to
the daemon, where it already lives. A broker that each account connects to
outbound needs no cross-account ACLs and keeps working when a recipient is
logged out. Keeping chat services out of the machine path avoids their
disclosure, loop, and size limits where they add no reachability.

## Consequences

- Agents in different persona accounts can message each other without
  weakening ENG-0339's isolation.
- The broker's store holds every account's messages and is among the most
  sensitive files on the machine. Running it as its own account keeps even
  the owner's account out of it.
- Each account costs one owner-approved pairing, repeated after revocation.
- Implementation, limits, and the bootstrap sequence belong to the
  agent-comms ADR series; daemon changes stay in agent-bot-identity under
  [ENG-0128](ENG-0128-agent-bot-runtime-ownership.md).
