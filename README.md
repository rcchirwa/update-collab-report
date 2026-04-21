# update-collab-report

A Claude Cowork skill that appends a structured session addendum to `COLLABORATION_REPORT.md` at the current project's root — in a consistent voice designed to be mined later for resume bullets, cover-letter paragraphs, LinkedIn posts, and portfolio content.

## Why this exists

Coding sessions with Claude produce a lot of evidence — pull requests, diagnostic arcs, credential-handling decisions, test-count deltas — that would make strong portfolio material if it were captured at the close of the session, before the details blur. Reconstructing it from `git log` a week later loses the reasoning, the failure modes, and the lessons. This skill exists to capture it on the spot, in a disciplined format, so the project's `COLLABORATION_REPORT.md` becomes a mineable log of real engineering work.

The voice is third-person neutral and deliberately un-marketed — plain engineering prose with concrete specifics (file paths, PR numbers, commit hashes, test counts). That's a format a future self (or a resume generator) can lift cleanly into a bullet without having to rewrite.

## Install

### Claude Cowork

1. Grab the packaged `update-collab-report.skill` file — either build it locally (see below) or download it from this repo's GitHub Releases.
2. In Claude Desktop, open the **Cowork** tab → **Customize** (left sidebar) → click the **+** button and look for the current upload affordance. It may be labelled **Upload custom plugin**, **Upload skill**, **Add custom skill**, or similar — Cowork's UI has shifted and the button name is not stable across versions.
3. Select `update-collab-report.skill`. Confirm the skill appears in the Customize list.
4. Start a fresh Cowork session for the skill to register.

> Cowork's upload-format support for custom skills is not authoritatively documented at the time of this writing. If the upload dialog rejects `.skill`, open a support ticket with the exact rejection message — the skill content itself is correct, the failure is likely in the upload affordance.

### Claude Code CLI

The CLI install path for v0.2.0 (post-plugin-format) is not yet verified. Two options:

- **Use the prior plugin-format release.** Tag `v0.1.2` in this repo is still a full Claude Code plugin with `.claude-plugin/` manifests — install via `claude plugin marketplace add /path/to/update-collab-report` + `claude plugin install update-collab-report@<marketplace-name>`. Check out `v0.1.2` before running those commands.
- **Wait for verified skill-install guidance.** If Claude Code CLI picks up skills from a user-level directory (e.g. `~/.claude/skills/`), that path is not documented as of 2026-04 — do not rely on it without testing.

Once either install path lands, the skill is invoked via natural-language phrase (see **Usage**) or by typing `/update-collab-report` in CLI versions that auto-map modern skill layouts to slash commands.

### Build the `.skill` file from source

Regenerating the upload artifact requires the `skill-creator` skill's packaging script, which must be invoked as a Python module from its own install directory. Also note that the script does NOT auto-exclude `.git/` or `.gitignore` — running it directly on a git-tracked working tree leaks ~60KB of git objects into the archive, bloating it from ~12KB to ~75KB.

```
# 1. rsync the skill content to a clean staging dir, excluding git/system noise.
rsync -a \
  --exclude='.git' \
  --exclude='.DS_Store' \
  --exclude='.gitignore' \
  /path/to/update-collab-report/ \
  /tmp/update-collab-report-build/update-collab-report/

# 2. Run the packager as a module from the skill-creator directory.
#    Substitute <path-to-skill-creator> for the actual install path
#    (e.g. ~/.claude/plugins/.../skills/skill-creator).
cd <path-to-skill-creator>
python3 -m scripts.package_skill \
  /tmp/update-collab-report-build/update-collab-report \
  /tmp/

# 3. The artifact lands at /tmp/update-collab-report.skill (~12KB, five files).
#    That's the file Cowork's upload dialog expects.
```

Direct-script invocation (`python3 scripts/package_skill.py`) will fail with `ModuleNotFoundError: No module named 'scripts'` — the module form is required.

## Usage

Inside a project that has a `COLLABORATION_REPORT.md` at its root — or is willing to have one scaffolded — invoke the skill with any of:

- `/update-collab-report`
- "update collab report"
- "update collaboration report"
- "log this session"
- "add session addendum"
- "journal this session"
- "write up this session for the collab report"

The skill will introspect the session — files edited, commits made, PRs opened, tests added, errors diagnosed, decisions taken — ask for a short topic phrase, draft the addendum, show it for three-field approval (target file path, create-or-append action, full draft), and only write on explicit `approve`.

See [`examples/example-addendum.md`](examples/example-addendum.md) for a canonical reference output.

## Format contract

Every addendum the skill writes conforms to these rules:

- **No emojis.** Anywhere.
- **Em-dashes (—) used liberally** for mid-sentence clauses and appositives.
- **Third-person neutral.** "A follow-up session addressed...", not "I did X". Keeps the file resume-mineable.
- **Concrete specifics over vague summaries** — file paths with line numbers, 7-char commit SHAs, PR numbers with links, test counts in `before → after` form.
- **Honest about what wasn't shipped.** Every addendum names at least one thing that was deliberately deferred or remains queued. Manufactured success stories are worse than honest partial ones.
- **No marketing language.** Banned: `robust`, `powerful`, `seamless`, `enterprise-grade`, `cutting-edge`, `revolutionary`, `game-changing`.
- **Bold sparingly** — only on the key term inside a lesson or bullet, never decoratively.

## What it explicitly does NOT do

- Does not `cat` `.env`, `claude_desktop_config.json`, or any file matching `*.credentials*`, `*.key`, `id_rsa*`.
- Does not embed full access tokens, API keys, or passwords in the addendum — even if the session's tool history contains them. Masked forms only (`shpat_…abcd`, `<len=38>`).
- Does not write to `COLLABORATION_REPORT.md` without showing the draft for approval first.
- Does not invent content when the session had nothing substantive to report — it declines and tells the user so.
- Does not modify existing addendums or reorder the file — it only appends.

## Known edge cases

- If `COLLABORATION_REPORT.md` doesn't exist, the skill offers to create a minimal scaffold and confirms before proceeding. The scaffold is self-documenting — it embeds the voice rules and addendum structure so the file remains authoritative even if the skill drifts.
- If the current directory isn't inside a git repo, the skill asks the user which directory is the project root.
- If the session spanned many sub-topics, the skill asks which thread the addendum should focus on rather than producing a muddled summary.
- The session-counter heuristic (`close+N`) reads existing addendum headers via regex `\(YYYY-MM-DD, close\+(\d+)\)`. Hand-edited headers that drift from this format will cause the counter to restart at 1.

## History

This project started life as a Claude Code plugin (`v0.1.0` through `v0.1.2`, tagged in this repo's history) before pivoting to a standalone skill at `v0.2.0` — the plugin format's Cowork distribution path was undocumented and the upload dialog rejected the plugin manifests. The skill format via `.skill` packaging is the current supported path.

## Contributing

Pull requests welcome. The house style is the format contract above — new example outputs or edits to the SKILL body should hold the same voice rules. Open an issue first for non-trivial changes.

## Credits

Built by Robert Chirwa using Claude Code.
