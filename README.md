# qwts-agent-sop

The `qwts` instance of [agent-sop](https://github.com/qwts/agent-sop) at commit [`bf072f7`](https://github.com/qwts/agent-sop/tree/bf072f7ab2ed077af781fdc3f754425c43f197c5), plus the [deltas recorded below](#deltas-from-the-template): the rules, procedures, and guides the organization follows — the ENG decision series, the shared SOPs, the org-wide agent conventions, the shared skills catalog, and the SDLC guides. The mechanisms the template used to host — shared CI, the docs-governance gate, the dependency inventory, and the Copilot SDLC chain — live in [capability repositories](#capability-repositories), each consumed at a pinned commit ([ENG-0355](docs/decisions/ENG-0355-static-router-one-pointer-pinned-capabilities.md)); the organization's own data — the governed-repos manifest, the App roster, the organization profile — lives in [qwts-agent-org](https://github.com/qwts/qwts-agent-org), whose `org.json` pins this repository as the SOP source. Imported from `agent-sop` with its Git history preserved; see the [repository migration contract](docs/reference/repository-migration.md).

It is also the home for **cross-repo engineering decisions** — see the [decision index](docs/decisions/README.md) — and for the org-wide agent conventions every repo's [AGENTS.md](AGENTS.md) points to; see [AGENTS.md](AGENTS.md) for this repo's own agent context.

## Engineering decisions (ENG series)

Durable records for decisions that span more than one repository: tooling
direction, shared conventions, where things live, language and platform choices.

### 📐 [Decision index](docs/decisions/README.md)

Decisions owned by a single repository stay in that repository. The routing test
is simple: **if exactly one repo would have to change, it is not an ENG record.**

## Documentation Structure

The `docs/` folder contains 22 comprehensive guides covering the complete Software Development Life Cycle (SDLC), organized by phase:

### 📋 [Complete Documentation Index](docs/00-documentation_index.md)

**Planning Phase** (Documents 1-6): Requirements, architecture, security, and technology decisions
- [Requirements Gathering](docs/01-requirements_gathering.md)
- [Technology Selection & PoC](docs/02-technology_selection_and_poc.md)
- [Data Governance & Strategy](docs/03-data_governance_and_strategy.md)
- [Security & Compliance Planning](docs/04-security_and_compliance_planning.md)
- [Testing Strategy](docs/05-testing_strategy.md)
- [Architecture Planning](docs/06-architecture_planning.md)

**Development Phase** (Documents 7-16): Infrastructure, deployment, and technical implementation
- [Project Structure Planning](docs/07-project_structure_planning.md)
- [Infrastructure Guidelines](docs/08-infrastructure_guidelines.md)
- [Compute Selection](docs/09-compute_selection.md)
- [Database & Storage Planning](docs/10-database_and_storage_planning.md)
- [Networking & Load Balancing](docs/11-networking_and_load_balancing.md)
- [Observability Stack Planning](docs/12-observability_stack_planning.md)
- [CI/CD Planning](docs/13-cicd_planning.md)
- [Disaster Recovery Planning](docs/14-disaster_recovery_planning.md)
- [Cost Optimization & FinOps](docs/15-cost_optimization_and_finops.md)
- [Performance & Optimization Planning](docs/16-performance_and_optimization_planning.md)

**Operations Phase** (Documents 17-22): Launch preparation, operations, and project completion
- [UAT & Pilot](docs/17-uat_and_pilot.md)
- [Final Validations](docs/18-final_validations.md)
- [End User Training & Change Management](docs/19-end_user_training_and_change_management.md)
- [Launch Checklist](docs/20-launch_checklist.md)
- [Post-Launch Operations](docs/21-post_launch_operations.md)
- [Decommissioning & Retirement](docs/22-decommissioning_and_retirement.md)

Each document includes navigation links, prerequisites, and cross-references to related topics. Use these guides to align on best practices, ensure consistency, and drive quality in your projects.

## Shared standards and tooling

- [Shared SOPs](docs/sop/README.md) — org-wide standard operating procedures for how work moves, inherited by every repo (ENG-0008).
- [Org-wide agent conventions](docs/reference/agent-conventions.md) — the shared agent working agreement every repo's `AGENTS.md` links to (ENG-0006).
- [Agent bot identity governance](docs/reference/agent-bot-identity.md) — the qwts App roster, permissions, coverage, and integration contract (ENG-0016, ENG-0128).
- [Agent execution identity policy](docs/reference/agent-execution-identity.md) — the private transcript-bound identity and audit boundary behind each agent conversation (ENG-0081).
- [Agentic primitives conformance checklist](docs/reference/agentic-primitives-conformance-checklist.md) — the ENG-0006 §6 checklist per-repo alignment issues link to.
- [Machine memory guard retirement](docs/reference/agent-memory-guard.md) — historical decision and source; the implementation and dormant backlog are retired.
- [Shared agent skills](skills/README.md) — skills centralized here and installed into every agent harness, rather than copied per repo (ENG-0004, ENG-0006).
- [Hook composition audits](docs/reference/hook-composition-audits.md) — fleet snapshots of repository-owned commands inside the managed hook adapters.
- [Dependency reuse policy](docs/reference/dependency-reuse-policy.md) — the ENG-0269 cache contract every consumer of the shared `bounded-dependency-install` action follows.
- [Repository migration](docs/reference/repository-migration.md) — how this instance was imported, what it pins, and the CI bootstrap delta it carries.
- [Documentation style guide](docs/23-documentation_style_guide.md) — conventions for writing docs in this playbook.
- [Contributing](CONTRIBUTING.md) — how changes to this repository land.

## Organization data

The qwts account's own state lives in [qwts-agent-org](https://github.com/qwts/qwts-agent-org) at [`18ce9fd`](https://github.com/qwts/qwts-agent-org/tree/18ce9fd3d8d4df20e3846eb7c59c42e464a7cf9f), the revision these links pin: the [governed repositories](https://github.com/qwts/qwts-agent-org/blob/18ce9fd3d8d4df20e3846eb7c59c42e464a7cf9f/docs/governed-repos.md) table generated from `governance/repos.json` (ENG-0011), the [GitHub account reference](https://github.com/qwts/qwts-agent-org/blob/18ce9fd3d8d4df20e3846eb7c59c42e464a7cf9f/docs/github-account.md), the App roster `governance/agents.json`, and the organization profile. Each moved out of this repository under ENG-0355; the last revision holding them here is [`8e9b32f`](https://github.com/qwts/qwts-agent-sop/tree/8e9b32fba1dad1147d36c5f0a11cca1fc0a7535a).

## Capability repositories

Each mechanism lives in its own repository and is consumed at a 40-hex commit, never a branch or tag. The pins below are the ones this repository's own CI and docs use; `qwts-agent-org`'s `org.json` records the same pins. Every link goes to the pinned revision.

- [qwts-agent-ci](https://github.com/qwts/qwts-agent-ci/tree/3a5617b287d922e37f262210a1d8750d8217b56d) at `3a5617b` — shared CI (ENG-0004, ENG-0267, ENG-0269): the composite actions `ci-policy`, `bounded-command`, `bounded-dependency-install`, `changeset-release-count`, and `ci-runtime-check`, the runtime-policy checker, the pin-reachability guard, and the [CI execution policy](https://github.com/qwts/qwts-agent-ci/blob/3a5617b287d922e37f262210a1d8750d8217b56d/docs/ci-execution-policy.md), [CI runtime budgets](https://github.com/qwts/qwts-agent-ci/blob/3a5617b287d922e37f262210a1d8750d8217b56d/docs/ci-runtime-policy.md), [governed CI rollout checklist](https://github.com/qwts/qwts-agent-ci/blob/3a5617b287d922e37f262210a1d8750d8217b56d/docs/governed-ci-rollout.md), and [release-lifecycle fleet handoff](https://github.com/qwts/qwts-agent-ci/blob/3a5617b287d922e37f262210a1d8750d8217b56d/docs/governed-ci-release-lifecycle-fleet.md).
- [qwts-agent-docs-gov](https://github.com/qwts/qwts-agent-docs-gov/tree/67db7dc9c20bc29222fb605b7ff9432fd58a2a3f) at `67db7dc` — the `docs-gov` gate and its reusable workflow, plus the on-demand `docs-eval` loop (ENG-0009, ENG-0010): [documentation governance](https://github.com/qwts/qwts-agent-docs-gov/blob/67db7dc9c20bc29222fb605b7ff9432fd58a2a3f/docs/documentation-governance.md) and [docs evaluation](https://github.com/qwts/qwts-agent-docs-gov/blob/67db7dc9c20bc29222fb605b7ff9432fd58a2a3f/docs/docs-evaluation.md). This repository's own configuration is `docs-gov.config.json`.
- [qwts-agent-inventory](https://github.com/qwts/qwts-agent-inventory/tree/d5746df21099c0394663b35dd16eacd171052a80) at `d5746df` — the report-only [dependency & tooling inventory](https://github.com/qwts/qwts-agent-inventory/blob/d5746df21099c0394663b35dd16eacd171052a80/docs/dependency-inventory.md), its reusable workflow, and the weekly fleet catalog (ENG-0015). This repository's own configuration is `dependency-inventory.config.json`; `inventory-defaults.config.json` is the fallback the fleet catalog applies to a repo without one.
- [qwts-agent-sdlc](https://github.com/qwts/qwts-agent-sdlc/tree/9168b22ad2a7c71938ae12c1c412753773887f04) at `9168b22` — the VS Code Copilot SDLC chain that walks the guides above: seven custom agents, 29 slash-command prompts, the Copilot instructions file, and the [usage guide](https://github.com/qwts/qwts-agent-sdlc/blob/9168b22ad2a7c71938ae12c1c412753773887f04/docs/usage.md).
- [qwts/agentic-code-analysis](https://github.com/qwts/agentic-code-analysis) — the advisory semantic-ratchet workflow of ENG-0160 moves there; the last revision of its reference document in this repository is [semantic-ratchets.md at `8e9b32f`](https://github.com/qwts/qwts-agent-sop/blob/8e9b32fba1dad1147d36c5f0a11cca1fc0a7535a/docs/reference/semantic-ratchets.md).

## Deltas from the template

What this instance carries beyond `agent-sop@bf072f7`, so the next alignment knows what to keep:

- Repository identity: `qwts-agent-sop` in `package.json`, the issue form, and every `qwts-agent-sop` link in the governed discovery block of [AGENTS.md](AGENTS.md); the `decisionSeries` home stays `qwts/agent-sop`, where the ENG issue numbers were allocated.
- The [repository migration](docs/reference/repository-migration.md) reference and the CI bootstrap delta it describes: the Action Policy job runs `ci-policy` in `authorization-only` mode and selects full validation itself, because the pinned `qwts-agent-ci` catalog has no lifecycle entry for this repository yet.
- The report-only dependency inventory stays a required lane of this repository's own CI, from `qwts-agent-inventory` at the pin above, with its configuration files kept here.
- `governance/release-lifecycles.json` stays until `qwts-agent-org` or `qwts-agent-ci` carries this repository's entry; nothing here reads it.
- Decision records that the template links to `agent-sop@ed5c5d8` link here to this repository's own last revision holding the moved files, `8e9b32f`, each with a one-line note on the new home.
- Not yet imported from the template: [ENG-0375](https://github.com/qwts/agent-sop/blob/bf072f7ab2ed077af781fdc3f754425c43f197c5/docs/decisions/ENG-0375-owner-account-agent-context-is-the-delegate.md), which supersedes ENG-0353; ENG-0353 stays `Proposed` here until that record is brought over.

## Usage
1. **[Usage guide](https://github.com/qwts/qwts-agent-sdlc/blob/9168b22ad2a7c71938ae12c1c412753773887f04/docs/usage.md)** — VS Code Copilot agents, slash commands, and workflows for interactive requirements gathering, from `qwts-agent-sdlc`.
2. Browse the `docs/` directory to find relevant sections of the SDLC.
3. Share and adapt the workflows for your team or project.
4. Keep the repository up to date with new insights and improvements.

> This repository is intended to evolve as a living playbook for engineering excellence.
