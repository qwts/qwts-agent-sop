# CLI skill command contract

The command contract behind
[ENG-0064](../decisions/ENG-0064-cli-skill-command-family.md): how a CLI
exposes its [ENG-0055](../decisions/ENG-0055-every-cli-ships-its-agent-skill.md)
bundle to agents. Output conventions, metadata, and the release gate are in
the [CLI skill contract](cli-skill-contract.md); this document does not
restate them. The examples use `exapp` as the CLI and `sync` as a feature.

## Bundle layout

```text
skills/exapp/
├── SKILL.md              # router, qwts-* metadata
├── references/
│   ├── sync.md
│   └── restore.md
├── assets/               # optional: templates, data
└── scripts/              # optional: documented, never run by skill commands
```

Feature identifiers use lowercase letters, digits, and single hyphens. The
CLI source holds one catalog mapping each identifier to its reference file
and a one-line summary. `list`, `show`, and packaging read it.

## Commands

```sh
exapp skill                          # print SKILL.md (the router)
exapp skill list                     # feature identifiers and summaries
exapp skill show sync                # print references/sync.md
exapp skill path                     # bundle directory and source commit
exapp skill export --target-dir DIR  # copy the complete bundle to DIR/exapp/
```

The top-level `--help` advertises `exapp skill`.

## Reading

- `exapp skill` and `skill show <feature>` write the packaged file's bytes
  verbatim to stdout, frontmatter included, with no banner, styling, or
  pager.
- `list`, `show`, and `path` need no login, network request, or filesystem
  write, and change no application state.
- An unknown feature fails with a stable `code` such as `unknown-feature`.
  The CLI never prints another feature instead.

## Export

`skill export --target-dir DIR` writes the complete bundle, including
`references/`, `assets/`, and `scripts/`, to `DIR/exapp/`. `--target-dir` is
required. Export does not edit an instruction file, register permissions, or
change application configuration. Before writing, the CLI:

1. Validates the bundle: the Agent Skills validator, the CLI skill contract,
   catalog and router agreement, and every internal reference.
2. Refuses a destination inside a known harness discovery path, such as
   `.claude/skills/`, `~/.claude/skills/`, `.agents/skills/`, or
   `~/.agents/skills/`, with `harness-path-refused`. Sources:
   [Claude Code](https://code.claude.com/docs/en/skills#where-skills-live),
   [Codex](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills).
   The list is non-exhaustive and must track each harness's documented
   locations. It lives in one maintained list, shared through
   `qwts-agent-ci` once that exists, not copied into each CLI.
3. Constrains every path to `DIR/exapp/`. It rejects absolute paths,
   traversal, and escaping symlinks, and does not follow a destination
   symlink.
4. Writes an export record, `DIR/exapp/.exapp-export.json`, holding the CLI
   version, source commit, export time, and a hash of each exported file. The
   record is outside the file set it describes. A hash detects change; it is
   not a signature.
5. Treats an identical export as a no-op (`unchanged`).
6. Refuses an existing destination by default. `--replace` applies only when
   the destination has this CLI's record and every file still matches it;
   otherwise the CLI refuses with `destination-modified`. There is no generic
   force option.
7. Stages a replacement beside the destination and swaps it in only after it
   validates. A failed export keeps the prior copy and exits non-zero.

The result reports `new`, `unchanged`, or `replaced`, with the version,
source commit, and destination; with `--json`, as one JSON document.

## Acceptance criteria

- `skill`, `list`, `show`, and `path` work offline, without credentials or
  writes, and are safe to pipe.
- Printed output equals the packaged bytes of `SKILL.md` or
  `references/<feature>.md`.
- `path` matches the release (CLI skill contract gate check 4).
- Unknown features, invalid bundles, catalog and router disagreement, and a
  missing `--target-dir` fail with documented codes.
- Export tests cover traversal, escaping symlinks, harness discovery
  destinations, unknown existing directories, local edits, identical
  exports, `--replace`, and an interrupted replacement.
- No read or export executes bundled code, requests permissions, or fetches
  from the network.
- Each feature reference has a tested normal workflow and a failure or
  recovery example with observable success criteria.

## Open questions

- **Atomic swap on Windows and across filesystems:** choose the staging and
  swap strategy and the cleanup of leftover staging directories.
- **Shared implementation:** whether `qwts-agent-ci` ships the catalog check
  and export guard as a library or only as gate checks.
- **Authoring skill:** tracked in
  [qwts/qwts-agent-sop#68](https://github.com/qwts/qwts-agent-sop/issues/68);
  `author-cli-skill` is the proposed name.
