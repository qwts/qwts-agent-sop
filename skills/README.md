# Shared agent skills

The fleet's skills home, per
[ENG-0004](../docs/decisions/ENG-0004-centralize-shared-cicd.md) — no per-repo
copies. Skills are agent primitives under
[ENG-0006](../docs/decisions/ENG-0006-agentic-primitives-governance.md): read as
directives, shipped with executable scripts, reviewed as code, and owned in
`.github/CODEOWNERS`. A skill listed here either lives in this repo under
`skills/<name>/` or lives in the repo that owns its domain and is cataloged
here by link — never copied into this tree.

## When an agent loads a skill

Do not read this file at session start, and do not install these skills into a harness. A harness injects every installed skill's name and description into every turn.

A procedure names one skill. Read that entry below and then its `SKILL.md` at the pinned commit. Leave the other entries unread. A branch or a tag is not a pin. The commit is the `org.json` capability pin when that repository is a capability, and a commit in this file otherwise. Links to the owning repository are not pins.

## Available skills

- [agent-bot](https://github.com/qwts/agent-bot-identity/tree/7e1f813347e49df78437098c5415269d8423bc72/skills/agent-bot)
  — owned by
  [qwts/agent-bot-identity](https://github.com/qwts/agent-bot-identity).
  Per-harness GitHub App identities for coding agents: bootstrap and
  installation, bot credential minting, authorized secure-store reads,
  GitHub-verified bot commits, transcript-bound Agent IDs, and Agent Spaces.
  Absorbed the old signed-commit skill (`signed-commit.mjs`). Load only when a procedure names this skill.
- [managed-machine](https://github.com/qwts/managed-machine/tree/32765719ed8bdeea21223366ed8435670e21b47b/skills/managed-machine)
  — owned by
  [qwts/managed-machine](https://github.com/qwts/managed-machine). Bootstrap,
  update, and manage a Mac via the `managed-machine` Homebrew formula: fresh
  setup, version reporting, setup scripts, brew ownership fixes, fleet SSH
  keys, gitleaks hooks, and agent-CLI installs. Load only when a procedure names this skill.
- [onboard-harness](https://github.com/qwts/managed-machine/tree/32765719ed8bdeea21223366ed8435670e21b47b/skills/onboard-harness)
  — owned by
  [qwts/managed-machine](https://github.com/qwts/managed-machine).
  Add a harness to managed-machine and managed-machine-config: catalog rows,
  setup scripts, tests, and docs. Load only when a procedure names this skill.
- [add-zsh-function](https://github.com/qwts/zsh-functions/tree/c48302c3e9107745030b6fb8fee7805fc66eacf4/skills/add-zsh-function)
  — owned by
  [qwts/zsh-functions](https://github.com/qwts/zsh-functions).
  Author a new zsh function: `functions/<name>` per that repo's `AGENTS.md`,
  reuse of the shared PATH/fpath API catalog, formula and `v*` tag release.
  Load only when a procedure names this skill.
- [migrate-to-zsh-functions](https://github.com/qwts/zsh-functions/tree/c48302c3e9107745030b6fb8fee7805fc66eacf4/skills/migrate-to-zsh-functions)
  — owned by
  [qwts/zsh-functions](https://github.com/qwts/zsh-functions).
  Convert legacy `~/.functions` loops, vendor PATH leaks, and unguarded
  `export PATH` lines into guarded `BEGIN/END zsh-functions` blocks plus API
  calls. Load only when a procedure names this skill.
- [audit-shell-writers](https://github.com/qwts/zsh-functions/tree/c48302c3e9107745030b6fb8fee7805fc66eacf4/skills/audit-shell-writers)
  — owned by
  [qwts/zsh-functions](https://github.com/qwts/zsh-functions).
  Read-only recon of who writes `.zshenv`/`.zprofile`/`.zshrc` across repos;
  run before migrating. Load only when a procedure names this skill.

The signed-commit skill previously lived here. A machine that installed it
from this repo has a dangling symlink; remove it:

```bash
rm -f ~/.claude/skills/signed-commit
```

## Adding a skill

1. `skills/<name>/SKILL.md` with `name` and `description` frontmatter. The
   description stays in that file. It is not injected into a session. The
   procedure that needs the skill names it; that name is the trigger
   ([ENG-0006](../docs/decisions/ENG-0006-agentic-primitives-governance.md)).
2. Link it from **Available skills** above. `docs-gov` fails a skill reachable
   from no index: guidance nothing links to is guidance no agent loads.
3. Keep it under the `perDoc` token budget. A skill that needs more is usually
   two skills.
4. State what an agent cannot derive from the code or existing docs, and link
   rather than restate — a shared fact in two agent files is a bug
   (ENG-0006 item 1).
5. Treat any script it ships as the supply chain it is: no secrets, no network
   fetch of unpinned code, least privilege.
