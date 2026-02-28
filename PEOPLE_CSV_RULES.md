# `people.csv` Maintenance Rules

This document defines how to update alumni data safely and consistently.

## Source of truth

- Data file: `assets/data/people.csv`
- Header (must not change):
  - `korname,engname,update,aff,bs,ms,phd,advisor,advisor2,handle,debut,mathgen,mrauthid,scholar,website,thesis,hide`

## Field definitions

- `korname`: Korean name
- `engname`: English name
- `update`: latest update time in `YYYYMM` (e.g. `202603`)
- `aff`: current affiliation (department/unit)
- `bs`: BS admission year (학번, not graduation year)
- `ms`: MS admission year
- `phd`: PhD graduation year
- `advisor`, `advisor2`: PhD advisor(s)
- `handle`: KAIST thesis handle ID only (e.g. `308564`, not full URL)
- `debut`: PhD year (same value as `phd` for new PhD alumni entries)
- `mathgen`: Mathematics Genealogy ID
- `mrauthid`: AMS MathSciNet Author ID
- `scholar`: Google Scholar user key (e.g. `0VOzmFEAAAAJ`)
- `website`: personal/professional website URL
- `thesis`: exact PhD thesis title
- `hide`: non-empty value means hidden from listings

## Required rules for new PhD alumni

1. `debut` must equal `phd`.
2. `handle` must be set if `thesis` is provided.
3. `thesis` must exactly match the title shown on the handle page.
4. Never guess `scholar`, `mrauthid`, `mathgen`. Leave blank if not confirmed.

## ID lookup references

- Handle (KAIST thesis): `http://hdl.handle.net/10203/<handle>`
- Mathematics Genealogy: <https://www.mathgenealogy.org/search.php>
- MathSciNet Author ID: <https://mathscinet.ams.org/mathscinet/authors-search>
- Google Scholar key: search by name on <https://scholar.google.com>

## Mandatory verification gate (for every newly added PhD alumni row)

1. Open `http://hdl.handle.net/10203/<handle>`.
2. Copy the thesis title shown on that page.
3. Compare against `thesis` in `assets/data/people.csv` (character-level exact match).
4. If mismatch, fix `thesis` and re-check before completing the task.

## Editing and formatting rules

- Keep one row per person.
- Keep the column count exactly 17 for every row.
- Preserve CSV quoting where needed (commas/quotes in field text).
- Keep Korean-name ordering convention used in the file.
- Keep line endings consistent across the file (mixed line endings can break CSV parsing/builds).

## Post-update checks

1. Duplicate check:
   - Confirm target name/English name appears exactly once.
2. CSV integrity check:
   - Parse CSV and verify all rows have 17 columns.
3. Rendering check:
   - `hugo` build should succeed.
   - Verify visibility on:
     - `index`
     - `undergraduate`
     - `phd`
     - `advisor`
     - `latest`

## Quick checklist

- [ ] Row added/updated in `assets/data/people.csv`
- [ ] `debut == phd` (for PhD entries)
- [ ] `handle` confirmed
- [ ] `thesis` exactly matches handle page
- [ ] `mathgen`/`mrauthid`/`scholar` confirmed or left blank
- [ ] Duplicate/CSV/build checks passed

