# SOP: Machine and shell setup

Which skill a machine task loads. This is not a session preamble. Read it
only when the task changes a Mac, adds a harness, or edits shell startup.
Then load the one skill named for that task from the
[skill catalog](../../skills/README.md), at that entry's commit. Do not read
the other entries, and do not install the skill into the harness.

## A Mac (mandatory — extend, don't drop)

Bootstrap, update, brew ownership, fleet SSH keys, gitleaks hooks, and
agent-CLI installs load `managed-machine`.

Adding a harness to managed-machine and managed-machine-config loads
`onboard-harness` instead. Do not load `managed-machine` for that task
unless the harness procedure tells you to.

## Shell startup (mandatory — extend, don't drop)

Repairing who writes `.zshenv`, `.zprofile`, or `.zshrc` loads
`audit-shell-writers` first. Load `migrate-to-zsh-functions` only if that
audit says to migrate. Do not load both at the start.

Authoring a new function in `qwts/zsh-functions` loads `add-zsh-function`.
That task does not load the audit or the migration skill.
