---
name: update-collab-report
description: "Append a structured session addendum to COLLABORATION_REPORT.md at the current project's root. Executes the full update-collab-report workflow defined in skills/update-collab-report/SKILL.md."
argument-hint: "[optional topic phrase]"
allowed-tools: [Read, Bash, Edit, Write, Glob, Grep]
---

# /update-collab-report

This command runs the full `update-collab-report` workflow — the same workflow the natural-language trigger `update collab report` invokes.

## Instructions

1. Read the file `skills/update-collab-report/SKILL.md` from this plugin directory. That file is the authoritative spec for this command's behavior.
2. Execute every numbered step in its `## Steps` section, in order. Do not short-circuit. Do not improvise beyond what the skill document prescribes.
3. The user's arguments to this command are: `$ARGUMENTS`. Treat them as the topic phrase for the addendum header (see step 5 of `SKILL.md`). If `$ARGUMENTS` is empty, derive a topic phrase from the session's actual work during step 4 (session introspection).

## Why this command exists

The modern `skills/<name>/SKILL.md` layout auto-maps to a slash command `/<name>` in the Claude Code CLI. In Claude Cowork this auto-mapping does not appear to take effect, so this file adds an explicit `commands/` entry using the legacy convention — the filename itself registers `/update-collab-report` in environments that don't auto-map skills. The skill file remains the single source of truth; this command is a thin dispatcher.
