---
name: ship
description: Commit, push, and create a PR with ## What / ## Why description. English title and branch name following project conventions.
---

# Ship Skill

Commit changes, push, and create a Pull Request with a clean `## What` / `## Why` description format. Titles, branch names, and descriptions are all in English.

## Steps

### Step 1: Detect affected service

1. Determine base branch: use the user-provided base branch, or default to `main`. Store as `BASE_BRANCH`.
2. Run `git diff $BASE_BRANCH...HEAD --name-only` to list changed files.
3. Classify the primary affected service by file path:
   - **ios**: `mobile/kauche-ios/`
   - **android**: `mobile/kauche-android/`
   - **customer**: `server/services/customer/`
   - **web/farm**: `web/farm/`
   - **web/farm-line**: `web/farm-line/`
   - **web/farm-shared**: `web/farm-shared/`
   - **farm**: `server/services/farm/`
   - **recommend**: `server/services/recommend/`
   - **commerce-product-recommendation**: `server/services/commerce-product-recommendation/`
   - **partner**: `server/services/partner/`
   - **account**: `server/services/account/`
   - **walk**: `server/services/walk/`
   - **web/ope**: `web/ope/`
   - **web/partner**: `web/partner/`
   - **other**: none of the above or shared/common logic
   - **multiple**: changes span multiple services equally
4. If **other** or **multiple**, pick the most fitting label from the list above.
5. Store result as `SERVICE`.

### Step 2: Create commit

1. Read `.ai/commands/commit.md` and follow its instructions.
2. If the user requested staged-only mode, only commit already-staged files.
3. Otherwise, analyze all changes, stage relevant files, and commit with a conventional commit message.
4. **Do NOT append `Co-Authored-By` lines to commit messages.**

### Step 3: Push

1. Check if the remote branch exists:
   ```bash
   git ls-remote --heads origin $(git branch --show-current)
   ```
2. If no remote branch: `git push -u origin $(git branch --show-current)`
3. Otherwise: `git push`
4. Stop and notify the user on failure.

### Step 4: Draft PR title

**Never switch branches. All operations happen on the current branch.**

Analyze `git log $BASE_BRANCH..$CURRENT_BRANCH --oneline` and `git diff $BASE_BRANCH...$CURRENT_BRANCH` to understand the full set of changes.

#### PR Title rules

- **English only**
- Format: `{SERVICE}: concise summary of what this PR achieves`
- Under 70 characters
- Use conventional-commit style type prefix when appropriate (e.g. `fix(customer):`, `feat(farm):`, `refactor(web/farm):`, `docs:`, `chore:`)
- Examples from this repo:
  - `web/farm: replace IconGroup farmStatus props with individual fields`
  - `customer: remove gRPC status dependency from recommendation_service`
  - `fix(customer): move FarmCouponAutoApply lock to subtests`
  - `docs: introduce chromatic storybook`
  - `terraform/fastly: prevent Vary header from degrading cache hit rate`

### Step 5: Draft PR description

Write the ## What and ## Why sections. What is a **decision tool**, not a description or summary.

```markdown
## What

<One sentence: what problem disappears when this PR is merged?>

## Why

<What prompted this? What risk, pain, or requirement makes this necessary?>
```

**## What rules:**
- Lead with the **problem being solved**, not the solution or technology
- Do NOT describe implementation details — the diff shows "how"
- Do NOT write a summary or overview — What is a decision tool, not a description
- The reader should understand the value/outcome without reading code
- Write in plain sentences. Use bullet points only when listing truly independent items

**## Why rules:**
- Explain the **motivation and necessity** for the change
- Why now? What prompted this? (bug report, requirement, tech debt, performance issue, etc.)
- Write in plain sentences. Use bullet points only when listing truly independent items

**Optional sections:**
- If ticket numbers are provided, add `## Related Tickets` listing them (e.g. `KX-8610, KX-8611`)

### Step 6: Self-check PR description (MANDATORY — do not skip)

Before proceeding to Step 7, review EVERY sentence in ## What against all 4 filters below.
If ANY filter matches ANY sentence, rewrite that sentence and re-run the check.
Do NOT proceed to Step 7 until all sentences pass all filters.

**Filter 1 — Function/method/API name:**
Does the sentence mention a specific function, method, callback, API, or class name
(e.g. `set_after_send`, `useEffect`, `handleClick`, `SecretStore::open`)?
-> Remove it. Describe the outcome instead.

**Filter 2 — "How" verb:**
Does the sentence use verbs like "migrate to", "refactor into", "move X into Y",
"replace X with Y", "consolidate into", "extract into", "split into"?
-> These describe structural changes (how), not goals (what). Rewrite to describe what the user/system gains.

**Filter 3 — Technical mechanism:**
Does the sentence describe WHERE or HOW the code runs
(e.g. "in the callback", "at the middleware layer", "via a cron job", "using a goroutine")?
-> Remove the mechanism. Describe the behavior change.

**Filter 4 — Diff-readable:**
Could the reader learn this information just by reading the diff?
-> If yes, it's "how" and should not be in What.

**Examples:**
- BAD: "Move cache TTL control into the set_after_send callback" -- function name (filter 1) + "move into" (filter 2) + "callback" (filter 3)
- GOOD: "Ensure CDN caches only successful responses, with a default 7-day TTL as fallback for missing origin Cache-Control"
- BAD: "Replace fmt.Errorf with liberrors.Wrap across the repository layer" -- function names (filter 1) + "replace with" (filter 2) + "repository layer" (filter 3)
- GOOD: "Standardize error wrapping to include stack traces and structured context in all repository errors"

### Step 7: Create PR

Create the PR as **draft** by default. Use `--assignee @me`.

If the user specified:
- reviewers: add `--reviewer` flags
- open mode: omit `--draft`
- ticket numbers: include `## Related Tickets` section
- base branch: use it instead of `main`

After PR creation, print the PR URL.

## Usage examples

```
/ship                          # commit all, push, draft PR to main
/ship -s                       # staged files only
/ship -t KX-8610               # include ticket number
/ship -b release/v1.0          # target a release branch
/ship -r reviewer1,reviewer2   # assign reviewers
/ship -o                       # create as open (not draft)
/ship -s -t KX-8610 -o         # combine options
```
