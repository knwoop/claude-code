---
name: platform-engineering-radar
description: Catch up on what shipped across Platform Engineering / SRE / Observability / Cloud Native — CNCF, Kubernetes, OpenTelemetry, Go / Rust / TypeScript releases, and influential practitioner blogs. Use when the user asks for the "latest", "this week", "catch-up", "trends", "what's new", or wants a weekly report or blog draft in this space. Combine with `/loop` or `/schedule` for recurring runs.
---

# Platform Engineering Radar

Information-collection skill for "what happened" across the Platform Engineering space. Fans out across CNCF, OpenTelemetry, language releases, practitioner blogs, **and the GitHub activity behind them** (commits, PRs, accepted proposals, notable issues), then produces a category-organized summary with implementation-level depth.

## When to use this skill

- Daily or weekly catch-up report
- Deep-dive on a specific topic (CNCF trends, OTel spec changes, Go runtime / library fixes, accepted proposals…)
- Blog post / internal-share doc draft
- Scouting topics for a reading club or lightning talk

This skill runs once per invocation. For recurring cadence, combine with `/loop` or `/schedule` — e.g. `/schedule daily 9am /platform-engineering-radar` or `/schedule weekly Monday 9am /platform-engineering-radar`. The skill itself stays stateless; scheduling lives in the runner.

## Core principles

1. **Sources live in `references/sources.md`** — the skill body stays stable; adding/removing sources happens in one file.
2. **Fan out with parallel `WebFetch` calls in a single turn** — never chain sequentially. Primary blogs, release pages, **and GitHub commit / issue / PR feeds all go in the same fan-out**.
3. **Blog silence is not community silence.** When a blog / release feed has no items in the window, do not drop the section — always fall through to commits, merged PRs, and accepted proposals on that project's repos. These have activity every day.
4. **Windows vary by source cadence** (see Step 1). Always state the window up front.
5. **Implementation-level depth is required.** For Go / OTel / Kubernetes, a report that lists only headline blog posts is a failed run. Every run should surface at least a handful of commit-, PR-, or proposal-grain items from these projects.
6. **Filter commits by interestingness, not recency** (see Step 3b). Skip dependency bumps, doc typos, test-only, pure internal refactors. Keep security fixes, behavior changes, perf changes with numbers, API additions/deprecations, notable proposal state-transitions, and discussions with core-maintainer engagement.
7. **Output is in English.** Preserve proper nouns, commands, API names, spec terms, commit hashes verbatim.
8. **Always attach the source list at the end** so the reader can trace back, including GitHub pages consulted.
9. **Rewrite every item in your own words** — no copy-paste of commit or blog text. Quoted phrases under 15 words, at most one quote per source.
10. **The skill is organization-neutral.** Stakeholder framing ("why it matters for our team") is opt-in via the `context=` arg or an explicit user hint.

## Workflow

### Step 1: Decide the scope

Extract four things from the user's request. If ambiguous, apply the default and state the assumption up front — do not ask back.

- **Categories**
  - Specified ("just CNCF", "OTel only") → only those
  - Unspecified → **all categories at `core` tier**
- **Time window** — defaults vary by source type. Language releases are slow, but **GitHub activity is constant**, so commit / issue feeds keep a short window.
  | Source type | Default window |
  | --- | --- |
  | Blog / Release feeds — CNCF / Kubernetes / Observability / Platform Eng / Cloud vendor / Influencers / Security / Media | **7 days** |
  | Blog / Release feeds — Go | **30 days** |
  | Blog / Release feeds — Rust, TypeScript | **90 days** |
  | **GitHub commits / PRs / issues / proposals — all projects** | **7 days** (or **3 days** in daily mode) |
  If the user specifies a window ("past 2 weeks", "past month", "today", "daily"), that overrides all categories uniformly. Presets: `daily` = 24–72h, `weekly` = 7d, `monthly` = 30d.
- **Output format**
  - Specified → follow it (blog draft / bullet summary / Obsidian note, etc.)
  - Unspecified → **category-organized Markdown summary**
- **Org context** (optional)
  - `context=<org-or-team>` arg, or a user hint like "for our platform team" → append a one-line relevance note under each item, framed for that context
  - Unspecified → omit org framing entirely

### Step 2: Consult the source list

Read `references/sources.md` and pull URLs for the target categories. Each entry carries a tier tag:

- **core**: always fetched on a default run
- **featured**: added when the user says "in detail", "broader", "full coverage"
- **archive**: only consulted when a specific topic is called out

Tier resolution:
- Default run → `core` only
- "In detail / full / broader" → `core` + `featured`
- Specific category called out → all tiers (`core` + `featured` + `archive`) become candidates

### Step 3a: Parallel `WebFetch` — blogs, releases, CHANGELOGs

Issue multiple `WebFetch` calls in the **same turn** as Step 3b. Sequential fetching wastes time — batch everything in one fan-out.

URL selection hints:
- Prefer `/releases/latest` or `CHANGELOG.md` on GitHub repos (explicit dates, easier to summarize)
- Blog landing pages are for scanning recent titles and excerpts — only chase individual posts if the headline is genuinely unclear
- Fall back to `WebSearch` only when a primary source can't be reached

### Step 3b: Parallel `WebFetch` — GitHub commits, PRs, proposals, issues

**This step is mandatory, not a fallback.** Run it in the same fan-out turn as Step 3a. The source list in `references/sources.md` tags GitHub commit / issue / PR feeds with the appropriate tier — fetch all the in-scope `core` entries (plus `featured` when in detail mode).

For each included project, you're looking for:

- **Commits** — `https://github.com/<org>/<repo>/commits/<default-branch>` shows recent merges. Read the commit list, identify interesting subjects (see filter below), and follow through to individual commits only if the subject line isn't self-explanatory.
- **Merged PRs** — `https://github.com/<org>/<repo>/pulls?q=is%3Apr+is%3Amerged+sort%3Aupdated-desc` for projects that squash-merge (commit subjects alone are uninformative).
- **Accepted proposals** — for Go: `https://github.com/golang/go/issues?q=is%3Aissue+label%3AProposal-Accepted+sort%3Aupdated-desc`. For Kubernetes: `kubernetes/enhancements` recently-updated PRs. Capture state transitions (Draft → Accepted, Accepted → Implemented, Active → Declined).
- **Hot issues** — `https://github.com/<org>/<repo>/issues?q=is%3Aissue+sort%3Acomments-desc+updated%3A>YYYY-MM-DD` for discussion-heavy threads. Include only when a core maintainer has engaged and the discussion implies a real direction.

#### Interestingness filter (applies to commits, PRs, and issues)

Keep an item only if at least one is true:

- **Security fix** — CVE reference, `security:` prefix, or commit message mentions an advisory
- **Behavior change** — default flip, deprecation, new opt-in/opt-out, removal
- **Performance change with concrete numbers** — "reduces allocation by 40%", "-15% p99"
- **Public API change** — addition, signature change, stability promotion (experimental → stable)
- **Bug fix in widely-used path** — stdlib, runtime, scheduler, GC, instrumentation hot path, signal-handling, networking — even without a CVE
- **Proposal state transition** — accepted, declined, entered active discussion, implemented
- **Discussion of note** — design debate with engagement from Russ Cox / Rob Pike / tracker leads / spec maintainers; push-back that changes a direction

Skip:

- Dependency bumps (unless they fix a CVE)
- Doc typos, README tweaks
- Test-only changes (unless they expose a subtle invariant)
- Pure internal refactors without observable effect
- CI / tooling chores
- Comment-only changes

When in doubt, ask: *"Would a working practitioner find this worth knowing?"* If no, drop it.

### Step 4: Filter by time window — never drop a section

Apply the window from Step 1 to every fetched item.

- Date present → strict filter
- Date missing (commit lists often omit dates on the landing page) → follow through or note "date unknown"
- **Never infer a date**; if unknown, say so

**Crucial rule:** if a category's Step 3a (blog / release) sources come up empty in the window, **do not drop the section**. The whole point of Step 3b is that Go / OTel / Kubernetes / major libraries always have activity — the report should reflect that. A section may be backed entirely by Step 3b commit-grain items; that's expected for quiet blog weeks.

Only drop a section if **both** Step 3a and Step 3b yield nothing passing the interestingness filter.

### Step 5: Dedupe across sources

The same release or announcement often shows up in multiple sources — e.g. a Kubernetes release appears in both `kubernetes.io/blog` and `cncf.io/blog`; an OTel spec cut appears in both `opentelemetry.io/blog` and the spec repo's releases page.

- Group items by topic (release tag, feature name, CVE id, incident handle, etc.)
- Keep the **primary source** — the one closest to the change:
  - Official project blog for releases
  - Spec or CHANGELOG for spec changes
  - GitHub release notes for detailed feature lists
  - Vendor engineering blog for vendor incidents / launches
- Drop duplicates from the main body; note them in the "Sources checked" list as `(dup of <kept-source>)`

### Step 6: Write the summary

Use the template below. A category is dropped only when **both blog-level and GitHub-level mining** yielded nothing passing the interestingness filter (see Step 4).

```markdown
# Platform Engineering Radar (as of YYYY-MM-DD)

> Scope: <categories> / Tier: <core | core+featured | ...>
> Windows: blogs 7d (Go 30d, Rust/TS 90d), GitHub 7d (or user-specified override)
> Fetched: N OK / M failed / K deduped / G GitHub feeds mined

## 🌐 Cloud Native / CNCF

- **[Post or release title](URL)** (YYYY-MM-DD) — `blog` / `release`
  - One or two lines summarizing the point in your own words.
- **[PR #12345: subject](URL) — kubernetes/kubernetes** (YYYY-MM-DD) — `commit/PR`
  - What changed, why it matters, any numbers the PR cites.

## 🔭 Observability / OpenTelemetry

(same shape; expect heavy Collector / Collector-Contrib / semantic-conventions commit coverage)

## 🐹 Go / 🦀 Rust / 🟦 TypeScript

(same shape; sub-headings per language are fine. For Go, expect commit / proposal items even in weeks without a blog post.)

### Go — proposals & issues

- **[#12345 proposal: title](URL)** — state: Accepted / Declined / Active
  - What's being proposed, who's driving it, what direction the latest comments suggest.

## 🏗️ Platform Engineering / SRE

(same shape)

## 🔥 Influencers / Performance

(same shape)

## ☁️ Cloud / Infra vendor blogs

(same shape)

---

## 📚 Sources checked this run

- <URL> — <category> — <OK / no items in window / fetch failed / dup of <URL>>
- ...
```

Each item carries:
1. Title link (commit subject / PR title / issue title / post title)
2. Date (or "date unknown")
3. A tag indicating the item type: `blog`, `release`, `commit/PR`, `proposal`, `issue`
4. 1–2 line summary in your own words, including concrete details (numbers, commit hashes, affected packages)

For commit/PR entries, include the org/repo so the reader can tell at a glance where the change lives (e.g. `kubernetes/kubernetes`, `open-telemetry/opentelemetry-collector-contrib`, `golang/go`, `google/uuid`).

If `context=<org>` was set (Step 1), add a one-line `_Relevance for <org>: …_`. Otherwise omit entirely.

### Step 7: Blog-draft mode (optional)

If the user says "make this a Zenn / Hatena draft" or "turn it into a blog post":

- Add a short intro paragraph (three lines on this run's headline topics)
- Append a one-line "why it matters" to each item
- Add a "References" section at the end
- Tone: measured, technical, engineer-to-engineer

## Output quality checklist

Before returning, verify:

- [ ] Time window stated at the top (including per-category overrides)
- [ ] Each item has (1) title link, (2) date or "date unknown", (3) type tag (`blog` / `release` / `commit/PR` / `proposal` / `issue`), (4) 1–2 line summary
- [ ] **Go, OpenTelemetry, and Kubernetes sections each contain at least one commit-, PR-, or proposal-grain item.** If any of these sections is blog-only, you have not run Step 3b properly — go back and mine the GitHub feeds.
- [ ] Commit/PR items name the repo (`org/repo`) and cite concrete details (commit hash, PR number, numbers, affected packages)
- [ ] Items pass the Step 3b interestingness filter — no dependency bumps, doc typos, test-only changes, or pure internal refactors
- [ ] Summaries are rewritten — no copy-paste of commit messages or blog text
- [ ] Quoted phrases stay under 15 words, at most once per source
- [ ] Uncertain claims use "reportedly", "appears to", or similar
- [ ] "Sources checked this run" section exists, listing OK / no-items / failed / dup — **including GitHub feeds**
- [ ] A category is dropped **only** when both its blog/release sources and its GitHub feeds came up empty after filtering
- [ ] Org framing appears only when `context=` or an equivalent hint was given

## Adding or removing sources

All sources and category assignments live in `references/sources.md`. Adding, re-tiering, or removing a source happens entirely in that file — no SKILL.md edits needed.

Maintenance rules for `references/sources.md`:

- New entries always carry a tier (`core` / `featured` / `archive`)
- Within a category, sort by tier ascending (`core` first)
- Sources that are persistently unreachable (404, consistently aborted) → demote to `archive` or remove
- Keep entries organization-neutral. Team-specific annotations ("we use this") belong in an optional overlay, not in this list.
