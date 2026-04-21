# Changelog

All notable changes to this project are documented in this file.

The format is loosely based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.2] - 2026-04-21

- Adds `commands/update-collab-report.md` — an explicit slash-command entry point that invokes the existing workflow. The modern `skills/<name>/SKILL.md` layout auto-maps to a slash command `/<name>` in the Claude Code CLI, but this auto-mapping does not appear to take effect in Claude Cowork; the new `commands/` file uses the legacy filesystem convention so `/update-collab-report` activates the skill in both environments. The command body is a thin dispatcher — it reads `skills/update-collab-report/SKILL.md` and executes it, keeping the skill as the single source of truth.

## [0.1.1] - 2026-04-21

- Target-file discovery is now strict. The skill resolves the project root via `git rev-parse --show-toplevel`, matches only the exact filename `COLLABORATION_REPORT.md` at that root (case-sensitive, no fuzzy matching), and surfaces ambiguous candidates to the user instead of writing to them. When the canonical file is missing, the scaffold now includes a self-documenting `## How this file is maintained` section — the voice rules and addendum structure are copied into the file so the in-file guidance is authoritative for future sessions.
- Draft approval is now a two-field confirmation. The approval prompt prints the absolute target-file path, the action (`CREATE new file` or `APPEND to existing file`), and the full draft — and requires an explicit `approve` or `approve but change target to <absolute path>` token before any write. Blanket approvals (`yes`, `looks good`) are rejected for the first write of a new session. Closes the smoke-test failure where an addendum was written to an unrelated report file because the user approved the draft body without seeing the target path.

## [0.1.0] - 2026-04-21

- Initial release.
- Ships the `update-collab-report` skill — appends a structured session addendum to `COLLABORATION_REPORT.md` at the current project's root.
- Voice contract: no emojis, em-dashes liberal, third-person neutral, concrete specifics, honest about what wasn't shipped.
- Canonical example addendum included at `skills/update-collab-report/examples/example-addendum.md`.
