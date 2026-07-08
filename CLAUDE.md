# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the Weinberg Lab website (weinberg-lab.github.io), a **Jekyll** site built on the [Lab Website Template](https://greene-lab.gitbook.io/lab-website-template-docs) and deployed via GitHub Pages. Most content is authored as Markdown + Liquid includes; there is little custom code. The distinctive machinery here is the automated **citation/publication pipeline** (Python + Manubot) and the GitHub Actions that drive builds and citation updates.

## Working in this repo

**Work within the template. Avoid new code unless there's a clear reason.** The template's value is that content is Markdown + Liquid includes with almost no custom code. Featured/filtered/restyled lists, featured citations, and member cards are all achievable by passing parameters to existing `_includes/*.html` or editing `_data/*.yaml`. Don't add new HTML/JS/SCSS or plugins unless the template genuinely can't do it — and if you must, say why. Keep changes scoped and reversible; never hand-edit `_data/citations.yaml` (CI regenerates it).

**Template docs are machine-readable — fetch on demand.** Index: `https://greene-lab.gitbook.io/lab-website-template-docs/llms.txt`. Any doc page is available as Markdown by appending `.md` to its URL (e.g. component refs at `…/basics/components/<name>.md`). Fetch the relevant page instead of guessing.

Deeper notes for AI agents (not published) live in `ai-docs/`.

## Local development

Local rendering runs through Docker (no need to install Ruby/Jekyll locally):

```bash
./.docker/run.sh          # builds image, serves at http://localhost:4000 with live reload
./.docker/run.sh <cmd>    # run an arbitrary command in the container
```

Alternatively, with a local Ruby toolchain: `bundle install` then `bundle exec jekyll serve`.

The citation pipeline is Python and runs separately:

```bash
pip install -r _cite/requirements.txt
python _cite/cite.py       # regenerates _data/citations.yaml (run from repo root)
```

`GOOGLE_SCHOLAR_API_KEY` (a SerpApi key) must be set in the environment for the google-scholar plugin; without it that plugin is skipped/errors but ORCID/PubMed/manual sources still work.

## Citation pipeline (the non-obvious core)

`_cite/cite.py` compiles publications into `_data/citations.yaml`, which pages then render. Do **not** hand-edit `_data/citations.yaml` — it is regenerated and overwritten.

Flow:
1. Plugins run in order: `google-scholar`, `pubmed`, `orcid`, `sources` (`_cite/plugins/*.py`).
2. Each plugin reads matching data files by name prefix: e.g. the `orcid` plugin processes every `_data/orcid*.yaml` (so `orcid-sam.yaml` matches), `sources` processes `_data/sources*.yaml`, etc. Plugins that hit third-party APIs ("metasources") expand one entry (an ORCID/Scholar id) into many sources; `sources.yaml` is the manual list.
3. Sources are merged by `id` (later entries override earlier), so a manually-entered entry in `sources.yaml` can enrich/override an auto-discovered one with the same DOI.
4. Each source's `id` (usually `doi:...`) is passed to **Manubot** to generate full citation metadata.

Practical editing patterns in `_data/sources.yaml`:
- Add a paper Manubot can't fetch: provide full fields manually under an entry.
- Feature/enrich an auto-found paper: add an entry with the same `id` plus `image`, `buttons`, etc.
- Hide a wrongly-included paper: add `{ id: <doi>, remove: true }`. (ORCID also pulls ISSN identifiers in as fake papers with `issn:...` ids — hide those the same way.)

Manubot failures are hard errors for manual `sources.yaml` entries but only warnings (source discarded) for API-discovered ones.

## Content structure

- `_config.yaml` — site title, links (email/orcid/google-scholar/github), Jekyll collections, plugins. Front-matter `defaults` assign layouts: everything → `default`, `_members/*` → `member`, `_posts/*` → `post`.
- **Pages** are `index.md` files per directory (`research/`, `publications/`, `team/`, `blog/`, `contact/`) plus root `index.md` and `404.md`. They are mostly Liquid `{% include ... %}` calls, not raw HTML.
- **Members**: one Markdown file per person in `_members/` with front matter (`name`, `image`, `role`, `aliases`, `links`). `role` maps to an icon/label via `_data/types.yaml`. `aliases` and `links.orcid`/`google-scholar` connect a member to their publications in the citation pipeline.
- **Blog posts**: `_posts/YYYY-MM-DD-title.md`.
- `_data/types.yaml` — maps roles and button/link/tag types to Font Awesome icons and default text.
- `_data/projects.yaml` — research project cards.
- `_includes/*.html` — reusable components invoked as `{% include name.html param=value %}` (e.g. `section.html`, `button.html`, `feature.html`, `card.html`, `citation.html`). `_layouts/` has the three page shells.
- `_styles/` (SCSS) and `_scripts/` are force-included via `_config.yaml`.

Rendering note: `list.html` accepts `filter=` and `style=`; `citation.html` shows an image/description/buttons/tags only when `style="rich"` (plain style is a clean text-only list). `citation.html lookup="<id or title fragment>"` renders a single hand-picked publication.

## CI/CD (`.github/workflows/`)

- **on-push** (push to `main`): regenerate citations → build → deploy live site.
- **on-pull-request**: regenerate citations for the PR branch → build a preview into the `preview/` folder.
- **on-schedule** (weekly, Monday 00:00 UTC): regenerate citations and open a PR (`citation-update`) if anything changed.
- `build-site.yaml` builds with Jekyll and deploys `_site` to the Pages branch. `update-citations.yaml` runs `_cite/cite.py` and commits/PRs changes to `_data/citations.yaml`.

Because citations are auto-updated by CI on push and weekly, expect `_data/citations.yaml` to change outside your edits.
