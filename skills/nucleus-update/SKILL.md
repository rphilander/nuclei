---
name: nucleus-update
description: Scan the local system's recent git activity and propose nucleus-worthy entries; place confirmed ones in the local nucleus and push to the nuclei repo
triggers:
  - /nucleus-update
---

# /nucleus-update — Evolve the local nucleus from local git activity

Reads the **local system's** recent git history (since the cursor), evaluates each meaningful change against the nucleus-worthiness bar from [CLAUDE.md](../../CLAUDE.md), walks the operator through propose/confirm, drafts entries in the spec's voice, places confirmed ones in the **local nucleus**, and commits + pushes the nuclei repo.

This is the canonical "update my nucleus" operation. A local `/evening` typically invokes it as the final step after system-specific intel work. The operator can also invoke it directly any time the local system has accumulated changes worth recording.

## Context auto-discovery

The skill reads `.nuclei.json` at the local system's repo root to determine identity:

```json
{
  "system": "personal-llm-wiki",
  "nucleus_file": "personal.html",
  "local_repo": "/home/sprite",
  "nuclei_repo": "/home/sprite/nuclei"
}
```

If the file is missing or malformed, refuse with a clear message — the operator should create it. Don't try to guess.

Per-sprite cursor lives at `~/.config/nuclei-cursor.json`:

```json
{
  "local": { "last_seen_commit": "<sha>" },
  "remote": { "...": "..." }
}
```

If the cursor file or its `local` section is missing, treat the cursor as the local repo's HEAD ~14 days ago (or HEAD~50, whichever is shorter) — that's a reasonable first-run window. Surface a note that this is a first run so the operator knows the window.

## Steps

### 1. Establish context

- Load `.nuclei.json` from `<local_repo>/.nuclei.json`. Validate the four required fields. Confirm both repos exist as git working trees.
- Load the cursor file. Determine `since_commit` (the cursor's last-seen commit, or the first-run fallback).
- Confirm the local repo's working tree is clean enough to reason about — uncommitted changes are fine (they'll just appear as "in progress"), but warn the operator if the tree has staged-but-uncommitted state that affects the scan.

### 2. Survey local activity

- Run `git log <since_commit>..HEAD --no-merges` in the local repo.
- For each commit, capture: subject, body, author, date, and changed file paths.
- Read commit messages carefully — well-written ones (per the `/checkpoint` discipline) already carry the reasoning that would go in a nucleus entry.

### 3. Evaluate candidates

For each commit, judge against the nucleus-worthiness bar from [CLAUDE.md](../../CLAUDE.md):

- **Design decisions** — choosing one approach over another with reasoning ✓
- **Bugs revealing a non-obvious constraint** ✓
- **Pattern introductions** — new conventions, content types, integrations ✓
- **Failed experiments** with their lesson ✓
- **Course corrections** ✓

Skip:
- Content additions (new pages, new records — the system doing its job)
- Routine maintenance (typo fixes, dep bumps, styling tweaks)
- Things already captured by existing nucleus entries (read the local nucleus to check)

The expected common case is "few or none." A nucleus that grows by many entries per scan is mostly noise.

### 4. Walk candidates with the operator

For each candidate, conversationally:

- Summarize the commit and why it might be nucleus-worthy
- Propose a draft entry (full HTML `<section>` block, kebab-case id, paragraphs of prose, cross-references where applicable)
- Ask: **add**, **edit**, or **skip**
- For *edit*: take operator feedback and revise, then re-confirm

Do not propose entries you yourself wouldn't endorse — if a commit looks borderline, raise the doubt explicitly.

### 5. Apply confirmed entries

For each confirmed draft:

- Validate against the spec: single `<section class="log-entry" id="...">…</section>`, kebab-case id, no collision with existing ids, has `<h2>` and at least one `<p>`.
- Assign the next sequential entry number by parsing existing `<h2>Entry N &mdash; …</h2>` headers and taking `max(N) + 1`.
- Rewrite the draft's `<h2>` with the assigned number, preserving the title.
- Insert the section just before `</main>` in the local nucleus file. Leave one blank line above and below.

### 6. Update metadata

- Set `<meta name="nucleus-updated" content="YYYY-MM-DD">` to today's date.
- Update the human-readable `<time>` in the meta paragraph if present.

### 7. Commit and push

- Verify the only modified file in the nuclei repo working tree is the local nucleus. If other files are dirty, refuse — the caller should clean up first.
- Stage the file.
- Commit with the message:
  ```
  Nucleus: add Entries N–M — <one-line theme>
  ```
  Body: short list naming each entry by title so future scanners can find them via git log.
- Push to origin.

### 8. Advance cursor

- Update `cursor.local.last_seen_commit` to the local repo's current HEAD SHA (not just the latest commit you reviewed — the operator may have skipped some, but the cursor advances regardless to avoid re-scanning the same window forever).
- Write the cursor file back.

### 9. Report

Return:
- Number of candidates considered
- Number added (with ids and entry numbers)
- Number skipped
- Commit hash on the nuclei repo, push status
- New cursor position

## Disciplines

- **Never auto-write.** Always propose and confirm. The "propose / confirm" cadence is what makes the discipline sustainable; skipping it makes the nucleus drift from the operator's actual judgment.
- **Bias toward fewer entries.** The common case is "nothing worth adding." That's healthy.
- **One file changed per commit.** Only the local nucleus file should be touched in the nuclei repo.
- **Atomic or visible.** If push fails, leave the local commit in place and report clearly. Don't silently retry.
- **Cursor advances on completion, not per-entry.** If the operator aborts mid-walk, leave the cursor where it was.
- **Spec is the source of truth.** Defer to [CLAUDE.md](../../CLAUDE.md) for ambiguous structural questions.

## Direct invocation

Operators may invoke `/nucleus-update` directly any time, outside of `/evening` or other orchestrators. The behavior is identical — auto-discover from `.nuclei.json`, walk the window, propose, apply, commit, advance.
