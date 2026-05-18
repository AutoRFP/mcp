---
name: content-library-contradictions
description: Audits an AutoRFP.ai content library for contradictions — conflicting factual claims across answers, outdated facts that contradict the company's current state, and internal capability contradictions (e.g. "we don't integrate with X" vs "we have X integration"). Use this skill whenever the user mentions auditing their content library, checking for conflicting answers, finding inconsistencies in the library, fact conflicts, or wants to surface where the library disagrees with itself. Also trigger on phrases like "check my library for contradictions", "find conflicting answers", "audit my library", "where does my library contradict itself", "are there conflicting facts in our content", "library health check", or "what's wrong with my content library". This is library-level hygiene — distinct from rfp-contradiction-checker which audits a single completed RFP response, not the underlying content library that feeds every response.
---

# Content Library Contradictions

## Why this skill exists

An AutoRFP.ai content library is the single source of truth that feeds every RFP, DDQ, and security questionnaire response. When two answers in the library disagree with each other, or when an old answer contradicts the company's current state, the response engine can pull the wrong one into a live deal and put incorrect information in front of a buyer. This is one of the highest-impact hygiene problems a content team can fix, and it compounds: every contradiction left in place will be pulled into future deals.

This skill audits the user's library for three classes of contradiction:

1. **Conflicting facts** — two answers state different numbers, names, or claims for the same thing (e.g. one answer lists one supported language count, another lists a different one).
2. **Outdated facts** — an old answer contradicts the company's current state (e.g. an answer says the company doesn't have a capability the company has since launched).
3. **Internal capability contradictions** — two answers describe mutually exclusive capabilities (e.g. one says data is hosted in a single region, another references multi-region deployments).

The skill **flags** contradictions; it does not auto-edit content. The output is an Excel file the user reviews to decide which version of each fact is correct.

## When to trigger

Trigger whenever the user asks about contradictions, inconsistencies, conflicting answers, or library health. Don't confuse with `rfp-contradiction-checker` (which scans a finished RFP response, not the library).

## Inputs you need from the user

Before pulling any data, **ask the user clarifying questions using the AskUserQuestion tool**. Even if the user's request seems clear, the audit is much more useful when scoped. Ask:

1. **Scope** — audit the whole library, or focus on a topic area? Common topic scopes include security, integrations, AI/architecture, company information, pricing, support, implementation. A scoped audit is faster and produces a more actionable list.
2. **Tag prioritisation** — call `list_tags` first to discover the org's actual taxonomy. Then ask: which tag categories matter most for this audit? Use the discovered tags as option labels — don't make up tags that don't exist in the user's library.
3. **Ground-truth anchors** — are there facts the user knows the *current truth* for? Typical anchor categories for a B2B SaaS company are: where customer data is hosted (regional architecture), current team size, AI/LLM model stack, certifications held, integrations supported, current customer roster used as proof, uptime / SLA commitments, pricing model. These anchors become the reference points for detecting outdated facts. If the user can't easily list anchors, derive a working hypothesis from the most recently updated content and flag this assumption clearly in the output.
4. **Time window** — include all content, or only content older than N months? Recent content is less likely to be outdated but can still conflict with peer answers.
5. **Severity threshold** — flag everything, or only contradictions in content that's actively being pulled into deals (usageCount above a threshold)? High-usage contradictions are the actionable subset.

The first question matters most — without scope, you'll surface noise. The other four make the output sharper.

## How the audit works

### Step 1 — Discover the taxonomy

Always start with `list_tags`. This tells you what categories the user's org actually uses. You'll need these to scope `list_content` and `search_content` calls, and to render meaningful "topic" labels in the output. Don't assume the taxonomy — discover it from the user's actual library.

### Step 2 — Establish ground truth

For each topic area in scope, collect 3-5 "anchor facts" the user has confirmed (from the clarifying questions). For each anchor fact, capture:

- The fact itself (a short, falsifiable statement)
- The current correct value as the user describes it
- Common stale phrasings the user can think of (single-region claims when multi-region, older team size figures, prior integrations, deprecated product names)

If the user can't supply anchors, build a working hypothesis from the 50 most recently updated items in scope — these are the most likely "current truth". Mark this clearly in the output as an assumption.

### Step 3 — Pull the content slices

Use `list_content` with the scoping filters from Step 1:
- `tagIds` to scope to the user's selected categories
- `fileType` to focus on Q&A pairs (most contradictions live in answers, not raw documentation)
- `hasAnswer: true` to skip blank entries
- Paginate through all matching items (25/page, watch `pagination.hasMore`)

Capture for each item: `id`, `question`, `answer`, `fileName`, `tagIds`, `updatedAt`, `usageCount`, **and `referenceUrl`**. The `referenceUrl` is mandatory — every flagged row in the Excel output must be clickable so the user can jump straight to the item in AutoRFP.ai. Never write `id` to user-facing output — use `referenceUrl` for links and `question` + `fileName` for human-readable identification.

### Step 4 — Cluster and compare

You won't fit a large library into context. Instead:

- **For ground-truth anchors**, use `search_content` with queries that target each anchor fact (e.g. queries like "data center location", "data residency", "where is customer data hosted" for a hosting anchor). Take the top 10-15 ranked results, check each against the anchor for contradictions.
- **For peer-to-peer conflicts**, group items by tag intersection. Within each group, look for near-duplicate questions that have substantively different answers. The fastest way is to scan questions in the group for similar wording and compare their answers.
- **For capability contradictions**, search for negation patterns ("we don't", "not currently", "not available", "do not support", "no, we don't") and check whether a positive claim exists for the same capability elsewhere.

Don't try to find every contradiction in one pass. Aim for the top ~30-60 highest-confidence flagged contradictions per audit. A short, sharp list is more useful than an exhaustive one the user won't act on.

### Step 5 — Score severity

For each flagged contradiction, capture:

- **Risk level** — Critical (in actively-used content, usageCount >= 10), High (usageCount >= 5), Medium (usageCount >= 1), Verify (usageCount = 0)
- **Recommendation** — which version to keep, or "review both", with one-line reasoning
- **The two (or more) `referenceUrl`s** for the conflicting items

Risk scoring matters because the user has limited time. A contradiction in content used 30 times is dramatically worse than one in content used once.

### Step 6 — Build the Excel

Use the bundled script:

```bash
python scripts/build_audit_xlsx.py --input <findings.json> --output <path/to/file.xlsx>
```

Pass findings as JSON in the structure described in `references/output_format.md`. The script produces four sheets:

1. **Summary** — headline finding, scope, library stats, recommended actions
2. **Conflicting Facts** — pairs of items that disagree on the same factual claim, with both `referenceUrl`s
3. **Outdated Facts** — items that contradict ground-truth current state, with `referenceUrl`s
4. **Capability Contradictions** — items that claim mutually exclusive capabilities, with `referenceUrl`s

Every row must include a clickable URL to each item involved. This is non-negotiable — the audit is only useful if the user can jump straight to each item to make a decision.

### Step 7 — Save and share

Save the output to the user's workspace with a filename like `content_library_contradictions_<YYYY-MM-DD>.xlsx`. Share the file path so the user can open it.

## What to avoid

- **Don't load the whole library**. Large libraries will burn context before getting anywhere useful. Cluster, sample, and search instead.
- **Don't auto-fix**. The user picks the canonical version. Your job is to flag, not to rewrite.
- **Don't surface noise**. A contradiction between two zero-usage items is technically a contradiction but rarely worth a user's time. Prioritise by usageCount.
- **Don't omit URLs**. Every flagged row must include the `referenceUrl`. If you can't link to it, you can't help the user fix it.
- **Don't conflate types**. Outdated-vs-current is a different category than peer-conflict. Keep them on separate sheets so the user can triage differently.

## Output format

See `references/output_format.md` for the exact JSON structure to pass to `scripts/build_audit_xlsx.py`. See `references/methodology.md` for deeper notes on clustering, search query patterns, and how to handle edge cases.
