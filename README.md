# DC Suite Documentation

Public documentation for **DC Suite** — a control plane for self-service GPU
cloud clusters on bare-metal infrastructure. Built with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and hosted on
Cloudflare Pages.

## Local preview

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve            # live-reload preview at http://127.0.0.1:8000
```

## Build

```bash
mkdocs build            # renders the static site into ./site
```

## Deploy to Cloudflare Pages

This repo builds cleanly on Cloudflare Pages with no extra configuration.

**Pages project settings**

| Setting | Value |
| --- | --- |
| Framework preset | None |
| Build command | `pip install -r requirements.txt && mkdocs build` |
| Build output directory | `site` |
| Environment variable | `PYTHON_VERSION = 3.12` |

Cloudflare Pages detects the `requirements.txt`, provisions Python, installs
MkDocs Material, and publishes the contents of `site/`. Every push to the
production branch triggers a rebuild; pull requests get preview deployments
automatically.

> **Tip:** to pin the exact MkDocs Material version across local and CI
> builds, replace the `>=` in `requirements.txt` with `==` and the version you
> tested against.

## Structure

```
docs/
  index.md                 # Landing page
  user-guide/              # For people using DC Suite to run GPU clusters
  operators/               # For teams deploying and running DC Suite itself
  reference/               # Glossary, FAQ, troubleshooting
mkdocs.yml                 # Site config + navigation
requirements.txt           # Build dependency (mkdocs-material)
```

## Contributing

Edit the Markdown under `docs/`, preview with `mkdocs serve`, and open a pull
request. The navigation lives in `mkdocs.yml` under `nav:`.
