# update-collab-report

A Claude Cowork skill that appends a structured session addendum to `COLLABORATION_REPORT.md` at the current project's root — in a consistent voice designed to be mined later for resume bullets, cover-letter paragraphs, LinkedIn posts, and portfolio content.

## Why this exists

Coding sessions with Claude produce a lot of evidence — pull requests, diagnostic arcs, credential-handling decisions, test-count deltas — that would make strong portfolio material if it were captured at the close of the session, before the details blur. Reconstructing it from `git log` a week later loses the reasoning, the failure modes, and the lessons. This skill exists to capture it on the spot, in a disciplined format, so the project's `COLLABORATION_REPORT.md` becomes a mineable log of real engineering work.

The voice is third-person neutral and deliberately un-marketed — plain engineering prose with concrete specifics (file paths, PR numbers, commit hashes, test counts). That's a format a future self (or a resume generator) can lift cleanly into a bullet without having to rewrite.

## Install (Claude Desktop)

1. In Claude Desktop: click **+** → **Plugins** → **Add plugin** → **From GitHub**.
2. Enter `https://github.com/rcchirwa/update-collab-report` and confirm.
3. Click **Manage plugins** and ensure `update-collab-report` is **enabled**.
4. Restart the chat (or start a new one) for the skill to appear.

If the repo isn't published yet, use **Add plugin** → **Local path** and select the plugin directory on disk.

## Install (Claude Code CLI)

One-shot session load — useful while iterating on the plugin itself:

```
claude --plugin-dir /path/to/update-collab-report
```

Persistent install via a local marketplace:

```
claude plugin marketplace add /path/to/update-collab-report
claude plugin install update-collab-report@local-dev
```

## Usage

Say any of the following in a Claude chat inside a project that has a `COLLABORATION_REPORT.md` at its root:

- "update collab report"
- "update collaboration report"
- "log this session"
- "add session addendum"
- "journal this session"
- "write up this session for the collab report"

The skill will introspect the session — files edited, commits made, PRs opened, tests added, errors diagnosed, decisions taken — ask for a short topic phrase, draft the addendum, show it for review, and only append on explicit approval.

See [`skills/update-collab-report/examples/example-addendum.md`](skills/update-collab-report/examples/example-addendum.md) for a canonical reference output.

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

- If `COLLABORATION_REPORT.md` doesn't exist, the skill offers to create a minimal scaffold and confirms before proceeding.
- If the current directory isn't inside a git repo, the skill asks the user which directory is the project root.
- If the session spanned many sub-topics, the skill asks which thread the addendum should focus on rather than producing a muddled summary.
- The session-counter heuristic (`close+N`) reads existing addendum headers via regex `\(YYYY-MM-DD, close\+(\d+)\)`. Hand-edited headers that drift from this format will cause the counter to restart at 1.

## Contributing

Pull requests welcome. The house style is the format contract above — new example outputs or edits to the SKILL body should hold the same voice rules. Open an issue first for non-trivial changes.

## Credits

Built by Robert Chirwa using Claude Code.
