# changelog-forms-demo

Proof-of-concept for **Issue Forms as a UI for the ROR changelog**.
Companion to [`beshu-tech/ror-api#86`](https://github.com/beshu-tech/ror-api/issues/86).

## What it does

1. Open a [new "Changelog entry" issue](../../issues/new/choose) — fill the form (dropdowns, validated version, etc.)
2. GitHub Action parses the form fields
3. Bot comments back with the **structured YAML** that would be committed to `changelog/<version>.yaml`

This is the *creation* path only. The full pipeline (real action) would also:

- Append entry to `changelog/<version>.yaml`
- Regenerate `changelog.md` from all YAMLs (deterministic template render)
- Open a PR; close issue on merge
- Push to master triggers downstream LLM workflow (portal-side) — keyed off **YAML hash**, not text diff

## Test it

→ [Open new issue](../../issues/new/choose) → pick "Changelog entry"

Try valid input first, then break it: leave version blank, type "1.68", invalid characters — the form blocks required fields, the action validates the version regex.

## Editing existing entries

**Out of scope** for forms/CLI. Edits = direct PR on the YAML file. That's git's strength; no tool needed.

## Files

- `.github/ISSUE_TEMPLATE/changelog-entry.yml` — the form schema
- `.github/ISSUE_TEMPLATE/config.yml` — disables blank issues
- `.github/workflows/changelog_form_demo.yml` — parse + comment action
