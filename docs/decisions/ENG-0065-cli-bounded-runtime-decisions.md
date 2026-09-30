# ENG-0065: CLIs may offer bounded, model-backed decisions that return proposals, never actions

**Status:** Proposed
**Date:** 2026-09-30
**Issue:** qwts/qwts-agent-sop#65

## Context

Some choices depend on the application's current state and on domain
knowledge the calling agent does not have: which export format fits a
consumer, which queue a request belongs in, which workflow comes next. The
agent guesses from partial information, or the CLI hard-codes rules that do
not cover the case.

A class of provider now answers a question by choosing from a fixed set of
answers, rather than by generating text. TypeSafe's
[System One API](https://docs.typesafe.ai/api) (model Jev) evaluates state
against typed questions: `choice`, `noul` (yes/no probability), and `score`
(rubric). `choice` and `score` return a distribution and a derived
confidence; `noul` returns no confidence field; input is text or structured
state, not images. OpenAI's Decisions API, announced 2026-09-29, has GPT-6
Luna pick one of the answers you supply from text or image context. As of
2026-09-30 it is in limited preview, and its request syntax, uncertainty
output, and price are unpublished.

[ENG-0064](ENG-0064-cli-skill-command-family.md)'s `skill` commands are
offline and read-only. A decision sends application state to a third party,
with credentials, latency, and cost. That is a different risk, so it is a
separate record.

## Decision

The contract details (commands, definition fields, flow, results, provider
candidates, acceptance criteria) are in the
[CLI decision contract](../reference/cli-decision-contract.md).

1. **Optional, per CLI.** A CLI offers decisions only where deterministic code
   cannot make the choice reliably. Neither ENG-0055 nor ENG-0064 requires it.
2. **The model picks an answer; the application does everything else.** The
   application owns context selection, the allowed answers, the mapping from
   answer to action, and validation. The model returns one answer ID from a
   finite set that includes abstention. The application builds the proposal
   as an argument array from an allowlisted template. A decision grants no
   permission and executes nothing. Model output never becomes a shell
   command.
3. **Only predefined questions.** Each question is defined in the CLI source,
   versioned, tested, and shipped in the release. It declares every field
   that may leave the machine. Definitions are application data, not part of
   the skill bundle.
4. **Providers are data, not decisions.** This record fixes the adapter
   contract: finite-answer selection, declared capabilities, and provenance.
   The allowed providers and models live in a registry, as
   [ENG-0151](ENG-0151-model-routing.md) does for model routing, each with a
   status (`verified`, `seeded`, `unverified`) and its data-handling terms.
   The CLI refuses an `unverified` or absent provider. A new provider changes
   the registry, not this record.
5. **Transmission needs authorization, not just credentials.** `decide ask`
   requires an explicit `--provider`; there is no default. The CLI sends only
   the question's declared fields, minimizes and redacts them, and never
   sends secrets. Request or provider content cannot change a definition, add
   actions, or widen data access.
6. **Failure never degrades silently.** Timeouts, retries, and spending are
   bounded. An outage never switches provider or model. Missing state,
   abstention, insufficient certainty, or provider failure returns
   `needs-input` with no proposal. The CLI passes the provider's uncertainty
   through and never manufactures a confidence.
7. **Revalidate before execution.** A proposal carries its state revision.
   The command that runs it revalidates target, arguments, and preconditions;
   changed state invalidates the proposal.

## Consequences

- **Choices that need domain knowledge get a bounded, auditable path.** Each
  proposal traces to a versioned definition, an answer, and a template.
- **Real costs:** provider adapters, per-question evaluations, latency,
  disclosure review, and failure handling. They are why the interface is
  optional.
- **Uncertainty is not comparable across providers.** Thresholds are tuned
  per question from evaluations, not set universally.
- **Logs can leak what was disclosed.** The CLI logs versions, outcomes, and
  redacted provenance, not the context sent to a provider.
- **A registry to build.** Until it exists, no provider is `verified` and
  `decide ask` refuses every call.

## Alternatives

- **Put decisions in ENG-0064:** rejected. It ties an offline read interface
  to a networked, credentialed one that needs separate review.
- **Name providers in the record:** rejected. A vendor change would need a
  new record (ENG-0151 decision 3).
- **Let the agent decide from the skill alone:** kept as the default; this
  interface is for choices that need state the agent lacks.
- **General text generation or function calling:** rejected. Its output is
  unbounded and would need parsing into actions.
- **Caller-defined dynamic questions now:** deferred until predefined
  questions have evaluation history.

## References

- [CLI decision contract](../reference/cli-decision-contract.md) ·
  [CLI skill contract](../reference/cli-skill-contract.md)
- [ENG-0064](ENG-0064-cli-skill-command-family.md) ·
  [ENG-0055](ENG-0055-every-cli-ships-its-agent-skill.md) ·
  [ENG-0151](ENG-0151-model-routing.md)
- [TypeSafe API](https://docs.typesafe.ai/api) ·
  [OpenAI DevDay 2026 recap](https://openai.com/index/devday-2026-recap/)
