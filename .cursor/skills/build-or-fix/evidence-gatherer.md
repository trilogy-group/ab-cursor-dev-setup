---
name: evidence-gatherer
description: Instructions for capturing and posting evidence that a feature works. Read and follow these instructions as Step 5 of the build-or-fix skill.
---

# Evidence gathering

**Why:** A reviewer looking at your PR should be able to confirm the feature works without checking out the branch. Screenshots show what the user sees. Test results show edge cases are handled. Together they build confidence.

## Inputs you need before starting

1. The list of features/behaviours you need to provide evidence for.
2. The edge cases that matter (empty states, error handling, boundary conditions)
3. The PR number

## Prerequisites

Figure out how to run the system locally. Do not connect to any external services. If you can't run it locally, ask the user for help.

If a **Local Supabase:** is required. Start local Supabase with `supabase start` and use the local credentials it outputs. The client-side Supabase client in this project already defaults to `http://127.0.0.1:54321` when env vars are absent. For Playwright tests that need admin access, set `SUPABASE_SERVICE_ROLE_KEY` from `supabase status -o env`.

**Playwright browser:** Run `npx playwright install chromium && npx playwright install-deps chromium` if browsers are not installed.

## What to produce

1. **Screenshots** — Write a temporary Playwright spec (e.g. `tests/e2e/evidence-screenshots.spec.ts`) that navigates to the relevant pages and captures screenshots of key states (happy path, empty state, error state, filter/sort variations). Save to `evidence/`.

2. **Test results** — Run `PW_TEST_HTML_REPORT_OPEN=never npx playwright test <relevant-spec>` and capture the pass/fail summary.

3. **Evidence README** — Create `evidence/README.md` summarising the screenshots, test results, and edge case coverage.

4. **Commit** — Add `evidence/` to the branch. This is a temporary workaround — see below.

5. **Clean up** — Delete the temporary screenshot spec file after capturing. It served its purpose.

## Screenshot delivery

> **Workaround:** Cursor background agents currently cannot post PR comments due to a GitHub token permission limitation.
> Tracking: https://forum.cursor.com/t/background-agents-spawned-via-api-cannot-post-pr-comments-or-reviews-despite-correct-github-app-permissions/153207
>
> Until this is resolved, commit screenshots to an `evidence/` directory in the repo. Include a README noting they should be deleted after PR merge.

Once the GH token issue is resolved, switch to uploading screenshots via `gh pr comment` with embedded images instead of committing them.

## Keep it concise

The goal is reviewer confidence, not documentation volume. One screenshot of the happy path and one of an empty/error state is worth more than ten paragraphs of description.
