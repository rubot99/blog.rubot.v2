# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Source for rubot.co.uk, a personal blog (software architecture notes + book reviews) built with MkDocs Material's blog plugin. Python dependencies are managed with `uv`; the site is deployed to Netlify.

## Commands

```bash
uv sync              # install/sync dependencies from uv.lock
uv run mkdocs serve  # local dev server at http://127.0.0.1:8000, rebuilds live, shows drafts
uv run mkdocs build  # production build to site/ (mirrors what Netlify runs)
```

There is no test suite or linter configured in this repo.

## Architecture notes

These are the parts that aren't obvious from any single file:

- **The blog *is* the homepage.** `plugins.blog.blog_dir: .` in `mkdocs.yml` makes the blog plugin auto-generate `index.md` (the post feed) at build time. **Never hand-author `docs/index.md`** — a manually created one silently wins over the generated feed. This has regressed more than once during development; if the homepage stops showing the post feed, check for a stray `docs/index.md`.
- **`nav:` is explicitly defined** (to include the About page), which disables the blog plugin's automatic nav generation. Per Material's docs, when `nav:` is defined you must include the blog index page (`index.md`) in it, and must *not* list individual posts — they're still handled automatically.
- **`plugins.blog.draft` controls the production build, not draft support itself.** `draft: true` means draft posts *are included* in the build (counterintuitive) — the default/desired value here is `false`. `draft_on_serve` (default `true`) is the separate, independent setting that shows drafts during `mkdocs serve`. Getting this backwards previously caused a draft post to go live on production.
- **Post front matter** uses a nested date field, not a flat scalar:
  ```yaml
  date:
    created: 2026-07-02
    updated: 2026-07-03
  categories:
    - Architecture   # restricted to plugins.blog.categories_allowed in mkdocs.yml
  tags:
    - technology
  authors:
    - rubot           # keys defined in docs/.authors.yml
  draft: true
  ```
  The RSS plugin (`plugins.rss.date_from_meta`) reads `date.created`/`date.updated` specifically, so posts must use this nested structure for feed dates to populate correctly.
- **`docs/tags.md`** is the landing page the `tags` plugin renders its tag index into (`tags_file: tags.md`) — it must exist for that page to work, even if its content is minimal.
- **`overrides/main.html`** is a Material theme override (`theme.custom_dir: overrides`) extending `base.html`. It owns the site's one `extrahead` block, which currently holds *both* the GoatCounter analytics script and the Open Graph/Twitter card meta tags — a Jinja block can only be defined once per template, so any future additions to `<head>` need to be merged into this same block rather than added via a new override file.
- **Social preview image is a single static file** (`docs/assets/social-card.png`), reused across all posts rather than generated per-post. This was a deliberate choice over MkDocs Material's built-in social-cards plugin, which depends on Cairo/Pillow native libraries that are fragile to get working on Netlify's build image.
- **Netlify build** (`netlify.toml`) installs `uv` itself via the official install script — Netlify's build image doesn't ship it — before running `uv sync && uv run mkdocs build`. Publish directory is `site/`.
- **Two separate licenses**: the repo's code (`LICENSE`) vs. the blog content itself, which is CC BY 4.0 (declared in `mkdocs.yml`'s `copyright` field, shown in the site footer).

## Known upstream risk

The Material for MkDocs maintainers have publicly warned that the upcoming MkDocs 2.0 will break all plugins and all theme overrides (including the `custom_dir` mechanism this repo relies on) with no migration path. Worth checking the state of that before any future MkDocs/Material major-version upgrade.
