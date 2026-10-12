# Diagnostic correlation operation inventory and acceptance plan

This is the review plan for Proposed
[ENG-0085](../decisions/ENG-0085-local-first-diagnostic-correlation.md), tracked
in [SOP #85](https://github.com/qwts/qwts-agent-sop/issues/85). It is not a
runtime schema or interoperability claim. Related epics
[comms #164](https://github.com/qwts/agent-comms/issues/164),
[bot #983](https://github.com/qwts/agent-bot-identity/issues/983) and
[host #370](https://github.com/true-line/geniusbar/issues/370) remain open.

## Observed broker boundaries

Inventory baseline: agent-comms merge
[`a9a53686`](https://github.com/qwts/agent-comms/commit/a9a53686ef9e6c54048686a95ae1eba355f78f21).
Rows identify candidate observations, not delivered instrumentation. Existing
request errors and canonical records remain authoritative.

| Boundary and source | Observation opportunity | Distinction to preserve |
| --- | --- | --- |
| [Client open/watch](https://github.com/qwts/agent-comms/blob/a9a53686ef9e6c54048686a95ae1eba355f78f21/lib/client.mjs) | Custody check, connect, request deadline, reply, stream disconnect and retry/backoff | Only unreachable/timeout watch failures retry; custody/refusal needs intervention. Deadline differs from idle timeout. |
| [Local channel](https://github.com/qwts/agent-comms/blob/a9a53686ef9e6c54048686a95ae1eba355f78f21/lib/platform/local-channel.mjs) | Listen, Windows broker proof, protocol validation, handler, response and close | Connectivity and broker authenticity differ; observation cannot bypass either check. |
| [Broker lifecycle](https://github.com/qwts/agent-comms/blob/a9a53686ef9e6c54048686a95ae1eba355f78f21/lib/broker.mjs) | Prepare, replay, listen, maintenance, uncertain append/compact and shutdown | Storage failure poisons writes until recovery; a diagnostic cannot make storage safe. |
| [Mailbox](https://github.com/qwts/agent-comms/blob/a9a53686ef9e6c54048686a95ae1eba355f78f21/lib/broker/mailbox.mjs) | Admission/dedupe, durable send, authorized read and acknowledgment | Attempt, durable acceptance and acknowledgment differ; acknowledgment is not task completion. |
| [Watch delivery](https://github.com/qwts/agent-comms/blob/a9a53686ef9e6c54048686a95ae1eba355f78f21/lib/broker/watch.mjs) | Live write, wake coalescing/rate limit, pending release and connection loss | Socket write or wake report does not prove receipt or execution. |
| [Health/census](https://github.com/qwts/agent-comms/blob/a9a53686ef9e6c54048686a95ae1eba355f78f21/lib/broker/census.mjs) | Authorized status and bounded aggregate presentation | Census filters before building rows; new aggregates need explicit visibility policy. |
| [Launch bridge](https://github.com/qwts/agent-comms/blob/a9a53686ef9e6c54048686a95ae1eba355f78f21/lib/broker/launch.mjs) | Authorized request, daemon progress/result and timeout | Broker observes transport/progress; daemon owns execution facts. |
| [A2A outbound](https://github.com/qwts/agent-comms/blob/a9a53686ef9e6c54048686a95ae1eba355f78f21/lib/broker/a2a-outbound.mjs) | Attempt, timeout, bounded retry, reconciliation and shutdown drain | Uncertain remote state is not safe retry or completion. No remote bodies or destinations captured. |

The private sink covers unexpected request, storage and maintenance failures
only. Its event ID identifies an observation, not a cross-process operation.
Existing native Windows and main-CI evidence applies to that slice only.

## Sibling boundaries requiring owner confirmation

Bot #983 proposes launch/resume/harness-exit, approval-result, runtime-selection,
credential-provider category, refresh and shutdown observations, building on
existing history and launch stages. These are issue requirements, not a source
audit or delivery claim. Bot review identifies exact routes, safe codes and
execution-versus-transport event ownership.

Host #370 describes web UI, Rust/Tauri shell and bundled Node bridge. Its
owners inventory UI-to-shell, shell-to-bridge and authorized engine calls.
Version metadata reuses host #290: bundled differs from actually running, with
unknown/disconnected states. Provider evaluation, symbols, UI and packaged
support stay in that repository.

## Joint-review choices before public adoption

1. Approve field types, finite code/details vocabularies, major compatibility
   and concrete JSON/IPC placement without accidentally widening request
   parsers. Specify older-peer and unsupported-major behavior.
2. Define context ingress remapping, outgoing span parentage, retries, restart
   discontinuity, external linking and local sampling. Valid-looking incoming
   IDs may encode secrets; validation alone is not disclosure approval. No
   baggage or opaque provider strings are forwarded.
3. Assign one producer per failure and specify forwarding/deduplication,
   finite cache eviction and host-related events.
4. Approve inspection/health/export visibility; restricted principals must not
   learn hidden entities through aggregates.
5. Record numeric resource/performance budgets, exporter dependency strategy,
   destination/credential custody, opt-out cancellation and queued-data
   disposal. No provider is selected by this plan.

## Acceptance matrix

Record exact component versions/commits, configuration, platform, expected
results, sanitized observations and measured limits for each scenario. Source
tests, native source runs and packaged runs are separate evidence.

| Scenario | Required evidence |
| --- | --- |
| Standalone broker and bot without host | Useful fixed code/stage/version and local timeline; no reporting account, collector or network. |
| Host refresh/delivery across supported IPC | One operation, distinct producer events and correct causal links; forwarding retains event identity. Canonical success/refusal unchanged. |
| Retry, duplicate reply, disconnect and restart | Attempts distinct, bounded dedupe, process boundary shown. No completion inferred from reconnect or socket write. |
| Moved bundle, mismatch and startup-before-ready failure | Correct component, bundled/running/unknown versions and safe outcome; no path or loader-stack disclosure. |
| Invalid, oversized, unsupported or adversarial context | Operation/authentication unchanged; bounded parsing; no baggage, sample override, secret-bearing valid IDs or unknown fields stored/forwarded. |
| Canaries in real failure paths | Errors/causes, requests, paths, environment and payloads tested; absent from local output, persistence, bundles and export. No private-state scraping. |
| Refusal and restricted-principal inspection | Refusal distinct from defect; IDs grant no authority; unauthorized rows/counts/details hidden. |
| Export disabled, offline, stalled or failing | No telemetry network before opt-in; real delivery/security and natural shutdown unchanged; finite queues/retries/memory and visible drops. |
| Opt-out with queued/in-flight reports | No subsequent sends, finite cancellation, explicit disposal/retention and truthful setting; transmitted data cannot be recalled. |
| Abrupt exit, blocked destination and saturation | No diagnostic crash/event-loop stall; finite in-flight work, helper cleanup, no recursive errors; incomplete evidence shown as incomplete. |
| Local support export | Authorized time/byte limits; preview matches sanitized output; failure truthful; no implied upload, attachment or public issue. |
| Canonical state and audit invariance | Diagnostics absent/sampled/dropped yields identical delivery/ack/task/security outcomes and retention/recovery semantics. |

Native macOS and Windows evidence is required for adopted platform behavior.
Host packaged tests do not establish engine standalone support; engine source
tests do not establish host packaging support.

## Independent work and staged implementation

Before joint acceptance, owners can improve inventories and build failure and
performance fixtures without public fields. Comms prioritizes deadline/retry,
abrupt broker restart and blocked destinations. Compare disabled and saturated
runs for latency, memory, retained handles and exit; publish numeric budgets
before claiming acceptance.

After review, implement a small lifecycle/transport subset, then authorized
local health/inspection, then optional reporting under the approved dependency
strategy. Persisting correlation across restarts or adding diagnostic disk files
requires retention/privacy review rather than reusing message state. Release
evidence links adoption ADRs, interface vectors and unresolved limits. Neither
this proposal nor the private sink completes a sibling epic.
