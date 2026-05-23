# nuclei

A collection of **nuclei** — captain's-log decision documents that capture the reasoning behind useful systems I've built across my sprites. Each file is one system; each entry is one decision, bug-revealing-a-constraint, pattern introduction, failed experiment, or course correction.

Designed to be **picked at**, not read end-to-end. If something here is useful to you, steal it. Cross-reference back if you can.

## How it works

Each nucleus documents one *system* — a project worth remembering the reasoning behind. The model has three pieces:

- **The local system** — a project with its own git repo (an LLM wiki, a language runtime, a content pipeline, etc.).
- **The local nucleus** — its decision log: a single HTML file in this repo.
- **Remote nuclei** — every other system's decision log in this repo, available to learn from or steal from.

Two slash-command skills handle the mechanics:

- **`/nucleus-update`** — reads recent git activity in the local system and proposes new nucleus entries for things worth preserving.
- **`/nucleus-scan-for-steals`** — surfaces new entries from *other* nuclei that you might want to adopt into your own.

The skills live in `skills/` and are designed to be symlinked from each sprite's `~/.claude/skills/`. See [CLAUDE.md](CLAUDE.md) for the full spec — entry format, voice conventions, metadata requirements, cross-reference style, and the stealing protocol.

## What's in a nucleus

Each nucleus is a single HTML file with:

- A `<meta name="nucleus-system">` tag naming the system it documents
- A `<meta name="nucleus-updated">` tag with the last-modified date
- A series of `<section class="log-entry">` blocks — kebab-case ids, numbered titles, paragraphs of prose

The audience is an LLM doing synthesis. A human can skim and follow what's going on, but the format isn't optimized for that.

## Files

- [`personal.html`](https://rphilander.github.io/nuclei/personal.html) — Personal LLM wiki ("AI Chief of Staff"). Workstreams, ingest sources, daily briefings, nucleus maintenance. **Active.**
- [`work-snapshot.html`](https://rphilander.github.io/nuclei/work-snapshot.html) — Frozen snapshot of the work-wiki nucleus that `personal.html` inherits from. Curated to include only the entries whose patterns carried over. **Will not be updated** — the active version lives in a private system at work.

More to come as additional systems get nuclei.

## License

Content here is published under [CC BY 4.0](LICENSE). Steal liberally; attribute where you can.
