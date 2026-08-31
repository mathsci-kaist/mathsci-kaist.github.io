# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Hugo static site listing KAIST Department of Mathematical Sciences alumni who are in academia. Deployed to GitHub Pages. All page content is generated from a single CSV; there are no per-person content files.

## Commands

```bash
hugo server        # local preview at http://localhost:1313
```

```bash
hugo --minify      # production build into public/ (same command CI runs)
```

CI (`.github/workflows/hugo.yml`) builds with Hugo **extended** 0.161.1 on push to `main` and deploys `public/` via GitHub Actions. GitHub Pages `Source` must be set to *GitHub Actions*, not *Deploy from a branch*. The templates use APIs that require ≥ 0.158 (`locale` in hugo.toml, `hugo.Data` in `navbar.html`), so lowering the CI pin will break the build — if a build behaves differently locally, check the version first.

There are no tests or linters. "Verification" for a data change means: `hugo --minify` succeeds, plus a visual check of `/`, `/undergraduate.html`, `/master.html`, `/phd.html`, `/advisor.html`, `/latest.html`.

## Data flow / architecture

`assets/data/people.csv` is the single source of truth. Everything else is rendering.

1. `layouts/partials/people/load.html` — the only place the CSV is read (`readFile` + `transform.Unmarshal`, project-root-relative path, *not* the resource pipeline). It zips the header row against each data row into a map per person, so **CSV column names are the field names used throughout the templates**. Renaming a header renames the template field. It also derives `display_engname`, `sort_group`, and `sort_*` keys, and drops any row with a non-empty `hide`. All values are strings — year comparisons in templates are string comparisons.
2. `layouts/partials/people/sorted.html` — canonical ordering: rows without a Korean name first (sorted by `englast/engfirst/korname`), then Korean-named rows by `korname`.
3. `layouts/partials/views/*.html` — one partial per page view: `index`, `bs`, `ms`, `phd`, `advisor`, `latest`. Each calls `load.html` itself (the CSV is parsed once per view, not shared).
4. `layouts/partials/people/person.html` / `person_thesis.html` — row renderers; `person_thesis.html` adds the thesis line linked to `hdl.handle.net/10203/<handle>`. `name.html` renders the name plus the external-link icons (🎓 mathgen, 🌿 mrauthid, 📚 scholar) and the 🎈 badge when `update` is within the last year.

### Wiring a page to a view

`content/*.html` files carry only front matter (`title`, `url`, `view`) and a heading. `layouts/_default/single.html` dispatches on `.Params.view` through an explicit if/else chain — **adding a new view requires editing both the content file and that chain**. The homepage is the exception: `content/_index.html` is rendered by `layouts/index.html`, which hardcodes `views/index.html`.

Nav tabs come from `data/menu.yml` (title + url); a new page needs an entry there too. Active-tab detection compares `.url` to `.RelPermalink`, so the `url` in menu.yml must match the content file's `url` exactly.

### View-specific quirks worth knowing

- `class.html` renders only the keys the caller passes in the dict, which is how each view suppresses its own axis (e.g. the PhD page omits the PhD link). When adding a view, pass the dict explicitly.
- `views/latest.html` only shows people with `update` > `202301` (string compare).
- `views/advisor.html` splits advisor names into two buckets by comparing against the literal `"zzzzz"` to get Korean names before/after Latin ones, then sorts each. Advisor grouping unions `advisor` and `advisor2`.
- `thesis` is emitted with `safeHTML` (inline markup like `<sub>` is allowed); `aff` goes through `plainify`.
- `debut` exists in the CSV but is currently unused by any template.

Styling: `assets/css/style.scss` compiled by Hugo (requires the extended build) on top of CDN Bootstrap 5.3.3 loaded in `layouts/_default/baseof.html`. `layouts/404.html` is a standalone page with inline CSS.

## Editing people.csv

`PEOPLE_CSV_RULES.md` is normative — read it before touching `assets/data/people.csv`. The rules that matter most:

- Header must stay exactly 18 columns, and every row must have 18 columns.
- `bs`/`ms` are **admission** years (학번); `phd` is the **graduation** year. `debut` must equal `phd` for PhD entries.
- `handle` is the bare KAIST thesis handle ID, not a URL. If `thesis` is set, `handle` must be set.
- Mandatory gate for any newly added PhD row: open `http://hdl.handle.net/10203/<handle>` and confirm `thesis` matches the title there character for character.
- Never guess `scholar`, `mrauthid`, or `mathgen` — leave blank if unconfirmed.
- `update` is `YYYYMM`; bump it when refreshing a person's info so they surface on `/latest.html`.
- Check for duplicates (name should appear exactly once) after adding a row.
