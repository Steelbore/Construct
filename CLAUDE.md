# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A catalogue of agent skills loaded by Claude Code, Gemini CLI, and Codex from
`~/.claude/skills/`, `~/.gemini/skills/`, `~/.codex/skills/`. There is no build
system, runtime, or test suite — every artifact is markdown plus a small number
of templates/JSON. Edits are content edits; shipping is rezipping.

The authoritative governance document for everything produced in this repo is
[`steelbore-standard/SKILL.md`](steelbore-standard/SKILL.md) (The Steelbore
Standard v1.1). Load it before any non-trivial edit — its §14 checklist is the
audit gate.

## Skill layout (per top-level directory)

```
<skill-name>/
├── SKILL.md           # frontmatter (name, description, license, maintainer, website) + body
├── LICENSE | LICENSE.md  # GPL-3.0-or-later (rust-guidelines is MIT — adapted from Microsoft)
├── references/        # optional; loaded on demand by the agent
└── assets/            # optional; templates, JSON catalogs, etc.
```

Skill IDs (directory and frontmatter `name`) are **functional**, not
metallurgical — Standard §2 reserves codenames for projects/modules/utilities,
not for skill identifiers. The README catalogue and the directory list must
stay in sync.

## Bundling (.zip and .skill)

Each skill ships as two bundles at the repo root: `<name>.zip` and
`<name>.skill`. They contain only `SKILL.md`, `LICENSE`, and `references/`
(plus `assets/` where present) — never tooling, generator scripts, or raw
upstream sources. Auxiliary inputs that don't belong in the shipped skill live
in `Excluded/` (e.g., `Rust-Guidelines.{md,txt}`, `skill.ps1`).

Rebuild pattern (from `.claude/settings.local.json`):

```sh
rm -f <name>.zip <name>.skill
zip -qr  <name>.zip   <name>/SKILL.md <name>/LICENSE <name>/references
zip -qrD <name>.skill <name>/SKILL.md <name>/LICENSE <name>/references
```

The `.skill` bundle uses `-D` to drop directory entries; the `.zip` keeps them.
Verify with `unzip -l <name>.zip` before committing. After editing any file
inside a skill directory, **rebuild both bundles in the same commit** — a stale
bundle ships broken content to every agent that installs from the zip.

## Workflow: every skill-directory change

The bundles are the install surface. A bundle that lags its `SKILL.md` /
`references/` / `assets/` ships broken content to every consumer. The contract
is mechanical — apply it after **any** edit inside a `<skill-name>/` directory:

1. **Rebuild both bundles** for the changed skill:
   ```sh
   rm -f <name>.zip <name>.skill
   zip -qr  <name>.zip   <name>/SKILL.md <name>/LICENSE <name>/references
   zip -qrD <name>.skill <name>/SKILL.md <name>/LICENSE <name>/references
   ```
   Add `<name>/assets` to both lines if the skill has an `assets/` dir.
   Omit `<name>/LICENSE` or `<name>/references` if the skill doesn't have them
   (`steelbore-standard` is SKILL.md-only, for example).
2. **Stage** the skill directory **and** both bundles in the same commit —
   never separately.
3. **Commit with UTC timestamps**:
   ```sh
   TZ=UTC GIT_COMMITTER_DATE="$(TZ=UTC date)" \
     git commit --date "$(TZ=UTC date)" -m "..."
   ```
   Steelbore Standard §12.2 forbids offset notation (`+0300`, `+00:00`); only
   `Z` / `+0000` is permitted. Signing is on globally
   (`commit.gpgsign=true`, `gpg.format=ssh`, `user.signingkey=~/.ssh/id_ed25519.pub`)
   — no extra flag needed.
4. **Push** to `origin/main` with no confirmation prompt — this repo is
   pre-authorised for auto-push on skill-directory changes.

If multiple skills changed in one turn, rebuild **all** of their bundles in the
same commit. Never let `git status` show a skill-dir change without its
matching bundle change.

`git log -1 --show-signature` may report "No signature" locally because
`~/.ssh/allowed_signers` isn't populated; this is a verifier-side gap, not a
signing failure. GitHub validates the SSH signature independently and shows
"Verified" if the public key is registered as a **Signing** key in GitHub
account settings (Authentication-only keys won't validate signatures).

## Editing rules specific to this repo

- **README §2 catalogue is load-bearing.** When adding a skill directory, add a
  matching alphabetical row to the table in `README.md`. When removing one,
  delete the row.
- **Dates are ISO 8601 UTC** anywhere they appear in SKILL.md, references, or
  changelogs (Standard §12). No AM/PM, no local-time strings.
- **Don't import skill content into `MEMORY.md` or CLAUDE.md.** The skills are
  the source of truth and are already loaded on demand.
- `Chat.txt` is a session export and is gitignored — never commit it.
- `Excluded/` is the holding pen for inputs that produce skill content but must
  not ship with it. Don't reference it from inside any `SKILL.md`.

## Installation (what consumers do)

```sh
git clone git@github.com:Steelbore/Construct.git ~/.claude/skills
git clone git@github.com:Steelbore/Construct.git ~/.gemini/skills
git clone git@github.com:Steelbore/Construct.git ~/.codex/skills
```

The SSH remote is configured for [Gitway](https://github.com/Steelbore/Gitway).
