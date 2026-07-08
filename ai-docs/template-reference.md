# Lab Website Template — reference for AI agents

> This site is built on the **Greene Lab "Lab Website Template"** (Jekyll +
> GitHub Pages). This file is a map to the official docs plus the local specifics
> so an AI agent can work confidently without re-deriving everything.
> **Not published** — this folder is excluded from the Jekyll build.

## The official docs are machine-readable — fetch them on demand

The template docs (GitBook) expose an `llms.txt` index and a **`.md` version of
every page** (append `.md` to any doc URL). When you need detail on a component
or feature, fetch the relevant `.md` rather than guessing.

- **Index:** https://greene-lab.gitbook.io/lab-website-template-docs/llms.txt
- **Component reference** (each at `…/basics/components/<name>.md`):
  `section`, `figure`, `button`, `icon`, `feature`, `list`, `citation`, `card`,
  `portrait`, `post-excerpt`, `alert`, `tags`, `float`, `grid`, `cols`, `search`,
  `site-search`
- **Core topics:**
  - Citations pipeline: `…/basics/citations.md`
  - Components overview: `…/basics/components.md`
  - Team members: `…/basics/team-members.md`
  - Blog posts: `…/basics/blog-posts.md`
  - Configure the site: `…/basics/configure-your-site.md`
  - Repo structure: `…/basics/repo-structure.md`
  - Customize the theme: `…/basics/customize-your-theme.md`
  - Data & collections: `…/advanced/data-and-collections.md`
  - Custom components: `…/advanced/custom-components.md`
  - Embeds / math / diagrams: `…/advanced/embeds.md`, `…/advanced/math-diagrams-videos-etc..md`

## How this repo is wired (local specifics)

- **Pages** are `index.md` per directory (`research/`, `publications/`, `team/`,
  `blog/`, `contact/`) plus root `index.md` and `404.md`. They are mostly Liquid
  `{% include ... %}` calls, not raw HTML.
- **Components** live in `_includes/*.html`, invoked as
  `{% include name.html param=value %}`. Layouts are in `_layouts/`
  (`default`, `member`, `post`).
- **Members**: one file per person in `_members/`. Front matter: `name`, `image`,
  `role`, `description`, `aliases`, `links`. `role` maps to an icon/label via
  `_data/types.yaml`. `aliases` + `links.orcid`/`google-scholar` connect a member
  to their publications in the citation pipeline.
- **Data**: `_data/types.yaml` (role/button/link/tag icons + labels),
  `_data/projects.yaml` (Research-page cards), `_data/sources.yaml` (manual
  publication list / overrides), `_data/citations.yaml` (**generated — never
  hand-edit**).
- **Styling** is `_styles/*.scss`; scripts are `_scripts/*.js`. Both are
  force-included via `_config.yaml`.

## Citation pipeline (the non-obvious core)

`_cite/cite.py` compiles publications into `_data/citations.yaml`, which pages
render. **Do not hand-edit `citations.yaml`** — CI regenerates it on push and
weekly.

- Plugins run in order (`_cite/plugins/*.py`): `google-scholar`, `pubmed`,
  `orcid`, `sources`. Each reads `_data/<plugin>*.yaml` by name prefix (e.g. the
  `orcid` plugin reads every `_data/orcid*.yaml`, so `orcid-sam.yaml` matches).
- Sources merge by `id` (usually `doi:...`); later entries override earlier, so a
  manual `sources.yaml` entry can enrich or override an auto-discovered one.
- Editing patterns in `_data/sources.yaml`:
  - Add a paper Manubot can't fetch → full manual entry.
  - Feature/enrich an auto-found paper → entry with the same `id` plus `image`,
    `buttons`, `description`, `tags`.
  - Hide a wrongly-included item → `{ id: <doi>, remove: true }`.
- **Known noise source:** ORCID pulls ISSN identifiers in as fake "papers"
  (they arrive as `issn:...` ids). Hide them with `remove: true`.

## Rendering notes worth knowing

- `list.html` accepts a `filter=` expression (e.g. `filter="group == 'featured'"`)
  and a `style=` — so publication/project lists can be narrowed or restyled
  without touching component code.
- `citation.html` only renders the **image, description, buttons, and tags** when
  `style="rich"`. Plain style (omit `style`) shows just icon + title + authors +
  publisher · date · id — a clean text list with no image. Use plain style for
  long "All publications" lists to avoid a wall of placeholder images; reserve
  `rich` for a short curated "Selected" set where real figure images exist.
- `citation.html lookup="<id or title fragment>"` renders a single publication by
  matching `citations.yaml`; use it to hand-pick featured papers.
