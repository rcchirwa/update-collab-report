---
name: update-collab-report
description: "Append a structured session addendum to COLLABORATION_REPORT.md at the current project's root. Captures what was built, verified, shipped, and learned in the current session — in a consistent voice designed to be mined later for resume bullet points, cover letters, LinkedIn posts, and portfolio content. Triggers on 'update collab report' and variants."
argument-hint: "[optional topic phrase]"
---

# update-collab-report

Append a structured session addendum to the project's `COLLABORATION_REPORT.md`. The addendum captures what was shipped, verified, and learned in the current session — in a voice designed to be mined later for resume bullets, cover-letter paragraphs, and portfolio content.

## Triggers

Fire this skill when the user says any of:

- `update collab report`
- `update collaboration report`
- `log this session`
- `add session addendum`
- `journal this session`
- `write up this session for the collab report`

## Procedure

Execute these steps in order. Do not skip steps. Do not parallelize.

### 1. Locate the target file

Execute these sub-steps in order. Do not short-circuit and do not fuzzy-match.

1. Resolve the project root via `git rev-parse --show-toplevel`. If the current directory is not inside a git repo, ask the user for the absolute project root path. Do not proceed until a root is established.
2. Look for a file whose filename is exactly `COLLABORATION_REPORT.md` at the project root. Case-sensitive. No fuzzy matching. Never consider files named `collab_report.md`, `REPORT.md`, `SESSION_NOTES.md`, `collaboration_report.md`, or any other variant as a match. If an ambiguous candidate exists at the root, list it to the user and ask whether to create a new canonical `COLLABORATION_REPORT.md` or treat the candidate as the target.
3. If `COLLABORATION_REPORT.md` is not present at the project root, create it using the scaffold below. Confirm with the user before creating. The scaffold must include an in-file `## How this file is maintained` section whose content is the voice rules and the addendum structure — copied verbatim so the file is self-documenting for any future contributor or future session.
4. After creation (or if the file already exists), read the `## How this file is maintained` section of the target file and treat it as authoritative. If the in-file guidance drifts from the SKILL.md guidance over time, the in-file guidance wins. Log a short note to the user when this happens.
5. Only then proceed to the session-introspection step.

Scaffold for a newly created `COLLABORATION_REPORT.md`:

```
# <Project Name> — Collaboration Report

**Project:** <Project>
**Repo:** <git remote origin URL>
**Started:** <today, YYYY-MM-DD>

## How this file is maintained

This file is append-only. New entries are added below the last `---` separator, never inserted mid-file, never reordered, never rewritten.

Each entry is a "Session Addendum" with a header of the form:

    ## Session Addendum — <Topic Phrase> (YYYY-MM-DD, close+N)

Where `N` is the session counter — one more than the highest `close+N` already in the file.

Voice rules:
- No emojis.
- Em-dashes liberally for mid-sentence clauses.
- Third-person neutral ("the session shipped", not "I shipped").
- Concrete specifics over vague summaries — file paths with line numbers, PR numbers with links, 7-char commit hashes, test counts in before → after form.
- Honest about what wasn't shipped — every addendum names at least one deferred or failed item.
- No marketing language (banned: robust, powerful, seamless, enterprise-grade, cutting-edge, revolutionary, game-changing).

Addendum section menu (pick 3–6, skip sections with no content):

- What shipped · The debugging layers · Design choices that deviated from the spec · Verification · Commit & PR · Secret-handling discipline held · Lessons banked · Not shipped (intentional) · What this added to the toolkit

Every addendum ends with a single `---` horizontal rule.

---
```

### 2. Introspect the session

Before drafting, scan the current conversation for evidence. Collect:

- Files created or edited, with absolute paths and approximate line counts.
- Shell commands of note — `git commit`, `gh pr create`, `pytest`, `curl` probes, diagnostics.
- PRs opened — extract PR number, URL, title, branch name, file stats (`files changed, +X / -Y`), and merge status.
- Commit hashes (7-char short SHAs) and their one-line descriptions.
- Tests added — including the total test count before and after.
- Errors encountered, how they were diagnosed, and in which layer they actually lived.
- Tool counts for MCP-server-style projects that track "N tools".
- What was deliberately not done, or what remains queued for a follow-up.

### 3. Determine the session counter

Scan the existing file for addendum headers matching the regex `\(YYYY-MM-DD, close\+(\d+)\)`. The new entry's counter is `max(existing) + 1`. If there are no matching headers, use `close+1`.

### 4. Ask the user for a short topic phrase

Ask for a 3–8-word phrase that frames the narrative hook — for example, `Shopify Connector Debug + Credential-Loading Hardening` or `CI Gate + Pytest Migration`. Do not auto-generate this unless the session has one unmistakable topic.

### 5. Draft the addendum

Produce the draft using the structure in [Addendum structure](#addendum-structure) below and the voice rules in [Voice rules](#voice-rules).

### 6. Show the draft for approval

The approval prompt is a two-field confirmation, not a single-field one. Print these three fields, in this order, before any write:

1. **Target file (absolute path).** The exact absolute path the skill is about to append to. Render on its own line, in backticks, with a leading label `Target file:`.
2. **Action.** Either `CREATE new file` or `APPEND to existing file`. Render on its own line.
3. **Full draft.** The complete addendum body that will be written.

After printing those three fields, require the user to type one of the following tokens to proceed:

- `approve` — the skill writes to the target file.
- `approve but change target to <absolute path>` — the skill redirects the write to the new path. The new path must also end in `COLLABORATION_REPORT.md` at some project root; reject any other filename.
- Any edit or regenerate request — the skill regenerates the draft and re-prompts with all three fields.
- `cancel` — the skill exits without writing.

Silence or a non-matching response counts as neither approval nor cancellation; re-prompt rather than proceed. Do not accept blanket approvals (`yes`, `looks good`, `sounds good`) for the first write of a new session — the approval token must explicitly match `approve` or the redirect form. This is narrow by design: ambiguous approval must not be treated as confirmation of a file choice the user has not seen.

### 7. Append to the file

- If the file ends with a trailer like `*This report was generated...*`, insert the new addendum immediately above it.
- Otherwise, append to the end of the file.
- Preserve all existing content byte-for-byte — this skill only adds, never edits.

## Addendum structure

```
## Session Addendum — <Topic Phrase> (YYYY-MM-DD, close+N)

<Opening paragraph: 2–5 sentences. Anchor the session — what triggered it, what the original intent was, what actually happened. Be honest if the session pivoted or the original goal was not completed.>

### <Section 1 heading>

<Pick 3–6 of the following sub-sections based on what the session actually involved. Do not include sections that have no content.>

- **What shipped** — bulleted list, one bullet per concrete deliverable. Include PR number + link, branch name, file path, and line deltas.
- **The debugging layers** — numbered list for debug-heavy sessions. Each layer starts with a short bolded principle, followed by concrete evidence.
- **Design choices that deviated from the spec** — numbered list when the session made deliberate trade-offs against a handoff or spec.
- **Verification** — bulleted list of how correctness was established: unit test counts, live checks, negative checks, CI runs. Include concrete numbers.
- **Commit & PR** — one bullet per PR. Format: `<type>: <title>` — [PR #N — `<branch-name>`](url). `<file count>` files, `+X / -Y` lines. `<merge status>`.
- **Secret-handling discipline held** — when credentials were involved. Enumerate each secret-adjacent action and how it was redacted.
- **Lessons banked** — numbered list, 2–4 lessons. Each lesson is a short bolded principle followed by one sentence of concrete evidence from this session.
- **Not shipped (intentional)** — bullet list of what was deliberately cut from scope.
- **What this added to the toolkit** — closing paragraph. 3–6 sentences with numeric deltas (tool count before/after, test count before/after). End with an honest note about what is queued next, if applicable.
```

Always end the addendum with a single `---` horizontal rule as a separator before any future addendum.

## Voice rules

These are non-negotiable.

1. **No emojis.** Anywhere. Ever.
2. **Em-dashes (—) used liberally** for mid-sentence clauses and appositives. This is part of the house style.
3. **Third-person neutral.** Write `a follow-up session addressed...`, `the session shipped...`, `running X revealed Y`. Not `I did`, not `we did`. This matches the existing entries and keeps the file resume-mineable.
4. **Concrete specifics over vague summaries.** File paths with line numbers (`shopify_client.py:41`), PR numbers with markdown links, 7-char commit hashes, test counts in `39 → 70` form, branch names in backticks.
5. **Honest about failure and what wasn't done.** Every addendum names at least one thing that was deliberately deferred, didn't work, or remains queued. A manufactured clean success story is worse than an honest partial one.
6. **Lessons are principles plus evidence.** Not `learned that debugging is important`. Instead: `**Cryptic errors hide real errors — fix the mask before diagnosing the symptom.** The 'str' object has no attribute 'get' was a Python error, not a Shopify error.`
7. **Numeric discipline.** State counts as `before → after`, e.g. `Tool count unchanged at **22**; offline test count up from 39 to **70** (+17).` Bold the final-state number.
8. **PR links as markdown.** `` [PR #10 — `branch-name`](url) ``.
9. **No marketing language.** Banned: `robust`, `powerful`, `seamless`, `enterprise-grade`, `cutting-edge`, `revolutionary`, `game-changing`.
10. **Bold for emphasis** on key terms inside a lesson or bullet — sparingly, never decoratively.

## Credential handling

This skill must never:

- `cat` `.env`, `claude_desktop_config.json`, or any file matching `*.credentials*`, `*.key`, `id_rsa*`, `service-account*.json`.
- Include a full access token, API key, database connection string, or OAuth secret in the addendum — even if the session's tool history contains one.
- Reproduce HTTP request bodies with real `Authorization:` headers.

When describing credential-handling work, use masked forms only:

- Shopify access token: `shpat_…abcd` (prefix plus last 4 characters).
- GCP API key: `AIza…XYZW` (prefix plus last 4).
- Length-only references: `<len=38>`.

If the user asks to include a full secret in the addendum, refuse and explain — the file is meant to be public-portfolio-ready.

## Edge cases

- **Session had nothing substantive.** If the session produced no concrete deliverables worth journaling, tell the user `this session produced no concrete deliverables worth journaling — skipping` and do not write.
- **Main thread unclear.** If the session spanned many sub-topics, ask the user: `Which thread should the addendum focus on? A) <option>, B) <option>, C) other.`
- **Project root ambiguous.** Ask which directory to treat as the project root rather than guessing.
- **Trailer line preservation.** Some collaboration reports end with a trailer like `*This report was generated by Claude Code.*`. Always insert new addendums above the trailer, never below it.

## Reference example

The canonical format reference lives at [`examples/example-addendum.md`](examples/example-addendum.md). Read it before drafting the first addendum in a new project so the voice lands correctly.
