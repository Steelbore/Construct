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
git clone git@github.com:Steelbore/skills.git ~/.claude/skills
git clone git@github.com:Steelbore/skills.git ~/.gemini/skills
git clone git@github.com:Steelbore/skills.git ~/.codex/skills
```

The SSH remote is configured for [Gitway](https://github.com/Steelbore/Gitway).
