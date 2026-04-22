# Harbinger In-House SEO Pipeline

AI-automated SEO pipeline for Harbinger Marketing partners, replacing the prior external vendor (eBrandz). Built around six Claude Skills that handle the full cycle from keyword research through content production, backlink outreach, and performance reporting.

This repo contains the skills, connector documentation, and process docs. Partner data lives in Airtable (source of truth for profiles) and in per-partner working directories (for cycle outputs). Claude Code desktop app is the primary interface for running the pipeline.

---

## Status

**Current phase**: MVP migration (targeting October 1, 2026 vendor wind-down)

**Skills**: All six skills installed and ready for use
**Connectors**: Airtable and DataForSEO connected via MCP
**Testing**: Individual skills being validated before first live-partner runs
**Active partners in pipeline**: 0 (pre-migration)

---

## Repository structure

```
/harbinger-seo/
  /skills/                          # The six Claude Skills
    seo-keyword-research/
    seo-strategy-light/
    seo-technical-package/
    seo-body-content/
    seo-outreach-draft/
    seo-reporting-analyst/
  /docs/                            # Process documentation
    mvp_playbook.md                 # The MVP workflow (8 processes)
    full_process_playbook.md        # The full 16-process pipeline (Phase 2+)
    technical_seo_standards.md      # Reference for the technical skill
  /examples/                        # Shared example data (non-confidential)
  README.md                         # This file
  .gitignore                        # Excludes partner data folders
```

**Not in the repo** — partner data. Real partner profiles live in Airtable; cycle working outputs (keywords.json, strategy.json, technical.json, body copy) live in a local working directory outside the repo. The repo holds the pipeline; partner data flows through it.

---

## The six skills

Each skill is a folder in `/skills/` with a `SKILL.md`, supporting `rules/` files, and worked `examples/`. Claude loads them natively in Claude Code when invoked.

| Skill | Purpose | Runs when |
|---|---|---|
| `seo-keyword-research` | Produces a scored, clustered target keyword list from partner profile + GSC history + DataForSEO volume/difficulty | At cycle start |
| `seo-strategy-light` | Converts approved keyword list into a page roadmap (create new / optimize existing / consolidate-redirect decisions) with internal linking plan and FAQ bank | Immediately after keyword research |
| `seo-technical-package` | Produces the technical SEO wrapper for a single page: title, meta, canonical, H1/H2/H3 outline, schema JSON-LD, internal link placements, image specs | Per page, after strategy is approved |
| `seo-body-content` | Writes the body copy that fills the technical package's outline. Honors the internal linking plan exactly. | Per page, after technical package |
| `seo-outreach-draft` | Drafts 3-email outreach sequences (initial + 2 follow-ups) for backlink acquisition, across 7 angles (resource page, guest post, broken link, local partnership, expert source, testimonial, data share) | Per prospect, throughout the cycle |
| `seo-reporting-analyst` | Answers ad-hoc analytical and reporting questions about partners, cycles, or the pipeline. Handles trend analysis, diagnostics, comparatives, pipeline-wide aggregates, cycle-close summaries, and partner-facing deliverables | On demand |

Open each skill's `SKILL.md` for full documentation on inputs, outputs, and usage.

---

## The end-to-end workflow

A partner's full cycle flows through the skills in this order:

### 1. Onboarding (~3.5-4.5 hours, one-time per partner)

Done before any skill invocation. Captures a baseline snapshot (top 20 keyword rankings, 90-day GSC metrics, indexed page count, backlink profile), runs a Screaming Frog audit for critical technical issues, pulls the existing backlink profile for vendor-era disavow analysis. All results stored in the partner's working directory and (for durable data) in Airtable.

### 2. Cycle strategy (~1-1.5 hours per partner, at cycle start)

**Step A**: Invoke `seo-keyword-research` with the partner profile from Airtable, GSC historical queries, and competitor domains. Output is the scored keyword list.

**Step B**: Invoke `seo-strategy-light` with the approved keyword list and site crawl data. Output is the page roadmap with internal linking plan and FAQ bank.

**Step C**: Engineer exports the roadmap for MD/MC review in Google Docs. MD/MC approves or requests changes.

### 3. Content production (~50 min per page, 3-4 pages per cycle)

For each approved page in the roadmap:

**Step A**: Invoke `seo-technical-package` with the page details, partner profile, and relevant FAQ set. Output is the technical SEO wrapper (title, meta, canonical, schema, outline, internal links, image specs).

**Step B**: Invoke `seo-body-content` with the technical package output and the partner profile. Output is the body copy in markdown, with every internal link from the plan placed correctly.

**Step C**: Engineer validates schema via Google Rich Results Test, reviews body copy for voice and factual accuracy, and pastes into the partner's CMS (WordPress or Webflow).

### 4. Backlink outreach (variable cadence, throughout the cycle)

For each approved prospect:

Invoke `seo-outreach-draft` with prospect details, partner value prop, and angle selection. Output is a 3-email sequence (initial + 2 follow-ups). Engineer sends manually via Gmail.

### 5. Reporting (on demand, and at cycle close)

Invoke `seo-reporting-analyst` with any analytical question: trend analysis, performance diagnostic, cycle-close summary, or partner-facing update. Output varies by question type.

### 6. Monitoring (ongoing, monthly + weekly during the 90-day transition window)

No skill runs here — the engineer performs GSC spot-checks monthly and weekly ranking checks for partners in their first 90 days post-migration.

---

## Running skills

### How Claude Code uses the skills

With skills installed in Claude Code desktop and the Airtable + DataForSEO connectors active, a typical skill invocation is conversational:

```
You: Run keyword research for Mile High Roofing.

Claude: [queries Airtable for the Mile High Roofing partner record]
        [pulls recent GSC data — either from cache or fresh via connector]
        [invokes DataForSEO connector for volume/difficulty on candidate keywords]
        [loads the seo-keyword-research skill]
        [invokes the skill with assembled inputs]
        [writes output to the partner's cycle working directory]

Claude: Done — generated 52 keywords across 6 clusters. 4 keywords
        are in striking distance. Want me to show the top-priority
        keywords or the full list?
```

The skill invocation is not something you type by command — you describe what you want, and Claude figures out which skill to load and what data to fetch. The skill's `SKILL.md` description is what tells Claude when it's the right skill for the job.

### Skill chaining

Skills are designed to chain cleanly. The output of `seo-keyword-research` is a direct input to `seo-strategy-light`. The output of `seo-strategy-light` feeds into `seo-technical-package`. The output of `seo-technical-package` feeds into `seo-body-content`.

In a typical conversation, you chain them explicitly:

```
You: Now run strategy using those keywords.

Claude: [reads the keywords.json from the partner's cycle directory]
        [loads the seo-strategy-light skill]
        [invokes the skill]
        [writes strategy.json to the cycle directory]
```

Each skill stays data-source-agnostic — it takes structured input objects and returns structured output. Claude Code handles the filesystem and data-fetching layer between skills.

### Common commands / conversation starters

| What you want | How to ask |
|---|---|
| Start a cycle for a partner | "Run keyword research for [Partner Name]" |
| Generate a page | "Generate the technical package for the [page name] page in [Partner]'s current cycle" |
| Draft outreach | "Draft outreach to [prospect domain/name] for [Partner], angle: broken link / resource page / guest post" |
| Check partner health | "How is [Partner Name] trending this quarter?" |
| Diagnose an issue | "Why did [Partner Name]'s [keyword] drop [positions]?" |
| Cycle close | "Generate a cycle-close summary for [Partner Name] covering cycle [N]" |
| Pipeline status | "Which partners are overdue for cycle close?" |

### Data freshness

Skills that use live data (DataForSEO, GSC) can be invoked with either cached or fresh data. Default is cached for speed and cost. For critical analysis or partner-facing deliverables, specify fresh:

```
You: Generate the cycle close summary for Mile High Roofing — use fresh GSC data.

Claude: [pulls fresh GSC data before invoking the skill]
```

---

## Data and file conventions

### Partner profiles — Airtable

Airtable is the source of truth for partner profile data. Skills read this at invocation time and assemble the `partner_profile` input object from Airtable fields.

**Required Airtable fields** (mapped to skill inputs):

| Airtable field | Skill input |
|---|---|
| Business Name | `business_name` |
| Business Type | `business_type` |
| Services | `services` (array) |
| Service Areas | `service_areas` (array) |
| Primary Location (City, State) | `primary_location.locality`, `primary_location.region` |
| Partner Type | `partner_type` (hyper_local / multi_city_local / regional) |
| Unique Differentiators | `unique_differentiators` (array of strings) |
| Brand Voice Notes | `brand_voice_notes` |
| Notable Credentials | `notable_credentials` (array) |
| Preferred CTAs | `preferred_CTAs` (array) |
| Things to Avoid | `things_to_avoid` (array) |
| Years in Business | `years_in_business` |
| Website URL | `website_url` |
| Competitor Domains | `competitor_domains` (array, used for keyword research) |

If Airtable field names don't match exactly, Claude Code handles the mapping at invocation — no separate config file needed.

**If a partner's profile is thin** (missing differentiators or credentials), skill outputs will be more generic. This is flagged in the skill's output rather than masked with fabricated content. The fix is to enrich the Airtable record.

### Cycle outputs — local working directory

Per-partner cycle outputs live in a local working directory outside the repo:

```
~/harbinger-seo-data/           # Local, NOT in repo, NOT in Airtable (for MVP)
  partners/
    mile-high-roofing/
      cycles/
        cycle-02-2026-Q4/
          keywords.json         # seo-keyword-research output
          strategy.json         # seo-strategy-light output
          pages/
            roof-repair-denver/
              technical.json    # seo-technical-package output
              body.md           # seo-body-content output
          outreach/
            prospect-denverhomeliving.json
          notes.md
```

These files accumulate as the cycle runs. Claude Code reads them when needed for chained skill invocations or reporting questions. Git-ignore the entire partner-data directory — it's working state, not source.

### Conventions

- **Partner directory names**: lowercase, hyphen-separated, match the Airtable `Business Name` converted to slug form
- **Cycle directory names**: `cycle-NN-YYYY-QQ` format (e.g. `cycle-02-2026-Q4`)
- **Page directory names**: lowercase, hyphen-separated, match the page's URL slug
- **File names**: lowercase, hyphens, descriptive — `keywords.json` not `KW.json`, `technical.json` not `tech_pkg.json`

---

## Integrations

The repo assumes these integrations are configured in Claude Code desktop:

### Required
- **Airtable** — via MCP server. Source of truth for partner profiles.
- **DataForSEO** — via MCP server. Keyword volume, difficulty, SERP data, and backlink pulls.

### Recommended
- **GitHub** — for repo interaction (skills updates, this README, etc.)
- **Gmail** — for outreach sending (manual in MVP, automation possible later)
- **Google Drive** — for sharing reports with MD/MC

### CMS connectors (as applicable per partner)
- **WordPress.com** — for partners on WordPress
- **Webflow** — for partners on Webflow

### Not yet integrated
- **Google Search Console API** — currently relying on manual CSV exports for GSC data. Can be automated later if it becomes a bottleneck.

---

## Working on the skills

### Modifying a skill

Skills are files in this repo. To modify a skill:

1. Edit the relevant file (`SKILL.md`, or files in `rules/` or `templates/`) on a branch
2. Test the change against the skill's `examples/` — output should still match the worked example within expected tolerance
3. Open a PR for review
4. After merge, re-pull the repo in Claude Code desktop to get the updated skill

Minor tweaks (a clarifying sentence, a new anti-pattern) usually don't need a PR. Larger changes (new output fields, new execution steps, new rules files) should go through review.

### Adding a new skill

If a new pipeline capability warrants a new skill (e.g. `seo-body-content` wasn't in the original five and was added later), follow the convention of the existing skills:

1. Create `/skills/<skill-name>/` with `SKILL.md` at the root
2. Add `rules/` subdirectory for authoritative reference files
3. Add `examples/` subdirectory with at least one worked example
4. Document the skill in this README's skill table
5. Update the workflow section if the new skill changes the cycle flow

The structure is convention, not enforced — but consistency makes the skills easier to reason about.

### Testing a skill

Before relying on a new or modified skill for real partners:

1. Run it in Claude.ai (desktop or web) using the `examples/` inputs — output should closely match the example output
2. Run it against a synthetic partner profile — verify behavior on edge cases
3. Run it against one real partner as a dry run — don't ship content until the output reviews cleanly
4. Document any findings in the skill's `rules/` or `SKILL.md`

---

## The deferred tool

There's a build spec for a skill runner tool (a React + Node app that wraps skill invocations in a web UI for non-technical operators) archived at `docs/archive/skill_runner_tool_build_spec.md`. It was superseded by the decision to use Claude Code desktop as the primary interface.

Revisit this if:
- Multi-user self-serve becomes a real need (MDs running skills themselves)
- Claude Code desktop stops fitting the workflow
- The pipeline scales past 100 partners and needs dashboard-style overviews

Until then, Claude Code desktop handles everything the tool was spec'd for.

---

## Glossary

- **Cycle** — a 6-month partner engagement period. A partner's work happens in sequential cycles; each starts with strategy and ends with a close summary.
- **MD/MC** — Managing Director / Marketing Coordinator, the Harbinger team members who own the partner relationship and review pipeline outputs before delivery.
- **Migration** — the transition of a partner from the external vendor (eBrandz) to the in-house pipeline.
- **Striking distance** — a keyword currently ranking in GSC positions 11-30; the highest-ROI optimization targets.
- **Consolidation / redirect** — a page action where a thin ranking page is absorbed into a more substantive new page via 301 redirect.
- **Partner type** — categorization that shapes strategy decisions: `hyper_local` (single city), `multi_city_local` (2-10 cities), `regional` (10+ cities or multi-state).
- **The technical wrapper** — shorthand for the output of `seo-technical-package`: title, meta, canonical, schema, outline, internal links. The body copy is generated separately by `seo-body-content`.

---

## Questions and contacts

**Project lead**: Ian Matthews, Senior Marketing Coordinator
**Primary operator**: Blaide (Digital department, AI Ops)
**Strategic oversight**: Jack and Benjamin (biweekly progress updates)

For workflow questions: refer to `docs/mvp_playbook.md`
For a specific skill's behavior: refer to the skill's own `SKILL.md`
For changes to the pipeline: propose via PR against this repo
