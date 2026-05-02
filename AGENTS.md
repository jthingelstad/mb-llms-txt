# Agent notes for `mb-llms-txt`

Context for AI coding assistants (Claude Code, Codex, etc.) working in this repo.

## What this is

A Micro.blog plug-in that publishes [llmstxt.org](https://llmstxt.org/) outputs (`/llms.txt`, `/llms-full.txt`) on any blog hosted on the Micro.blog platform. Lives in the [Micro.blog plug-in directory](https://micro.blog/account/plugins/register), distributed via this GitHub repo (`jthingelstad/mb-llms-txt`). Users install it from the directory or via "Install from URL" pointing here.

A Micro.blog "plug-in" is structurally a lightweight Hugo theme: a `plugin.json` with metadata + user fields, an optional `config.json` with Hugo config snippets, and a `layouts/` tree that layers on top of the host theme.

## Repo layout

| File | Purpose |
|---|---|
| `plugin.json` | Plug-in metadata + user-configurable fields (shown in the Micro.blog plug-in UI). `version` here is what the directory displays. |
| `config.json` | Hugo `mediaTypes` + `outputFormats` + `outputs` — registers the custom text outputs. |
| `layouts/index.llmstxt.txt` | Template for `/llms.txt` (home page, `llmstxt` output format). |
| `layouts/index.llmsfull.txt` | Template for `/llms-full.txt` (home page, `llmsfull` output format). |
| `icon.png` | 1024×1024 plug-in tile shown in the directory. Generated programmatically via `/tmp/make_icon.py` (Pillow). |
| `README.md` | User-facing docs. |
| `ROADMAP.md` | What's shipped, what's wanted, what's blocked. |
| `NOTES_FOR_MICROBLOG.md` | Outgoing draft note to Manton describing platform changes needed. Not for the public README. |
| `LICENSE` | MIT. |

## Hard-won constraints — read before changing layouts

Micro.blog's plug-in build pipeline has surprising behaviour. Two failure modes have been hit on live installs and are documented in `ROADMAP.md` and `NOTES_FOR_MICROBLOG.md`:

1. **Plug-in `outputs.page` is not merged** into the site config. Adding `"page": ["HTML", "markdown"]` to `config.json` does not make per-page `.md` files appear; the addition is silently dropped.
2. **Any `_default/single.*` template ships in the plug-in is dangerous.** Because the custom output format isn't actually registered (per #1), Hugo's lookup falls through and the `.md` template is applied to the *HTML* output of every post on the host blog — replacing the theme and rendering raw markdown in browsers. Confirmed on roadsignmath.xyz with v1.0.0 *and* v1.0.1 (the format-explicit `single.markdown.md` rename did not save us).

**Therefore**: do **not** add files under `layouts/_default/single.*` or `layouts/_default/list.*` until Micro.blog's pipeline supports plug-in `outputs.page` merge. Stick to home-output templates (`layouts/index.<format>.<ext>`) — those have proven to work end-to-end.

If the user reports Manton has shipped the platform fix, the per-post `.md` feature can be re-added — see "What needs to happen" in `ROADMAP.md` for the exact files to restore.

## Local testing setup

Hugo is installed via Homebrew (`hugo v0.161.x+`). A test site lives at `/tmp/mb-llms-test/` that symlinks `/Users/jamie/Projects/mb-llms-txt/layouts/` as a Hugo theme. To rebuild and inspect:

```bash
cd /tmp/mb-llms-test && rm -rf public && hugo --quiet
cat public/llms.txt
cat public/llms-full.txt
```

The test site has both titled posts and titleless microblog notes (the latter exercise the truncation fallback). Add fixture posts under `/tmp/mb-llms-test/content/post/` if you need to test new edge cases.

A second "basetheme" at `/tmp/mb-llms-test/themes/basetheme/` provides a stub `_default/single.html` so you can verify the plug-in doesn't override host-theme HTML. **Always run a local build with the basetheme present before shipping changes that touch `layouts/`.**

## Verifying on a live Micro.blog install

The user's test blog: **roadsignmath.xyz**. After pushing a new version:

1. User installs/updates the plug-in via Micro.blog's Plug-in UI.
2. User triggers a full site rebuild (Posts → Design → Rebuild Site).
3. Verification curls (script lives in conversation history, key checks):
   - `curl -s https://www.roadsignmath.xyz/llms.txt | head -30`
   - `curl -sI https://www.roadsignmath.xyz/llms.txt` (must be `text/plain; charset=utf-8`)
   - `curl -sI https://www.roadsignmath.xyz/feed.json https://www.roadsignmath.xyz/feed.xml` (both must still be 200 — confirms we didn't clobber Micro.blog defaults)
   - `curl -s https://www.roadsignmath.xyz/2023/10/21/genesis.html | head -3` (must look like real HTML, not raw markdown — the regression check for the v1.0.0/1.0.1 catastrophe)

## Release flow

1. Bump `plugin.json` `version`.
2. Commit + push to `main`.
3. The Micro.blog directory listing pulls from this repo on next user install/update; no separate submission needed for updates.
4. If the change is non-trivial, ask the user to test on roadsignmath.xyz before announcing more broadly.

Git config inherits from `~/.gitconfig` (Jamie Thingelstad). No special CI; plug-in is text files only.

## Style

- Templates use `{{- ... -}}` whitespace control liberally — the output is plain text and stray newlines matter.
- For Markdown output destined for LLM consumption, prefer `.RawContent` over `.Content` (the latter renders HTML).
- Use `htmlUnescape` on anything piped through `plainify` — Hugo's plainify converts smart quotes to entities, which look bad in plain-text output.
- User-facing text in `plugin.json` field labels should include the default value in parentheses (e.g. `"Number of posts in 'Recent Posts' section of /llms.txt (default 20)"`).
