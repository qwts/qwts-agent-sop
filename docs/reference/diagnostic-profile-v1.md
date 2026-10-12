# Proposed diagnostic profile v1

Concrete adoption candidate for Proposed
[ENG-0085](../decisions/ENG-0085-local-first-diagnostic-correlation.md) and
[SOP #85](https://github.com/qwts/qwts-agent-sop/issues/85). This document is
for joint review, not a released API. No producer, endpoint or exporter adopts
these fields through documentation alone. The existing private broker stderr
format remains unchanged until its separately reviewed adoption.

## Exact event shape

One event is a UTF-8 JSON object, at most 2048 bytes including its JSONL newline.
All objects have only the listed fields. Optional fields are absent when
unknown; null is allowed only for `buildVersion`. Strings are Unicode scalars.
There are no free-form messages, Error objects, paths, identity IDs or arbitrary
attribute maps. Producer code constructs the object; it does not spread input.

| Field | Required | Type and bound |
| --- | --- | --- |
| `v` | Yes | Integer `1`. |
| `component` | Yes | Registered literal, at most 32 ASCII bytes. Initial comms values are `agent-comms-broker` and `agent-comms-client`; bot and host register their own literals. |
| `buildVersion` | Yes | Running version string, 1–64 ASCII bytes matching `^[A-Za-z0-9][A-Za-z0-9.+_-]{0,63}$`, or null when unavailable. |
| `buildSource` | Yes | `running` with a version; `unknown` with null. Bundled version provenance stays in host #290's metadata, not this field. |
| `eventId` | Yes | Producer-generated random lowercase UUID v4; one ID per observation, preserved by forwarding. |
| `sessionId` | Yes | Random lowercase UUID v4 generated at process start; never a persisted installation, account or soul ID. |
| `event` | Yes | Registered literal, at most 64 ASCII bytes; selects the exact tuple and details schema. |
| `operation`, `stage`, `code`, `outcome` | Yes | Exact registered tuple, each at most 64 ASCII bytes. No values copied from arbitrary thrown objects. |
| `observedAtUnixMs` | Yes | Nonnegative safe integer, producer wall-clock observation time. Clock ordering across producers is approximate. |
| `durationMs` | No | Nonnegative safe integer elapsed monotonic milliseconds for the named stage; round down. Absent when unmeasured; no cross-process clock subtraction. |
| `context` | No | Exact `{ operationId, traceId, spanId, parentSpanId? }` shape described here. |
| `links` | No | 1–4 exact `{ component, eventId, traceId?, spanId? }` objects. Trace/span fields are both present or both absent. No caller-defined link labels. |
| `details` | No | Exact event-specific object; absent unless the registry permits it. |

All UUID v4 fields use RFC variant `[89ab]`. Context IDs are locally generated
random values: `operationId` is UUID v4; `traceId` is 32 lowercase hex digits;
`spanId` and optional `parentSpanId` are 16 lowercase hex digits. Trace/span
IDs cannot be all zero. IDs are never metric labels or authorization inputs.
Root spans omit `parentSpanId`; child spans retain their trace and operation.

Within profile v1, producers emit only this exact shape. For an additive future
minor revision, older consumers project their known allowlist and discard new
fields before retention/forwarding; they do not preserve opaque extensions.
Unsupported major versions produce a fixed compatibility status and discard
the event. Real request outcomes remain unchanged. No JSON canonicalization or
signature semantics are borrowed from owner-decision protocols.

## First broker registry

These are proposed mappings, not claims of new instrumentation. Rows with two
listed stages/codes are explicit finite alternatives, never open-ended strings.
The fixed private #167 tokens remain valid in their existing private format.

| Event | Operation | Stage | Code | Outcome | Details |
| --- | --- | --- | --- | --- | --- |
| `broker.request-failure` | `broker.request` | `handler` | `internal` | `failed` | Absent |
| `broker.storage-failure` | `broker.storage` | `append` or `compact` | `storage-failed` | `recovery-required` | Absent |
| `broker.maintenance-failure` | `broker.maintenance` | `maintenance` | `internal` | `failed` | Absent |
| `broker.maintenance-storage` | `broker.maintenance` | `maintenance` | `storage-failed` | `recovery-required` | Absent |
| `transport.unreachable` | `transport.connect` | `connect` | `broker-unreachable` | `failed` | Absent |
| `transport.timeout` | `transport.request` | `deadline` | `broker-timeout` | `failed` | Absent |
| `transport.untrusted` | `transport.connect` | `custody` or `proof` | `broker-untrusted` | `refused` | Absent |
| `transport.retry` | `transport.watch` | `backoff` | `retry-scheduled` | `succeeded` | Exact `{ attempt, delayMs }`; saturating positive safe integer attempt and integer delay 0–10000 |
| `diagnostics.dropped` | `diagnostics` | `queue` | `diagnostics-dropped` | `dropped` | Exact `{ droppedCount }`; saturating nonnegative safe integer |

Client transport events use `agent-comms-client`, not attribution to a broker
that was never reached. Bot/host owners add finite registries from
their operation inventory in the adoption PR. Existing public error codes keep
their meanings. Expected request refusals use their canonical reply/status;
they are not logged as unexpected internal failures. A scheduled retry proves
only scheduling, not reconnection, delivery or eventual success.

## Context ingress and cross-process links

Preferred v1 privacy choice: each untrusted IPC ingress starts local context;
it never persists or exports raw incoming operation/trace/span IDs. Syntax
validation alone cannot prove those strings contain no secret. Incoming
sampling flags, baggage and provider-specific context are discarded.

An authorized response may return a separately reviewed diagnostic reference
containing recipient-generated event/context IDs. The initiator may link its
own observation to those IDs; obtaining a reference grants no inspection or
target authority. Retries retain the initiator's logical operation ID while
each receiver attempt gets its own local context. No new request field is
required by this choice. Older peers without references simply lack the link.

This yields linked trace segments, not a claim of one continuous trace across
every trust boundary. Within a process, optional manual spans retain parentage;
an OTel adapter can map reviewed links without recording incoming baggage.
A later reviewed internal-boundary profile may permit trace continuation; it
must identify the disclosure policy and cannot rely on authentication alone.

Links require a reviewed, locally trusted producer path as well as an authorized
typed interface. Authorization alone does not prove that a compromised or
untrusted producer generated random, safe IDs. References from untrusted peers
are remapped to new local IDs or dropped; their original values are not retained
or exported. Imported support files and arbitrary host input are not trusted
event sources. Restricted principals see only references
and events their existing grants authorize; aggregates cannot reveal hidden
targets. No host scrapes engine logs to obtain links.

## Adoption vectors and next implementation

Use fixed synthetic UUIDs/versions to test exact required/optional fields and
every tuple, with explicit non-claim of cryptographic or identity authority.
Reject extra producer fields, free-form details, wrong tuples, malformed IDs,
all-zero spans, mismatched build provenance, excessive links and 2049-byte
records. Test 2048-byte equality, unknown-major discard, and old-reader
projection of additive fields with secret canaries in those discarded fields.

Cross-process fixtures first prove separately generated contexts and an
authorized returned reference, then retry/deduplication and restricted-principal
visibility. A malicious incoming valid-looking ID must be absent from local,
retained and exported output. This is a prerequisite for reporting the linked
timeline as tested, not a future cleanup.

Next bounded comms implementation adopts local broker/client registry mappings
and private stage durations under the existing sink, with no host field propagation
or new inspection route. A subsequent jointly reviewed interface change adds
authorized references. The
[numeric budgets](diagnostic-resource-budgets.md) and
[acceptance matrix](diagnostic-correlation-acceptance.md) gate each adoption.
