---
name: nucleus-scan-for-steals
description: Pull the nuclei repo and surface new entries from REMOTE nuclei that the operator might want to steal into the local nucleus
triggers:
  - /nucleus-scan-for-steals
---

# /nucleus-scan-for-steals — Find new entries in remote nuclei

Given the per-sprite cursor of last-seen entry ids, this skill:

- Pulls latest from the nuclei repo
- For each **remote** nucleus (every nucleus that's not the local system's own), identifies entries that have appeared since the cursor's last update
- Walks the operator through them as steal candidates with **steal / skip / defer** dispositions
- For confirmed steals, drafts an adapted entry with an attribution cross-reference and hands it to [`/nucleus-update`](../nucleus-update/SKILL.md) for placement
- Advances the cursor for entries that were *resolved* (stolen or skipped)

**Frozen nuclei** (those with `<meta name="nucleus-frozen" content="true">`) are skipped — they don't generate new content by design.

## Context auto-discovery

Reads `.nuclei.json` at the local system's repo root to determine identity and locate the nuclei repo:

```json
{
  "system": "personal-llm-wiki",
  "nucleus_file": "personal.html",
  "local_repo": "/home/sprite",
  "nuclei_repo": "/home/sprite/nuclei"
}
```

The `nucleus_file` value identifies which file in the nuclei repo is the *local* nucleus (and therefore excluded from the scan).

Per-sprite cursor lives at `~/.config/nuclei-cursor.json`:

```json
{
  "local":  { "last_seen_commit": "<sha>" },
  "remote": {
    "personal.html":      { "last_seen_entries": ["...", "..."], "last_pulled_at": "ISO" },
    "truespeech.html":    { "last_seen_entries": ["..."],         "last_pulled_at": "ISO" },
    "work-snapshot.html": { "last_seen_entries": ["...", "..."], "last_pulled_at": "ISO" }
  }
}
```

The `remote` section maps each remote nucleus filename to the ids of entries the operator has resolved (whether by stealing or skipping). Entries left as *deferred* don't get added to this list — they'll surface again next scan.

If the cursor file or its `remote` section is missing, treat all entries as new — that's expected first-run behavior.

## Steps

### 1. Pull and prepare

- Load `.nuclei.json`; abort if missing.
- Run `git pull` in `nuclei_repo`. If the pull fails (network, conflict), abort cleanly and tell the operator.
- Enumerate `*.html` files at the nuclei repo root.
- For each file, read `<meta name="nucleus-system">` and `<meta name="nucleus-frozen">`.
- Exclude:
  - The file whose path matches the local `nucleus_file`
  - Any file with `nucleus-frozen` set to `true`

### 2. Find new entries

- Load the cursor file (or treat as empty on first run).
- For each remaining remote nucleus:
  - Parse all `<section class="log-entry" id="...">` blocks
  - Compare entry ids against `cursor.remote[filename].last_seen_entries`
  - Collect ids not in the cursor as **candidates**
- If no candidates across any remote, report "nothing new" and exit.

### 3. Walk candidates with the operator

For each candidate, conversationally:

- Show source nucleus, entry id, entry title, and the entry's full prose
- Ask: **steal**, **skip**, or **defer**

**Steal** means: I want to incorporate the idea into my own nucleus. The skill drafts an adapted entry — same idea, but written in the local system's voice and adapted to the local system's context. The draft includes an attribution cross-reference:

```html
<p class="note">Adapted from
  <a href="https://rphilander.github.io/nuclei/&lt;source&gt;.html#&lt;entry-id&gt;">&lt;source&gt; Entry N</a>.
</p>
```

Confirm the adapted draft with the operator, then hand it to [`/nucleus-update`](../nucleus-update/SKILL.md) as a confirmed entry for placement.

**Skip** means: I've seen this, it's not for me. The entry id gets added to the cursor so it doesn't resurface.

**Defer** means: I'm not sure yet. The entry stays out of the cursor and will surface again on the next scan.

### 4. Update cursor

After the walk completes (or the operator stops), append the ids of *stolen* and *skipped* entries to `cursor.remote[filename].last_seen_entries` for each affected file. Update `last_pulled_at` to now. Leave *deferred* ids out.

Write the cursor file back.

### 5. Report

Return:
- Number of remote nuclei scanned
- Number of candidates surfaced
- Counts by disposition (stolen, skipped, deferred)
- For stolen entries: the new ids and entry numbers in the local nucleus

## Disciplines

- **Read-only on remote nuclei.** This skill never edits remote files. Stealing means *copying and adapting*, not modifying the source.
- **Attribution is required for steals.** Every stolen entry carries a cross-reference back to the source. The hyperlink is the attribution.
- **Cursor advances only on resolution.** Deferred entries leave the cursor untouched so they resurface.
- **Skip frozen.** Frozen nuclei don't generate new content; respect the convention.
- **Self-exclusion is required.** If the local `nucleus_file` can't be determined, refuse rather than risking scanning the local nucleus as if it were remote.

## Direct invocation

The operator may invoke `/nucleus-scan-for-steals` directly outside a `/morning` flow — useful for spot-checking what's new in other systems' nuclei. Direct invocation behaves identically: walk candidates, apply dispositions, advance cursor for resolved ones.
