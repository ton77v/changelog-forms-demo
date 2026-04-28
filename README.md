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

Submit → bot replies with structured YAML that would land in `changelog/<version>.yaml`. Edit the issue → bot refreshes its comment.

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
Form    →   bot parses   →   (real action) commits YAML   →   PR
                                                                ↓
changelog.md  ←  render workflow  ←  merge to main
```

Three primitives: a form, a YAML file, a render. No DB, no admin UI, no extra tool.

## Stack

One official action: `actions/github-script@v7`. Pure JS parser inline. Zero third-party deps, no Python step. Cold-start ≈ 5s.

## Files

- `.github/ISSUE_TEMPLATE/changelog-entry.yml` — the form (2 fields)
- `.github/ISSUE_TEMPLATE/config.yml` — disables blank issues
- `.github/workflows/changelog_form_demo.yml` — parse form + comment (single step)
- `.github/workflows/render_changelog.yml` — regen `changelog.md` from YAMLs on push
- `changelog/1.41.0.yaml` — example YAML (single source of truth per release)
- `changelog.md` — auto-generated; editing this file is wrong, edit the YAML
