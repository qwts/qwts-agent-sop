# ENG-0085: Local-first diagnostics share semantics across engine and host boundaries

**Status:** Proposed
**Date:** 2026-10-11
**Issue:** qwts/qwts-agent-sop#85

## Context

[agent-comms #164](https://github.com/qwts/agent-comms/issues/164),
[agent-bot-identity #983](https://github.com/qwts/agent-bot-identity/issues/983)
and [GeniusBar #370](https://github.com/true-line/geniusbar/issues/370) need
correlated evidence across independently usable components. A host SDK cannot
observe every standalone operation. Separate vocabularies can make one engine
failure look like several incidents.

Accepted agent-comms
[ADR-0007](https://github.com/qwts/agent-comms/blob/main/docs/decisions/ADR-0007-observability-and-geniusbar.md)
keeps harness collectors read-only, session-bound and beside the account daemon.
[ADR-0002](https://github.com/qwts/agent-comms/blob/main/docs/decisions/ADR-0002-messaging-plane-on-the-agent-bot-daemon.md)
requires zero runtime npm dependencies. The private sink in
[agent-comms #167](https://github.com/qwts/agent-comms/pull/167) establishes
neither a shared schema nor a host diagnostics API.

This proposal adds no public runtime fields, endpoints, exporter, dependency
exception or reporting consent. The
[acceptance plan](../reference/diagnostic-correlation-acceptance.md) records
operation boundaries, evidence and unresolved joint-review choices.

## Proposed decision

### Ownership and authority

| Owner | Produces | Consumes and presents |
| --- | --- | --- |
| agent-comms | Broker connection, delivery, queue, acknowledgment and broker lifecycle observations | Standalone diagnostics and authorized broker interfaces |
| agent-bot-identity | Launch/resume, harness exit, execution, runtime selection, approval and account-daemon lifecycle observations | Existing operational history and authorized daemon interfaces |
| Host, including GeniusBar | UI, native shell and bridge observations | Engine observations through supported authorized APIs; combined presentation and optional reporting |

No host reads harness logs, workspaces or engine-private state to implement
this contract. Existing collectors are reused rather than duplicated.
Correlation is never identity or authority. Authentication, principal grants,
recipient permissions and custody checks apply independently. New health or
inspection aggregates need a reviewed visibility policy that does not reveal
hidden entities, including through counts.

Canonical mailbox state, acknowledgment, task acceptance/completion, lifecycle
history and audit/security receipts remain distinct from sampled diagnostics.
No observation proves a soul is present, reachable or ready for work.

### Event and context semantics

The shared profile distinguishes these concepts; concrete wire names and IPC
placement require adoption review:

- A schema major identifies the diagnostic profile. Additive optional fields
  may change within a major; readers discard unknown fields rather than store
  or forward them. Unsupported majors yield a bounded compatibility status
  without failing the underlying operation.
- Component and actual running build version identify the producer. Missing
  versions are unknown. Bundled versions and provenance remain separate;
  GeniusBar reuses its existing #290 version scheme.
- A producer-generated random event ID identifies one observation. Forwarding
  preserves that ID and producer attribution. A host may add a related event
  with its own ID, but does not re-emit an engine event as a new engine
  incident. Deduplication state has finite count and lifetime.
- A random operation ID groups a logical operation; retry attempts are
  separate stages or child spans. Message IDs, soul IDs, account names and
  paths are not correlation IDs. Restart starts a new process session;
  durable continuation is not inferred from diagnostic IDs.
- Trace/span context represents causality independently of event identity.
  Use a constrained [W3C Trace Context](https://www.w3.org/TR/trace-context/)
  profile, with validation at every IPC boundary and local sampling
  decisions. No arbitrary baggage or provider-specific context is propagated.
  This restricted profile does not claim complete W3C forwarding conformance.
- Allowlisted operation, stage, stable code and outcome describe an event.
  Expected refusal is distinguishable from transport/implementation failure
  and uncertain storage. Existing public codes retain their meanings;
  diagnostic codes are append-only and never authorize action.
- Observation time supports display ordering. Duration is a nonnegative
  monotonic elapsed measurement with stated unit and stage scope. Wall-clock
  differences between processes are not latency. Unknown measurements are
  absent or explicitly unknown, never fabricated zeroes.

Incoming context is untrusted even after validation: a caller can encode
sensitive data in a valid-looking identifier. At untrusted ingress the
preferred policy is to mint local context. Reusing an incoming identifier or
retaining a causal link needs a reviewed disclosure policy; incoming sampling
flags cannot enable upload or override local limits. Joint review must settle
trust boundaries and remapping before cross-process continuity is claimed.

### Privacy, resource limits and reporting

Construct events from fixed allowlists before persistence or export. Do not
serialize Error objects, arbitrary messages, stacks, causes, raw log lines,
environment values, bodies, prompts, tool inputs/results, source, memory,
credentials, identifying paths or stable soul identifiers. A per-code details
registry permits explicitly typed, bounded values only; unknown details are
dropped. No automatic dumps, screenshots, replay or attachments are enabled.

Metric labels come from finite component/operation/stage/code/outcome sets.
Event IDs and principal, soul, account or message identifiers are not metric
labels. Queue depth/age and acknowledgment latency state their scope and do
not imply execution or completion.

Each adopter publishes finite event/attribute bytes, queue bytes/count,
in-flight work, retry/time limits, deduplication, sampling and retained-volume
budgets. Components may use stricter limits. Disk/rotation limits apply when
a component adds diagnostic persistence; the broker's stderr sink creates no
diagnostic disk store. Drops use saturating counters and cannot recursively
create diagnostics. Failed or stalled sinks/exporters cannot delay delivery,
launch, permission decisions or natural shutdown.

Standalone diagnostics require no host, reporting account, vendor, collector
or telemetry network.
Reusable modules do not initialize global exporters. OTel-compatible manual
instrumentation uses a reviewed dependency-free adapter or an explicitly
approved dependency exception in its owning repository.

Remote reporting is off by default. Operator or host configuration and consent
select an identified, replaceable destination. Disabling reporting stops future
sends, including queued work, with documented cancellation and queue disposal;
transmitted data cannot be recalled. Local inspection/export is separately
authorized, previewable and bounded by time and bytes. Preview and written
content agree; failures are not success. Export never implies upload or public
issue creation.

### Adoption and review

Joint review involves broker, execution-engine and host owners. Each repository
then records its ADR consequences, versioned schema, fixed code/details
registry, numeric budgets and compatibility vectors before shipping public
fields. Host provider/Sentry evaluation, Tauri/Rust/web/Node integration,
symbols and consent UI stay host decisions. This ENG grants no deployment or
symbol-upload authority.

Adoption records exact versions/commits, source and packaged platform evidence,
privacy/performance limits and sibling integration status. One repository's
tests do not establish interoperability. All three epics remain open until
their own acceptance and closure evidence are satisfied.

## Why

A semantic contract lets independent engines remain useful while a host adds
presentation. Manual boundary instrumentation can respect dependency and
privacy constraints more directly than blanket capture. Separate event and
operation identity supports forwarding without changing authoritative state.

## Consequences

- Joint review and local adoption delay full correlation; private diagnostics
  and failure/performance fixtures can proceed meanwhile.
- Allowlisting and finite queues deliberately lose detail. Abrupt exits leave
  incomplete timelines; absent diagnostics do not prove success.
- Context remapping and finite deduplication can split a trace or permit a late
  duplicate. Consumers show these limits honestly.
- Adapters and compatibility/privacy vectors require maintenance. A provider
  SDK or repository consolidation is not a substitute for the shared boundary.
