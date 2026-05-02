# llms.txt — a plug-in for Micro.blog

<img src="https://raw.githubusercontent.com/jthingelstad/mb-llms-txt/main/icon.png" width="200" height="200" alt="llms.txt for Micro.blog">

Adds support for the [llmstxt.org](https://llmstxt.org/) standard to any Micro.blog hosted blog. Once installed, the plug-in publishes:

- **`/llms.txt`** — a structured Markdown index of your blog (title, description, recent posts, full year-by-year archive). This is what an LLM or AI agent fetches first to understand your site.
- **`/llms-full.txt`** — the full Markdown body of your most recent posts inlined into a single file (capped, configurable). Useful for one-shot context loads.

## Installation

1. In Micro.blog: **Plug-ins → Install from URL**, paste this repository's URL, click Install.
2. Optionally configure under **Plug-ins → llms.txt** (all settings have sensible defaults — installing without configuring just works).

## Settings

| Setting | Default | What it does |
|---|---|---|
| Intro blockquote | Site description | The blockquote at the top of `/llms.txt`. Markdown allowed. |
| Recent Posts count | 20 | How many posts appear under the "Recent Posts" section of `/llms.txt`. |
| `/llms-full.txt` post count | 100 | How many recent posts get inlined into `/llms-full.txt`. Set to **0** to disable `/llms-full.txt` entirely. |
| Include static pages | off | Include your static Pages in the `/llms.txt` index. |
| Include full archive | off | Include the full year-by-year list of every post in `/llms.txt`. Recommended for most blogs; turn off if you have many thousands of posts and the file gets unwieldy. |

## Designed to scale

Micro.blog blogs range from a few dozen posts to 20+ years and 18,000+ posts. The plug-in handles both:

- **Small blogs**: leave everything on, get a complete `/llms.txt` and a meaningful `/llms-full.txt`.
- **Large blogs**: turn off "Include full archive" and lower the `/llms-full.txt` post count. `/llms.txt` still gives a discoverable index of recent posts.

## Per-post Markdown alternates — not in this version

The llmstxt.org spec also recommends serving a clean Markdown alternate for every page (e.g. a `.md` companion to every post). Adding a per-page output format from a Micro.blog plug-in requires platform support that doesn't currently exist; if/when Micro.blog merges plug-in `outputs.page` config additions, that feature will be added here.

## Requirements

Micro.blog Hugo 0.91 or newer. Compatible with all standard themes.

## Acknowledgements

Built following the patterns of [`plugin-norobots`](https://github.com/microdotblog/plugin-norobots), [`plugin-search-page`](https://github.com/microdotblog/plugin-search-page), [`plugin-metatags`](https://github.com/microdotblog/plugin-metatags), and [`plugin-archive-months`](https://github.com/microdotblog/plugin-archive-months). Spec by Jeremy Howard at [llmstxt.org](https://llmstxt.org/).
