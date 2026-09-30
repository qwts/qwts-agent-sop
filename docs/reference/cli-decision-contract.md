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
  schema;
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
4. Checks the provider's registry entry and declared capabilities, then sends
   only the disclosed fields within a timeout and cost limit.
5. Validates the returned answer and its uncertainty against the policy.
6. Builds the proposal from the template and validated values, as an argument
   array with no shell interpolation.
7. Returns the proposal. The caller runs the existing command separately,
   under its normal permissions and confirmation rules.

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
  "status": "proposed",
  "question": "export-format@1",
  "state_revision": "orders:42",
  "provider": {"id": "typesafe", "model": "jev-1.13.0"},
  "answer": "csv",
  "proposal": {
    "action_id": "export-preview",
    "argv": ["exapp", "export", "orders", "--format", "csv", "--dry-run"],
    "preconditions": {"dataset_revision": "42"}
  }
}
```

## Initial registry candidates

| Candidate | Status | Notes |
| --- | --- | --- |
| TypeSafe Jev (`jev-latest`) | `seeded` | Map the portable contract to `choice`. Expose `score` and `noul` only through a later versioned extension with a conversion and abstention policy. Text only. |
| OpenAI Decisions API (GPT-6 Luna) | `unverified` | Unavailable until the public request format and uncertainty output are verified. Do not invent endpoints or confidence fields. Function calling and routers are not substitutes. |

## Acceptance criteria

- Definitions are schema-validated, with fixtures for the normal case,
  missing state, adversarial input, and abstention.
- `decide list` and `decide show` work offline without credentials.
- Adapter tests preserve capabilities and uncertainty. Invalid answers,
  outages, unsupported inputs, and exhausted budgets produce no proposal.
- Tests cover disclosed-fields-only transmission, refusal of unverified or
  absent providers, no implicit fallback, deterministic mapping, validated
  arguments, and state revalidation.
- A decision never changes permissions or bypasses confirmation, and `skill`
  commands never require a provider.

## Open questions

- **Registry home:** a new `governance/decision-providers.json` beside
  ENG-0151's registry, or an extension of it.
- **Dynamic questions:** whether callers may supply a typed question within a
  declared extension boundary, validated before transmission.
- **Numeric answer types:** whether `score` or `noul` may ever select an
  action, and under what versioned mapping.
- **Data-handling approval:** who approves a provider's terms before its
  status becomes `verified`.
