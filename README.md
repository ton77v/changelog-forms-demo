# changelog-forms-demo

**Two fields. One form. Single entry or whole release — same UX.**

Companion to [`beshu-tech/ror-api#86`](https://github.com/beshu-tech/ror-api/issues/86).

## Try it

→ [Open new issue](../../issues/new/choose) → "Changelog"

Paste one or many lines:

```
new (es): 9.0.2, 8.18.2, 8.17.7 support
new (kbn): 9.0.2, 8.18.2, 8.17.7 support
fix (es): [patching on Windows](https://forum.readonlyrest.com/t/...)
security (es): CVE-2024-53382
```

Submit → bot replies with structured YAML that would land in `changelog/<version>.yaml`. Edit the issue → bot refreshes its comment.

## Format

`<type> (<component>): <text>`

- **type**: `new`, `fix`, `security`, `warning`, `enhancement` (aliases: `feat`, `bugfix`, `sec`, `enh`)
- **component**: `es`, `kbn`, `es|kbn`
- **text**: any markdown — links, code, etc.

Lines starting with `#` or `//` are ignored.

## Why so few fields

Audited 798 entries from real `changelog.md`. 99% fit `<type> (<component>): <text>`. ~1.5% edge cases (`KBN < 7.9.0`, `KBN|PRO`) are handled by editing the generated YAML directly. Date is always per-version, defaults to today.

Anything beyond what survives that audit was friction without payoff.

## Editing existing entries

**Out of scope.** Edits = PR on the YAML file. Devs already PR daily; no UI helps here.

For "add 1.0.1 to existing Kibana support" → file a new entry: `new (kbn): 1.0.1 support`. Render-time grouping handles presentation.

## Stack

One official action: `actions/github-script@v7`. Pure JS parser inline. Zero third-party deps, no Python step. Cold-start ≈ 5s.

## Files

- `.github/ISSUE_TEMPLATE/changelog-entry.yml` — the form (2 fields)
- `.github/ISSUE_TEMPLATE/config.yml` — disables blank issues
- `.github/workflows/changelog_form_demo.yml` — parse + comment (single step)
