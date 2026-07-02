# blog.rubot.v2

Source for [rubot.co.uk](https://rubot.co.uk) — personal blog covering software architecture and book reviews.

Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/), dependencies managed with [uv](https://docs.astral.sh/uv/), deployed on [Netlify](https://www.netlify.com/).

## Local development

```bash
uv sync
uv run mkdocs serve
```

Then open `http://127.0.0.1:8000`. The server rebuilds live as files change.

## Writing a post

Add a new file under `docs/posts/`, e.g. `docs/posts/2026-07-02-my-post.md`:

```markdown
---
date: 2026-07-02
categories:
  - Architecture   # or: Books
draft: true
---

Opening paragraph shown in the post feed.

<!-- more -->

Rest of the post, only shown on the full post page.
```

Leave `draft: true` while writing — draft posts are visible locally (`mkdocs serve`) but excluded from the production build. Flip it to `false` (or remove it) when ready to publish.

Book review posts additionally use `author`, `rating`, and `link` front matter fields for structured metadata.

## Deployment

Netlify auto-deploys on every push to `main` — there's no separate review/staging step, so only flip a post out of draft when it's ready to be public.

## License

- **Code** (site config, templates, tooling): see [LICENSE](LICENSE).
- **Content** (blog posts): [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — reuse with attribution.
