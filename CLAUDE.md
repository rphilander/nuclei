# Nuclei — what a nucleus is, and how to maintain one

A **nucleus** is a captain's log of decisions for a useful system you've built. It documents *why* the system is the way it is, with enough reasoning preserved that the system can be adapted to a different context — or that someone else can build a new one inspired by it.

The concept is system-agnostic. The first nuclei in this repo happen to be for LLM wikis, but a nucleus could just as well document a language runtime, a data pipeline, a content workflow, an internal tool — any *useful system* whose design decisions are worth preserving.

This repo is a collection of nuclei across the systems I've built (and a few snapshots of systems I built elsewhere). It's designed to be **picked at**, not read end-to-end. If something here is useful to you, steal it. Cross-link back if you can.

## Audience

**The reader is an LLM** — typically a future Claude (or other model) that's rebuilding, extending, or stealing from the system. Entries are written in prose for synthesis, not in templates for lookup.

A human should be able to skim a nucleus and follow what's going on. But the format isn't optimized for human consumption — it's optimized for an LLM doing knowledge integration. If a tradeoff arises between "easier for a human to scan" and "denser for a model to synthesize," lean toward the model.

## What a nucleus is for

Most systems lose their reasoning over time. Decisions stay in the code; rationale stays in someone's head; a few years later nobody knows why things are the way they are. A nucleus is the cheapest format I've found for capturing reasoning at the moment a decision is made, so it doesn't evaporate.

Useful systems evolve. Each meaningful evolution is a candidate for an entry. The accumulation of entries, over time, is the system's *thinking* — distinct from its code and content.

## What goes in an entry

- **Design decisions** — choosing one approach over another, with reasoning.
- **Bugs that revealed a non-obvious constraint** — the failure mode is the lesson.
- **Pattern introductions** — new conventions, new content types, new integrations.
- **Failed experiments** — things tried and reverted, with the reason they didn't fit.
- **Course corrections** — changes to how the system works that future builders need to know.

## What doesn't go in

- **Content additions** — new pages, new records, new entries in the system's regular operation. That's just the system doing its job, not a design decision.
- **Routine maintenance** — typo fixes, styling tweaks, dependency bumps.
- **Things already captured** by an existing entry. Amend the original if needed; don't duplicate.

The common case is "nothing nucleus-worthy today." That's healthy. A nucleus that grows by 10 entries a day is mostly noise.

## Voice and shape

- **Narrative prose, not template fields.** Each entry is a paragraph or two of reasoning. No "Problem / Solution / Outcome" headings — those flatten the why.
- **Cross-reference liberally** to thread compounding decisions and adjustments across entries. Decisions don't stand alone; they build on each other.
- **Write dense.** The reader can hold a lot of context. Avoid restating what surrounding entries already establish.

## Structure

Each nucleus is a single HTML file. Layout:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="nucleus-system" content="<short-system-name>">
  <meta name="nucleus-updated" content="YYYY-MM-DD">
  <!-- optional: <meta name="nucleus-frozen" content="true"> for snapshots -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.conditional.min.css">
  <title>Nucleus &mdash; <system name></title>
</head>
<body>
  <main class="pico container" data-theme="light">
    <h1>Nucleus</h1>
    <p class="meta">Author: …<br>Last modified: <time>YYYY-MM-DD</time></p>

    <h2>Preface</h2>
    <p>… short framing of what this system is and why this nucleus exists …</p>

    <hr>

    <section class="log-entry" id="kebab-slug">
      <h2>Entry N &mdash; Short descriptive title</h2>
      <p>The decision, the reasoning, the constraint that surfaced. Multiple <p> tags as needed.</p>
    </section>

    <!-- more entries, in chronological order -->
  </main>
</body>
</html>
```

### Required metadata

- `<meta name="nucleus-system">` — short identifier for the system (lowercase-hyphenated, e.g. `personal-llm-wiki`, `truespeech-runtime`).
- `<meta name="nucleus-updated">` — ISO date of the most recent entry or amendment. Used by tooling to detect new content.

### Optional metadata

- `<meta name="nucleus-frozen" content="true">` — declares this nucleus is a snapshot that won't be updated. Tooling should skip it when scanning for new entries.
- `<meta name="nucleus-snapshot-date">` — for frozen snapshots, when the snapshot was taken.

### Entry conventions

- **IDs** are kebab-case slugs. They're stable — don't rename after publishing.
- **Numbering** is chronological, never reused. If you remove an entry, the number stays gone (or amend the entry to mark it superseded).
- **Cross-references**:
  - Same file: `<a href="#entry-id">Entry N</a>`
  - Same repo, different nucleus: `<a href="other-nucleus.html#entry-id">other Entry N</a>`
  - Across repos: absolute URL, e.g. `<a href="https://other.example.com/nucleus.html#entry-id">…</a>`

## Stealing

You're encouraged to steal. The protocol:

1. Read another nucleus. Find an entry that fits a system you're building.
2. Adapt it to your context — wholesale copy is fine, but the result should make sense for your system.
3. In your nucleus, write the entry in *your* voice with *your* reasoning, and link back to the source with a cross-reference. Attribution by hyperlink, not by quotation.

## Skills in this repo

These are first-class slash commands. They can be invoked directly by the operator or as sub-invocations from system-specific orchestrators (e.g. a sprite's local `/morning` or `/evening`).

- **`/nucleus-add-entry`** — given a drafted entry and a target nucleus file, validates the entry against this spec, assigns the next number, places it in the file, bumps `nucleus-updated`, commits and pushes. Never judges entry-worthiness — that's the caller's job. Never edits prose — places what it's given.
- **`/nucleus-scan-for-steals`** — pulls latest from this repo, compares against a per-sprite cursor of last-seen state, returns a structured list of new entries in *other* nuclei (excluding the local sprite's own). Caller walks them with propose/confirm and may invoke `/nucleus-add-entry` to incorporate stolen entries.

Local skills like `/morning` and `/evening` are system-specific — they own the judgment calls about what to scan, what to surface, and what's nucleus-worthy. They invoke the shared skills above for nucleus-protocol operations.

## License

Content here is CC BY 4.0 — see [LICENSE](LICENSE). Steal liberally, attribute where you can.
