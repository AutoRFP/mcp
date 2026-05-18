# Audit Methodology

This file goes deeper than SKILL.md on how to actually run the audit well. Read it when SKILL.md tells you to, or any time the audit produces noisy or low-confidence results.

## The three failure modes to avoid

1. **Loading too much**. A mature AutoRFP.ai library can hold thousands or tens of thousands of items. You will run out of context before you find anything if you try to grep everything. Use `list_tags`, `list_content` with tight filters, and `search_content` instead.
2. **Finding "contradictions" that aren't**. Two answers can say things in different ways without contradicting. Reserve the contradiction flag for substantive factual conflict (different numbers, different yes/no claims, different lists of supported items). Stylistic variation is not contradiction.
3. **Burying the actionable items**. If you flag 200 contradictions of equal weight, the user does nothing. Sort by usageCount and present the top 30-60.

## Discovering the taxonomy

Run `list_tags` first. It returns categories grouped by parent. Typical AutoRFP.ai orgs configure categories such as:

- **Requirement Type** with sub-tags like Features, Integrations, Security, Privacy, Pricing, Legal, Implementation, Support, Account Management, Company
- **Questionnaire Type** with values like RFP, DDQ, RFI, SQ (security questionnaire), Marketing Award, Grant
- **Region** with values like APAC, AMER, EMEA, Global

Every org's taxonomy is different. Some orgs have product-line categories, vertical categories, or custom tags for asset classes / business units. Use what `list_tags` returns — don't assume the structure.

When you ask the user about scope, use the discovered tag labels as option labels in the AskUserQuestion call. This is what makes the audit usable. Don't ask "which tags?" abstractly — present the real ones.

## Establishing ground truth

The strongest contradiction signal comes from comparing library content against the *current truth*. Without ground truth, you can find peer-conflicts but not outdated facts.

Good anchor facts are short, falsifiable, and concrete. Anchors should describe what is true *today* in clear language.

Examples of strong anchor patterns:

- Current regional deployment footprint (single region vs multi-region, specific regions named)
- Current team size (a number)
- Current AI/LLM stack (specific models or vendors)
- Current certifications held (and certification version, e.g. ISO 27001:2022)
- Current list of supported integrations
- Current uptime / SLA commitment
- Current pricing model (per-user, per-project, tiered)
- Current customer roster used as social proof (named accounts)

Bad anchors:

- "We're a leader in the space" — not falsifiable
- "Our product is the best" — not factual
- "We use AI" — too vague to detect contradictions against

When asking the user for anchors, prompt them across these categories: data residency / regional architecture, team size, AI/LLM stack, certifications, integrations supported, customer proof points, response time / uptime, pricing model. These are the buckets where contradictions cause the most damage in real deals.

## Search query patterns for each contradiction type

### Conflicting facts (peer-to-peer)

Use `search_content` with the topic phrasing prospects actually use in questionnaires. Compare top 10-15 results pairwise.

Examples of useful search queries:

- "languages supported" — find every answer that lists a language count
- "datacenter location" / "data residency" / "where is data hosted"
- "pricing model" / "subscription pricing" / "per-user pricing"
- "uptime SLA" / "availability commitment"
- "team size" / "company size" / "number of employees"
- "supported integrations" / "what integrations" / "third-party connectors"

For each topic, expect to see 5-20 answers. Where two or more of them disagree substantively, you have a peer-conflict.

### Outdated facts (item-vs-anchor)

For each ground-truth anchor, run `search_content` with 2-3 paraphrasings of the topic, then check every returned answer against the anchor. Flag any that contradicts.

Example workflow for an anchor about regional architecture:
1. search "where is customer data hosted"
2. search "datacenter region"
3. search "data sovereignty"

For each result, look for phrases that contradict the current anchor — single-region claims when the company is multi-region, references to deprecated regions, references to old data centers. These are the signals of an outdated answer.

### Capability contradictions (positive vs negative)

These are the most common and the easiest to miss. Search for negation phrases first:

- "we don't currently"
- "not natively"
- "not currently available"
- "we do not support"
- "no, we don't"
- "we are exploring" / "actively exploring"
- "on the roadmap"
- "in development"
- "coming soon"

For each negative claim, identify the capability being denied, then search for a positive version of the same claim. If both exist, you have a contradiction (or at minimum, a stale negative claim where the capability has since been added).

A common pattern in real libraries: an older answer says "we are actively exploring integration with [Platform X]" — but a newer answer or the company's current product positioning lists [Platform X] as a supported integration. That negative answer is now wrong and will get pulled into deals if not archived or updated.

## Risk scoring

Apply this exactly so the user can scan the Excel quickly:

| Risk | Criteria |
|---|---|
| Critical | usageCount >= 10 — being pulled into deals heavily |
| High | usageCount 5-9 — being pulled into deals regularly |
| Medium | usageCount 1-4 — has been pulled into deals |
| Verify | usageCount 0 — never used, but still in the library |

When there are two items in a contradiction (peer-conflict or capability), use the higher usageCount of the two.

## Handling edge cases

**Same answer in multiple file sources.** A library often has the same Q&A imported from different RFP responses or document sources. These are duplicates, not contradictions. Recognise them by similar answers and skip — but flag the duplication separately if scope allows.

**Templated customer names.** Some answers contain `{{customer}}` placeholders or hard-coded customer names. A hard-coded customer name in an answer that's reused across customers is a quality bug worth flagging as a capability contradiction (the content "claims" to be customer-specific when it isn't).

**Time-relative language.** Phrases like "currently underway", "in progress", "recently completed", "next quarter" go stale immediately. Flag any item containing these patterns regardless of contradiction status — they will become outdated by definition.

**Documentation vs Q&A.** Documentation items (`fileType` = "documentation") usually don't contain conflicting claims — they're reference chunks. Focus the audit on Q&A and snippets. Skip documentation unless explicitly in scope.

**Confidential project membership.** `list_content_usage` filters out usages from confidential projects the requesting user isn't a member of. The actual usage count may be slightly higher than what you can see. This is a known limitation worth noting if usage data drives the risk score.

## When the audit returns very few results

If you find fewer than 10 flagged items, do not pad. Either:

1. The library is well-maintained (good outcome, report this honestly), or
2. The scope is too narrow (expand it with the user), or
3. The ground-truth anchors are weak (re-ask the user for sharper anchors)

Don't manufacture contradictions to fill space. A clean audit is more valuable than a padded one.

## When the audit returns too many results

Cap at 60 items across all sheets. If you have more, present the top 60 by risk score (Critical first, then High, then by usageCount). Note in the Summary that more were found and offer to run a follow-up audit on a narrower scope.

## URL requirement

Every single flagged row must contain `referenceUrl` values for every item involved. If a row's URL is missing, the audit is broken — the user can't fix what they can't reach. The `referenceUrl` is returned by both `list_content` and `search_content`. Never substitute the `id` field for a URL in user-facing output.
