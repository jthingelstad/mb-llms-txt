# Roadmap

What this plug-in ships today, what it would like to ship, and what's currently in the way.

## Shipped (v1.0.2)

- `/llms.txt` at the site root — H1, blockquote, optional Pages section, Recent Posts, optional year-grouped full archive.
- `/llms-full.txt` at the site root — full Markdown bodies of the N most recent posts, configurable cap.
- Five user settings exposed via the Micro.blog plug-in UI (intro, recent count, full count, include pages, include archive).

## Wanted: per-post Markdown alternates

The [llmstxt.org](https://llmstxt.org/) spec recommends that every page be available as a clean Markdown twin (e.g. `/2025/06/19/post.html` ↔ `/2025/06/19/post.md`), with a `<link rel="alternate" type="text/markdown">` tag in the HTML head for discovery. This is the spec's canonical "atomic content unit" for LLM consumption — it's what lets an agent fetch only what it needs instead of pulling all of `/llms-full.txt`.

**What's blocking it**: Micro.blog's plug-in build pipeline doesn't appear to merge `outputs.page` additions from a plug-in's `config.json` into the site config. Without a `markdown` output format actually registered for pages, no `.md` files are produced — and any `_default/single.<format>.md` template the plug-in ships risks being applied to the *HTML* output, breaking every post page on the host blog. (We hit exactly this on a live install in v1.0.0/v1.0.1; v1.0.2 removed the feature to keep installations safe.)

**What needs to happen**: see [`NOTES_FOR_MICROBLOG.md`](./NOTES_FOR_MICROBLOG.md) for the detailed write-up sent to Manton. Short version — Micro.blog's pipeline needs to merge plug-in `outputs.page` (and ideally `outputs.section`/`outputs.taxonomy`/`outputs.term`) the way it apparently already merges `outputs.home`. Once that lands, we re-add `_default/single.markdown.md`, the `markdown` output format in `config.json`, and the head-injection partial — all of which were proven to work in local Hugo testing before being removed.

## Wanted: per-year files (`/llms-2025.txt`, `/llms-2024.txt`, …)

A per-year split of `/llms-full.txt` would let large-archive blogs (e.g. 20-year micro.blogs with 18,000+ posts) publish full content without producing one giant unusable file. An LLM client could then ask for only the year(s) it cares about.

**What's blocking it**: Hugo can't natively emit one file per year at a predictable root URL from a single template. The standard mechanisms are:

- **Taxonomies** — would emit one file per term, but Hugo populates taxonomy values from per-post front matter. We can't retroactively add `years: ["2025"]` to thousands of existing posts on every blog this plug-in installs onto.
- **Sections** — would require one content file per year, manually authored. Doesn't scale to "any user's blog."
- **`resources.ExecuteAsTemplate`** in a loop — can write multiple files, but only into `/assets`, with awkward URL routing back to the root.

This is confirmed by [`microdotblog/plugin-archive-months`](https://github.com/microdotblog/plugin-archive-months), which faces the same constraint and works around it by emitting a single HTML page and filtering year-by-year client-side with JavaScript. That trick doesn't translate to plain-text outputs.

**What might unblock it**:

1. **Micro.blog auto-tags posts with their publication year** as a hidden taxonomy (e.g. populating `years: ["2025"]` on every post at build time). The plug-in could then ship a `term.llmsfull.txt` template and Hugo would emit one file per year for free. This is the cleanest path.
2. **A Micro.blog-provided helper that writes arbitrary files** to the site root from plug-in templates. More flexible but a bigger surface to design.
3. **A pure-Hugo workaround** using `resources.ExecuteAsTemplate` plus a permalinks rewrite — possible but fragile, and depends on which Hugo features Micro.blog allows in plug-ins. Worth a spike if Micro.blog stays unchanged.

## Smaller things worth considering

- **`/llms.txt` description rendering**: when a user's site description contains raw HTML (e.g. `<a href="…">@handle</a>`), it shows up literally in the blockquote. Should pipe through a Markdown-safe transform, or convert HTML links to Markdown links.
- **Custom intro override per post type** — currently the intro blockquote applies to both `/llms.txt` and `/llms-full.txt`. Some users may want different framing in each.
- **Validation hook** — a CI check that lints the generated `/llms.txt` against [llmstxt.org](https://llmstxt.org/) before publishing. Useful for plug-in maintenance, not user-facing.

## Non-goals

- A WordPress-style "include external feeds" mode. This plug-in is about exposing the host blog's own content; cross-site aggregation belongs elsewhere.
- A built-in LLM crawler/cache. Static text files are the contract; how they're consumed is up to the client.
