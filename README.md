# Gako documentation

Product documentation for [Gako](https://gako.app/) — the administrator and
end-user guide. Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/)
and published to **https://docs.gako.app/**.

> [!NOTE]
> This is a **temporary** home. While the Gako source repository is private we
> publish docs from this public repo so GitHub Pages can serve them on the free
> plan. When `gako-app/gako` goes public, the contents of `docs/` move into
> `gako/docs/guide/`, the custom domain moves with them, and this repo is
> archived. The public URL (`docs.gako.app`) stays the same across that move.

## Local development

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve        # live-reloading preview at http://127.0.0.1:8000/
```

Build the static site (what CI publishes):

```sh
mkdocs build --strict   # --strict fails on broken links / nav
```

## Layout

| Path | Contents |
|---|---|
| `mkdocs.yml` | Site config and navigation |
| `docs/` | Markdown sources (one folder per top-level section) |
| `docs/CNAME` | Custom-domain marker (`docs.gako.app`), copied to the site root |
| `docs/stylesheets/extra.css` | Brand colors |
| `.github/workflows/deploy.yml` | Build + deploy to GitHub Pages on push to `main` |

## Publishing

Every push to `main` builds the site and deploys it to GitHub Pages via Actions.
See [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml).
