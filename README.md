# nuclei

A collection of **nuclei** — captain's-log decision documents that capture the reasoning behind useful systems I've built across my sprites. Each file is one system; each entry is one decision, bug-revealing-a-constraint, pattern introduction, or course correction, with the reasoning preserved.

Designed to be picked at, not read end-to-end. If something here is useful to you, steal it. Cross-reference back if you like.

## What's in a nucleus

Each nucleus is a single HTML file with:

- A `<meta name="nucleus-system">` tag naming the system it documents
- A `<meta name="nucleus-updated">` tag with the last-modified date
- A series of `<section class="log-entry">` blocks, each with an `id` (kebab-case slug), a numbered title, and a paragraph or two of prose

Entries are numbered and chronological. Internal cross-references use `<a href="#entry-id">`. Cross-nucleus references use absolute URLs.

## Files

- [`personal.html`](https://rphilander.github.io/nuclei/personal.html) — Personal LLM wiki ("AI Chief of Staff"), adapted from a work system. Workstreams, ingest sources, daily briefings, nucleus maintenance.

More to come as additional systems get nuclei.

## License

Content in this repo is published under [CC BY 4.0](LICENSE). Steal liberally; attribute where you can.
