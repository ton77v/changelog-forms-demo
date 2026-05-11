# Implementation Plan — for reference

Demo greenlit by Mateusz K. + Simone (2026-05-10). This document records the agreed migration plan from `changelog.md` (free-text) → per-version YAML + GH Issue Forms, across three repos: this demo, `beshu-tech/readonlyrest-docs`, `beshu-tech/ror-api`.

Tracker: [beshu-tech/ror-api#86](https://github.com/beshu-tech/ror-api/issues/86)

---

# ROR Changelog: free-text MD → per-version YAML + Issue Forms UI

## Context

Replacing fragile `changelog.md` flow at `beshu-tech/readonlyrest-docs` with structured single source of truth + GH Issue Forms. Demo at `ton77v/changelog-forms-demo` greenlit by Mateusz K. + Simone. Hard ask from review: **restrict TO maintainers ONLY**. Mateusz preference: minimum-friction form (2 fields). Goal: kill regex-on-diff in `ror-api`, give devs a UI they actually use.

**User decisions locked:**
- Phase 1 first (demo polish), no real-repo changes yet
- Keep 2-field freeform notes form
- Migration script runs locally (one-shot), single PR for review
- 1-week dual-payload window at cutover
- Flow stays on `master` branch (releases tag from there)
- Version-deletion: in practice never happens (released = released). Keep `version_deleted` event-typed but minimal handler — log + skip, no DB mutation needed
- After cutover: `changelog.md` stays as auto-generated mirror (RSS/human browsing/external links)
- Migration script preserves bullet strings byte-exact (hash parity), no LLM normalization
- **`detailed_changelog.md` STAYS** — GitBook depends on it. Source becomes YAML + `ChangelogDBEntry.description` rows; portal still bot-pushes it to docs repo on `release_update` completion. Phase 4 keeps the file, just retargets its source.

---

## Repo setup prerequisites (apply once, every target repo)

These are **repo-level settings**, not code. The workflow files won't function without them. Apply in `ton77v/changelog-forms-demo` (done), `beshu-tech/readonlyrest-docs` (Phase 3), and any other target repo.

### 1. Allow Actions to create PRs

**Without this**: `peter-evans/create-pull-request@v6` fails at the final API call with:
> `##[error]GitHub Actions is not permitted to create or approve pull requests.`

(The branch push succeeds — only the PR-open call is blocked.)

**Fix** — Settings → Actions → General → Workflow permissions → check **"Allow GitHub Actions to create and approve pull requests"**.

Or via API (faster, idempotent):

```bash
gh api -X PUT repos/{owner}/{repo}/actions/permissions/workflow \
  -F default_workflow_permissions=read \
  -F can_approve_pull_request_reviews=true
```

Verify:
```bash
gh api repos/{owner}/{repo}/actions/permissions/workflow
# expect: {"default_workflow_permissions":"read","can_approve_pull_request_reviews":true}
```

`default_workflow_permissions` stays `read` — our workflows declare needed perms inline (`contents: write`, `pull-requests: write`). Don't widen the global default.

### 2. Issue labels

Form action applies `changelog-entry` label, gate filters on it. GitHub auto-creates labels on first use, but for clarity:

```bash
gh label create changelog-entry --repo {owner}/{repo} \
  --description "Auto-applied by Changelog entry form" --color 0E8A16
```

### 3. (Optional) Disable blank issues

`.github/ISSUE_TEMPLATE/config.yml` should have `blank_issues_enabled: false`. GitHub's native "Maintainers only" badge appears on Blank issue under this setting; can't be applied to custom forms.

---

## Phase 1 — Demo polish (`ton77v/changelog-forms-demo`)

### Add: maintainer gate

**File:** `.github/workflows/maintainer_gate.yml` (NEW)
- Trigger: `issues: [opened]`, label filter `changelog-entry`
- Check `github.event.issue.author_association ∈ {OWNER, MEMBER, COLLABORATOR}`
- Reject path: `gh issue close --reason not_planned --comment "..."` + `gh issue lock`
- Auth: `${{ github.token }}` (no PAT)
- **Race window**: ~10–30s issue is publicly visible before close. Accepted by Mateusz, flagged.

### Replace: mock-comment → real commit-YAML-and-PR

**File:** `.github/workflows/changelog_form_demo.yml` (MODIFY)
- Drop `peter-evans/find-comment` + comment-update logic
- Use `peter-evans/create-pull-request@v6`:
  - branch: `changelog/v{version}-issue-{number}` (idempotent → re-runs amend same branch)
  - commit message: `Add changelog entry for {version} (#{issue})`
  - PR title: same as auto-renamed issue title
  - Author: `github-actions[bot]` via `github.token` (sufficient, maintainers gate upstream)
- After PR created: comment PR link on issue, close issue
- **PAT decision:** stick with `github.token`. Validation runs on `push` to feature branch (which `peter-evans` does push). Downstream `pull_request` triggers won't fire from bot-authored PRs — acceptable since `render_changelog.yml`'s push trigger covers feature branches if we add `branches: [main, 'changelog/**']`.

### Add: pre-release version sorting

**File:** `.github/workflows/render_changelog.yml` (MODIFY at sort step ~lines 48-55)
- Replace `va.split('.').map(Number)` with `npm i -g semver` + `semver.rcompare(a, b)` in script step
- Verify against fixtures: `1.64.2`, `1.64.2-rc1`, `1.65.0-beta`, `1.69.1`

### Add: full schema validation

**File:** `.github/schemas/changelog-entry.schema.json` (NEW)

```json
{
  "type": "object",
  "required": ["version", "release_date", "entries"],
  "properties": {
    "version": {"type": "string", "pattern": "^\\d+\\.\\d+\\.\\d+(-(rc|beta|alpha)\\d*)?$"},
    "release_date": {"type": "string", "format": "date"},
    "entries": {
      "type": "array", "minItems": 1,
      "items": {
        "type": "object",
        "required": ["type", "components", "text"],
        "properties": {
          "type": {"enum": ["new", "fix", "security", "warning", "enhancement"]},
          "components": {"type": "array", "minItems": 1, "items": {"enum": ["es", "kbn"]}},
          "text": {"type": "string", "minLength": 1}
        }
      }
    }
  }
}
```

**File:** `.github/workflows/render_changelog.yml` (MODIFY)
- Add step before render: `npx ajv-cli validate -s .github/schemas/changelog-entry.schema.json -d "changelog/*.yaml"`
- Add `pull_request:` trigger with `paths: ['changelog/**.yaml']` so PRs validate before merge

### Demo exit criteria
- Non-maintainer files issue → auto-closes <30s
- Maintainer files issue → green-CI PR <2min, YAML committed, issue auto-closed with PR link
- 5 schema-violation cases (missing field, bad enum, bad date, bad version, empty entries) caught at PR-time
- `1.64.2-rc1` sorts above `1.64.2` (or whatever semver says — verify)
- README updated with new flow

---

## Phase 2 — Historical migration (local one-shot)

### Script: `scripts/migrate_changelog.py` (run on user's machine, NOT in CI)
- Input: `readonlyrest-docs/changelog.md` (798 entries)
- Logic:
  - Port parser from `src/integrations/repos/changelog/parse_changelog.py` (lines 37-58: `is_version_line`, `parse_version_details`)
  - For each version: extract bullets, parse `* **{emoji}{Type}** ({Component}) {text}`
  - Map emoji+label → schema enum (`🚀New` → `new`, `🐞Fix` → `fix`, `🚨Security Fix` → `security`, `⚠️Warning` → `warning`, `🧐Enhancement` → `enhancement`)
  - Components: `ES` → `[es]`, `KBN` → `[kbn]`, `ECK` → `[eck]`, combinable via `|` (e.g. `ES|KBN` → `[es, kbn]`, `KBN|ECK` → `[kbn, eck]`)
  - Edge cases (~1.5%: `KBN < 7.9.0`, `KBN|PRO`): emit YAML with raw component string + flag for manual fix
  - Date: parse from `### (YYYY-MM-DD) What's new in **ROR X.Y.Z**` heading
  - Output: `readonlyrest-docs/changelog/{version}.yaml`
- LLM-assist (Claude API) only for the ~1.5% ambiguous cases — not the 98%+ that fit clean regex

### Hash-stability gate (LOAD-BEARING)

**Reason:** if `HashTitles.from_md()` of YAML→MD-rendered titles differs from hash of original MD titles, all 798 LLM `description` rows in `version_details` table re-fire on next portal poll. Cost + time hit.

**Verification:** `scripts/verify_hash_parity.py`
- Port `off_spaces` + `strip_markdown` + `to_str` + `hash_string` from `src/integrations/repos/changelog/hash_titles.py`
- For each version:
  - render YAML back to MD bullet (exact `* **{emoji}{Label}** ({Comp}) {text}` format from existing `render_changelog.yml`)
  - compute `hash_string(",".join(off_spaces(strip_markdown(b)) for b in rendered_bullets))`
  - compare to `hash_string` of same fn applied to original MD bullets parsed via `parse_version_details`
  - **MUST match for every version** — fail-loud if any diff

### Phase 2 exit criteria
- 1 PR on `readonlyrest-docs` feature branch `migrate/yaml-changelog`
- 798 YAMLs + verify-script-pass output attached as PR comment
- `changelog.md` NOT yet deleted (kept until Phase 3 cutover)
- Mateusz approval

---

## Phase 3 — Production cutover (coordinated, 1-week dual-payload)

### `readonlyrest-docs` (master)

After Phase 2 PR merges:
- Add: `.github/ISSUE_TEMPLATE/changelog-entry.yml`, `.github/ISSUE_TEMPLATE/config.yml`
- Add: `.github/workflows/changelog_form.yml`, `render_changelog.yml`, `maintainer_gate.yml`
- Add: `.github/schemas/changelog-entry.schema.json`
- **Modify** `.github/workflows/changelog_watch.yml` — payload becomes:

  ```json
  {
    "event": "version_added | version_modified | version_deleted",
    "version": "1.64.2",
    "yaml": { "...parsed YAML for that version..." },
    "changelog_diff": "..."
  }
  ```

  (`changelog_diff` kept for 1-week dual-window only.)

- Detect change-type from git: filter `changelog/*.yaml` in last commit, classify A/M/D

### `ror-api` changes

**File:** `src/routes/api.py:78-107` (MODIFY) `update_changelog()`
- Accept new payload preferentially: if `req_data.get("version")` present → new path; else fall back to `find_versions_affected(req_data.get("changelog_diff", ""))`
- New path:
  - On `version_added` / `version_modified`: `add_version_release_jobs([Version(req_data["version"])])`
  - On `version_deleted`: log + skip. Per user: released versions don't get deleted in practice. No DB mutation needed; portal keeps row, next render of `changelog.md` won't include the version anyway since YAML is gone.

**File:** `src/integrations/repos/changelog/yaml_loader.py` (NEW)
- `load_version_yaml(version: Version) -> dict` — `safe_request` (mirror `get_changelog_raw` from `get_release_details.py`) for `https://raw.githubusercontent.com/beshu-tech/readonlyrest-docs/master/changelog/{version}.yaml`
- `yaml_to_md_entries(yaml_dict: dict) -> list[str]` — produces same bullet strings the LLM workflow expects (so `parse_version_details_str` output format preserved end-to-end through hash_titles)

**File:** `src/integrations/repos/changelog/get_release_details.py:15` (MODIFY) `get_release_details(version)`
- First try `yaml_loader.load_version_yaml`; on 404 (legacy versions, pre-cutover transition) fall back to `parse_changelog(get_changelog_raw(), version)`
- Phase 4 removes fallback

**File:** `src/integrations/repos/changelog/parse_changelog.py:15` (KEEP) `parse_changelog_structured()`
- Used by RSS/JSON renderers (`render_changelog.py:44, 98`) and `detailed_changelog.py:91-116`
- Don't touch in Phase 3; Phase 4 retires

**File:** `src/integrations/repos/changelog/find_versions_affected.py` (DEPRECATE, don't delete)
- Add `# DEPRECATED: phase-4 removal` header
- Still imported by `api.py:99` for fallback

### Tests (ror-api)

**File:** `tests/test_yaml_loader.py` (NEW)
- Fixture: `tests/abcs_mocks_utilities_etc/changelog/sample_1.64.2.yaml`
- Assert: `HashTitles.from_md(yaml_to_md_entries(yaml))` == `HashTitles.from_md(parse_version_details(original_md, idx))`
- Assert: 404 raises expected error type

**File:** `tests/test_changelog_webhook.py` (NEW or extend `tests/test_routes_api.py` if exists)
- New payload triggers single version job
- Old payload still works (dual-window)
- `version_deleted` triggers DB-delete job

### Phase 3 exit criteria
- Staging: file an issue → PR → merge → portal LLM workflow runs end-to-end on staging
- Prod: 1-week dual-window without errors in logs
- All ror-api tests green
- Mateusz files a real release via Issue Forms

---

## Phase 4 — Cleanup (after 1-week dual-window stable)

- Drop `changelog_diff` from `changelog_watch.yml` and `api.py:99` fallback
- Delete `find_versions_affected.py`
- Delete `parse_changelog()` (line 65); rewire `parse_changelog_structured` to iterate YAMLs
- **Retarget `detailed_changelog.py` source from MD-parsing → YAML-loading:**
  - `update_detailed_changelog()` @ `detailed_changelog.py:133` keeps producing `detailed_changelog.md`
  - Source switches: iterate `changelog/*.yaml` via new `yaml_loader`, hydrate with `ChangelogDBEntry.description` from DB
  - Bot-push pattern unchanged (commit + push back to `readonlyrest-docs`)
  - GitBook integration unaffected
- Update `docs/internal/llm_workflows.md`
- Update `readonlyrest-docs/README.md` contributor section

---

## Critical files to modify (real impl, by repo)

### `ton77v/changelog-forms-demo` (Phase 1)
- `.github/workflows/maintainer_gate.yml` (NEW)
- `.github/workflows/changelog_form_demo.yml` (MODIFY)
- `.github/workflows/render_changelog.yml` (MODIFY)
- `.github/schemas/changelog-entry.schema.json` (NEW)

### `beshu-tech/readonlyrest-docs` (Phases 2-3)
- `changelog/*.yaml` (NEW × 798)
- `.github/ISSUE_TEMPLATE/changelog-entry.yml` (NEW, copy from demo)
- `.github/ISSUE_TEMPLATE/config.yml` (NEW)
- `.github/workflows/changelog_form.yml` (NEW)
- `.github/workflows/render_changelog.yml` (NEW)
- `.github/workflows/maintainer_gate.yml` (NEW)
- `.github/workflows/changelog_watch.yml` (MODIFY → structured payload)
- `.github/schemas/changelog-entry.schema.json` (NEW)
- `README.md` (MODIFY — contributor section)
- `changelog.md` (kept as auto-gen mirror)
- `detailed_changelog.md` (kept — Phase 4 source-switches)

### `beshu-tech/ror-api` (Phase 3-4)
- `src/routes/api.py:78-107` (MODIFY)
- `src/integrations/repos/changelog/yaml_loader.py` (NEW)
- `src/integrations/repos/changelog/get_release_details.py:15` (MODIFY)
- `src/integrations/repos/changelog/parse_changelog.py:15,65` (MODIFY P3, partial DELETE P4)
- `src/integrations/repos/changelog/find_versions_affected.py` (DEPRECATE P3, DELETE P4)
- `src/integrations/repos/changelog/detailed_changelog.py` (RETARGET source P4, keep file)
- `tests/test_yaml_loader.py` (NEW)
- `tests/test_changelog_webhook.py` (NEW)
- `tests/abcs_mocks_utilities_etc/changelog/sample_*.yaml` (NEW fixtures)

### Existing functions to reuse
- `HashTitles.from_md` @ `hash_titles.py:38` — load-bearing identity check, must keep producing same hash
- `parse_version_details` @ `parse_changelog.py:48` — port to migration script
- `is_version_line` @ `parse_changelog.py:37` — port to migration script
- `safe_request` (in `get_release_details.py`) — pattern for new `load_version_yaml`
- `add_version_release_jobs` @ `release_update.py:62` — entry point for new payload

---

## Verification (end-to-end)

**Phase 1 demo:**
1. Open new issue from non-collaborator alt account → verify auto-close <30s
2. File from maintainer → verify PR appears, YAML present, issue closed
3. Add `1.64.2`, `1.64.2-rc1`, `1.65.0-beta`, `1.69.1` → verify desc order
4. Submit YAML missing required field → verify CI fails with schema error
5. Edit YAML in PR → verify re-validation runs

**Phase 2 migration:**
1. Run `scripts/migrate_changelog.py` → 798 YAMLs in working tree
2. Run `scripts/verify_hash_parity.py` → all-green or fail-loud listing
3. Render YAMLs → MD → diff against original `changelog.md` → byte-identical or documented exceptions
4. Push branch, open PR, Mateusz reviews

**Phase 3 cutover:**
1. Stg: trigger webhook with new payload, observe portal logs
2. Stg: trigger webhook with old payload (legacy fallback path), observe
3. Prod: monitor `release_update` task queue for 1 week, no extra LLM runs vs baseline
4. Prod: file 1 real release via Issue Forms end-to-end

**Phase 4:**
1. Run ror-api tests after deletions
2. Confirm RSS/JSON `/changelog` endpoint still works
3. Confirm `detailed_changelog.md` regenerates from new YAML+DB source, GitBook page intact

---

## Critical risks / decisions

1. **Hash stability is non-negotiable.** YAML→MD render must round-trip through `off_spaces(strip_markdown(...))` to identical hashes. Phase 2 verifier is the gate. Zero diffs or no Phase 3.
2. **Maintainer gate race window** ~10-30s. Accepted by Mateusz.
3. **`peter-evans/create-pull-request` + `github.token`**: bot-authored PRs don't trigger downstream `pull_request` workflows. Mitigate by adding `push: branches: ['main', 'changelog/**']` to `render_changelog.yml`.
4. **Pre-release semver:** `npm semver.rcompare` must agree with Python `packaging.version.Version` ordering used in ror-api (`parse_version`). Test fixture: `1.64.2-rc1` < `1.64.2` in both.
5. **Version deletion + diff:** new event-typed payload solves the false-positive massive-diff problem from YAML deletion. Don't try to handle deletions before Phase 3 cutover.
6. **`parse_changelog_structured` retirement (Phase 4)** must coordinate with RSS/JSON endpoint consumers. Verify zero external dependencies before deletion.
7. **`detailed_changelog.md` STAYS** for GitBook. Source-switches in Phase 4 from MD-parsing to YAML+DB. Bot-push pattern unchanged.

---

## Unresolved questions

(none — all decisions locked)
