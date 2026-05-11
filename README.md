# changelog-forms-demo

**Two fields. One form. Single entry or whole release — same UX.**

Companion to [`beshu-tech/ror-api#86`](https://github.com/beshu-tech/ror-api/issues/86).

## Try it

→ [Open new issue](../../issues/new/choose) → "Changelog"

Paste one or many lines:

```
new (es) 9.0.2, 8.18.2, 8.17.7 support
new (kbn) 9.0.2, 8.18.2, 8.17.7 support
fix (es) [patching on Windows](https://forum.readonlyrest.com/t/...)
security (es) CVE-2024-53382
```

Submit → bot opens a PR with the YAML at `changelog/<version>.yaml`. Edit the issue → bot pushes new commit to the same PR.

On parse error → bot posts an updateable comment on the issue. Fix the form body → bot refreshes.

## Format

`<type> (<component>) <text>` — matches existing `changelog.md` style. Optional colon also accepted.

- **type**: `new`, `fix`, `security`, `warning`, `enhancement` (aliases: `feat`, `bugfix`, `sec`, `enh`)
- **component**: `es`, `kbn`, `es|kbn`
- **text**: any markdown — links, code, etc.

Lines starting with `#` or `//` are ignored.

## Why so few fields

Audited 798 entries from real `changelog.md`. 99% fit `<type> (<component>): <text>`. ~1.5% edge cases (`KBN < 7.9.0`, `KBN|PRO`) are handled by editing the generated YAML directly. Date is always per-version, defaults to today.

Anything beyond what survives that audit was friction without payoff.

## Editing existing entries

**No form needed — edit the YAML directly.**

Try it: open [`changelog/1.41.0.yaml`](changelog/1.41.0.yaml) → click the pencil → fix the wrong `release_date` (auto-set to today, real value should be `2022-06-21`) → "Propose change" → opens PR.

On merge to `main`, the **render workflow** regenerates [`changelog.md`](changelog.md) automatically. No manual MD editing ever.

Adding `1.0.1` to an existing "Kibana 1.0.0 support" entry? Open the YAML, change `1.0.0` to `1.0.0, 1.0.1`. Done. Same flow for any in-place fix.

## End-to-end loop

```
                          maintainer_gate (non-maintainer → close)
                          │
Form  →  parse  →  write YAML  →  open/refresh PR
                                       │
                                       ├── ajv schema validation
                                       └── filename ↔ inner version check
                                       │
                                       ↓ merge
                       render workflow → regen changelog.md
```

Three primitives: a form, a YAML file, a render. No DB, no admin UI, no extra tool.

## Guarantees (Phase 1)

- **Maintainer-only**: non-owner/member/collaborator issues auto-close <30s (`author_association` check)
- **Schema-validated**: every YAML must pass `.github/schemas/changelog-entry.schema.json` (ajv) before render
- **Filename = inner version**: render fails if `changelog/1.69.1.yaml` has `version: 1.69.0`
- **Pre-release sorting**: `1.69.0-rc1` orders correctly vs `1.69.0` (uses `semver.rcompare`)
- **Validation on PR**: `pull_request` trigger runs schema + filename check before merge

## Stack

Two npm deps installed per-run: `ajv-cli` (schema), `semver` (sort). One official action: `actions/github-script@v7`. One community action: `peter-evans/create-pull-request@v6` (idempotent branch + PR). Pure JS inline elsewhere.

## Files

- `.github/ISSUE_TEMPLATE/changelog-entry.yml` — the form (2 fields)
- `.github/ISSUE_TEMPLATE/config.yml` — disables blank issues
- `.github/workflows/maintainer_gate.yml` — close non-maintainer issues
- `.github/workflows/changelog_form.yml` — parse form + write YAML + open PR
- `.github/workflows/render_changelog.yml` — validate + regen `changelog.md`
- `.github/schemas/changelog-entry.schema.json` — single source of YAML truth
- `changelog/1.41.0.yaml` — example YAML (single source of truth per release)
- `changelog.md` — auto-generated; editing this file is wrong, edit the YAML
- [`PLAN.md`](PLAN.md) — full migration plan across 3 repos
