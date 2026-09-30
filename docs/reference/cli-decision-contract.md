# CLI decision contract

The contract behind
[ENG-0065](../decisions/ENG-0065-cli-bounded-runtime-decisions.md): how a CLI
offers bounded, model-backed decisions. Output conventions are in the
[CLI skill contract](cli-skill-contract.md); this document does not restate
them. The examples use `exapp` as the CLI.

## Commands

```sh
exapp decide list                  # question IDs, versions, answers (offline)
exapp decide show export-format@1  # one question's definition (offline)
exapp decide ask export-format@1 --request request.json --provider <id> --json
```

`list` and `show` make no network call and need no credentials. `ask`
requires `--provider`.

## Question definitions

A definition is versioned application data in the CLI source. It holds:

- the question ID and criteria;
- the state schema and the context sources the application may add;
- the **disclosed fields**, which are every field that may leave the machine;
- finite answer IDs, including abstention;
- an allowlisted mapping from each answer to an action template and argument
  schema, where every target command enforces the emitted preconditions,
  including the state revision;
- preconditions, expected effects, and required confirmation;
- the uncertainty policy and test fixtures.

An exported skill never carries or activates a definition.

## Flow

For `ask`, the CLI:

1. Validates the request against the state schema and size limits.
2. Adds domain context with its existing read permissions, keeping caller
   input apart from authoritative application facts.
3. Applies deterministic eligibility rules in code and narrows the allowed
   answers to the eligible ones.
4. Refuses unless the provider's registry entry is `verified` and its
   declared capabilities cover the question, then sends only the disclosed
   fields within a timeout and cost limit.
5. Validates the returned answer and its uncertainty against the policy.
6. Builds the proposal from the template and validated values, as an argument
   array with no shell interpolation that always includes the precondition
   arguments.
7. Returns the proposal. The caller runs the existing command separately,
   under its normal permissions and confirmation rules, and that command
   refuses if a precondition no longer holds.

## Results

One JSON document, backward compatible within a major version (ENG-0055
decision 5). It holds the schema version, question ID and definition version,
request ID, state revision, provider and model identity as reported, a
`status`, the accepted answer, and the provider's uncertainty data as
returned.

- `proposed`: a proposal names the action ID, argument array, preconditions,
  expected effects, and required authorization. Exit 0.
- `no-action`: a valid outcome, not an outage. Exit 0.
- `needs-input`: missing state, abstention, insufficient certainty, an
  exhausted budget, or a provider failure, with a stable `code` and a safe
  next step. No proposal. Documented non-zero exit.
- `error`: a malformed request or contract violation. Documented non-zero
  exit.

## Example

```json
{
  "schema_version": 1,
  "request_id": "req_7f3a",
  "question": {"id": "export-format", "version": 1},
  "state_revision": "orders:42",
  "provider": {"id": "typesafe", "model": "jev-1.13.0", "registry_status": "verified"},
  "status": "proposed",
  "answer": "csv",
  "uncertainty": {"distribution": {"csv": 0.91, "json": 0.07, "abstain": 0.02}, "confidence": 0.84},
  "proposal": {
    "action_id": "export-preview",
    "argv": ["exapp", "export", "orders", "--format", "csv", "--dry-run", "--if-revision", "orders:42"],
    "preconditions": {"state_revision": "orders:42"},
    "effects": "local-write: writes a preview file; no remote change",
    "authorization": "none beyond the caller's; no confirmation for --dry-run"
  }
}
```

`uncertainty` is the provider's data as returned; the values are
illustrative.

## Initial registry candidates

| Candidate | Status | Notes |
| --- | --- | --- |
| TypeSafe Jev (`jev-latest`) | `seeded`: documentation only until its data-handling terms are approved | Map the portable contract to `choice`. Expose `score` and `noul` only through a later versioned extension with a conversion and abstention policy. Text only. |
| OpenAI Decisions API (GPT-6 Luna) | `unverified` | Unavailable until the public request format and uncertainty output are verified. Do not invent endpoints or confidence fields. Function calling and routers are not substitutes. |

## Acceptance criteria

- Definitions are schema-validated, with fixtures for the normal case,
  missing state, adversarial input, and abstention.
- `decide list` and `decide show` work offline without credentials.
- Adapter tests preserve capabilities and uncertainty. Invalid answers,
  outages, unsupported inputs, and exhausted budgets produce no proposal.
- Tests cover disclosed-fields-only transmission, refusal of `seeded`,
  `unverified`, and absent providers, no implicit fallback, deterministic
  mapping, validated arguments, and a stale-revision refusal by the target
  command when `argv` is run as returned.
- A decision never changes permissions or bypasses confirmation, and `skill`
  commands never require a provider.

## Open questions

- **Registry access at runtime:** the registry is a separate
  `governance/decision-providers.json` beside ENG-0151's
  `agent-models.json`; whether a CLI reads it from the governed checkout or
  from a pinned copy in its release is open.
- **Dynamic questions:** whether callers may supply a typed question within a
  declared extension boundary, validated before transmission.
- **Numeric answer types:** whether `score` or `noul` may ever select an
  action, and under what versioned mapping.
- **Data-handling approver:** who reviews a provider's terms, since approval
  is required before its status becomes `verified`.
