# Proposed diagnostic resource and performance budgets

Concrete default candidates for the
[v1 profile](diagnostic-profile-v1.md), under Proposed
[ENG-0085](../decisions/ENG-0085-local-first-diagnostic-correlation.md).
Repository engineering adoption review ratifies these limits with tests in
adoption PRs; this is not a new QWTS permission gate for routine implementation
or budget choices. This proposal does not change current runtime limits.
Stricter component limits are compatible.

## Finite defaults

Limits apply per process, not per soul, principal, message or trace. Reporting
returns immediately after bounded local work; no diagnostic queue is awaited
by a real operation. Export is disabled by default and initializes only after
explicit configuration/consent. Existing transport retry semantics are unchanged.

| Resource | Proposed cap and overflow behavior |
| --- | --- |
| Complete event | 2048 UTF-8 bytes including newline; discard an oversize event before enqueue. |
| Local pending diagnostics | 64 events and 64 KiB total; one additional event in flight. Drop newest on either bound; no producer backpressure. |
| Local event rate | Token bucket 100 events/second, burst 64; overflow increments drops. Fixed aggregates still count allowed outcomes. |
| Diagnostic writer | At most one helper and one outstanding event per process. No restart loop; failed helper disables that destination. Exit never waits for it. |
| Drop counters | Saturate at `Number.MAX_SAFE_INTEGER`; record queue/rate/discard totals without recursive events. Summary at most once/second after progress. |
| Deduplication | 256 event IDs for at most 15 minutes, oldest eviction. Late duplicates are possible and shown as such. |
| Metric series | 64 pre-registered series, no dynamic labels. IDs, versions, paths, accounts, endpoints and retry numbers are not labels. |
| Duration aggregates | 16 fixed buckets per registered duration metric; integer boundaries 0, 1, 2, 5, 10, 20, 50, 100, 200, 500, 1000, 2000, 5000, 10000, 30000 ms, then overflow. Counts saturate. |
| Trace sampling | Off until enabled locally; then 1% deterministic random-ID sampling, capped at 2 newly sampled operations/second with burst 4. Caller flags cannot override it. |
| Optional inspection ring | Off until authorized interface adoption; at most 128 events, 128 KiB and 15 minutes. Oldest eviction is counted; no implicit disk. |
| Optional exporter queue | 64 events and 64 KiB, one request in flight; maximum body 32 KiB. Drop newest; no unbounded batching or response buffering. |
| Export attempts | Initial attempt plus at most 2 retries, each overall deadline 5 seconds, total lifetime 60 seconds. Backoff 1 then 5 seconds with bounded jitter; no retry after opt-out. |
| Export response | At most 4096 bytes read then discarded; never retained as a diagnostic message. Redirects disabled; configured destination changes require explicit selection. |
| Optional diagnostic disk | Disabled by default. If separately adopted: two 4 MiB segments, total 8 MiB and maximum age 24 hours; prune before writing, never reuse message storage. |
| Local support bundle | Explicit action only: maximum 2 MiB output, 512 events and a selected range no longer than retained evidence. Preview and export share one frozen sanitized snapshot. |

These are separate bounded stores when enabled: a full event queue cannot
borrow inspection or exporter capacity. The existing #167 broker budget of
64 pending events/32 KiB and 512 bytes per event is stricter and need not expand
merely because a shared profile allows more. Its one-helper lifecycle remains
the production baseline for isolated stderr writes.

Oversize/unknown-schema/rate/queue discards update fixed in-memory counters.
When a destination recovers, one bounded summary reports cumulative drops;
a permanently blocked destination cannot carry its own report. Sampling/drop
behavior never removes audit receipts or changes canonical message retention.

Opt-out clears the export queue and prevents scheduling new requests before
returning its effective disabled state. Abort the outstanding request; bound
cleanup by its existing deadline, without delaying real shutdown. Bytes already
transmitted cannot be recalled. Persisted diagnostic export queues are outside
v1; adding one requires a separate disposal/consent design.

## Measured adoption gates

Use the same supported runtime, hardware, workload and storage for paired runs;
record exact versions and configuration. Test normal operation at 100 requests
per second for 60 seconds, then a 1000-event/second diagnostic burst for
10 seconds, with healthy, failed and unread destinations. These rates bound the
fixture and are not product throughput promises.

Compare with diagnostics disabled: p95 request latency increase at most 2 ms,
p99 at most 5 ms, and completed-operation throughput at least 98% of baseline.
Real refusal, delivery/ack, storage recovery and security results must match.
Retained event data must remain within the declared caps after warmup; memory
must plateau under the 10-second burst rather than grow with event count.
Record main/helper RSS and CPU separately. Candidate additional total RSS cap
is 64 MiB including one Node helper; validate and review native macOS and Windows
evidence before making it a supported-platform claim. If a cap fails, fix the design or jointly
revise the recorded budget before adoption, rather than silently excluding the
helper from measurement.

After closing normal broker/daemon/host handles, natural exit completes within
2 seconds under a stalled or failed diagnostic destination. No `process.exit`
substitutes for the test; helpers must be gone. Abrupt-exit tests permit lost
pending observations but no surviving diagnostic service. Packaged host evidence
is separate from source/native engine tests.

Each adoption links the numeric results and platform limitations to
[SOP #85](https://github.com/qwts/qwts-agent-sop/issues/85) and its implementation
epic. #85 remains open until joint profile/budget review and engineering adoption
decisions are recorded; sibling epics retain their own release acceptance.
