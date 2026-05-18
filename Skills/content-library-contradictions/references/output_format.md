# Output Format Spec

The Excel build script (`scripts/build_audit_xlsx.py`) expects a single JSON file in the following shape. Every contradiction must include the `reference_url` of every item involved — this is non-negotiable.

```json
{
  "scope": "Whole library",
  "library_size": 8500,
  "items_audited": 3200,
  "ground_truth_anchors": [
    {
      "fact": "Regional deployments",
      "current_truth": "Multi-region (per the user's stated anchor)",
      "source": "user-supplied"
    },
    {
      "fact": "Languages supported",
      "current_truth": "<current number from the user>",
      "source": "user-supplied"
    }
  ],
  "conflicting_facts": [
    {
      "topic": "Languages supported",
      "item_a": {
        "question": "<the question text from the older item>",
        "answer_excerpt": "<first ~250 chars of the answer>",
        "file": "<fileName as returned by list_content>",
        "date": "<updatedAt as YYYY-MM-DD>",
        "usage": 25,
        "reference_url": "<referenceUrl returned by the MCP — required>"
      },
      "item_b": {
        "question": "<the question text from the newer / conflicting item>",
        "answer_excerpt": "<first ~250 chars of the answer>",
        "file": "<fileName>",
        "date": "<YYYY-MM-DD>",
        "usage": 0,
        "reference_url": "<referenceUrl — required>"
      },
      "conflict": "<one-sentence description of the substantive disagreement>",
      "recommendation": "<which version to keep and why; one short sentence>",
      "risk": "Critical"
    }
  ],
  "outdated_facts": [
    {
      "topic": "Data residency",
      "item": {
        "question": "<the question text>",
        "answer_excerpt": "<first ~250 chars>",
        "file": "<fileName>",
        "date": "<YYYY-MM-DD>",
        "usage": 17,
        "reference_url": "<referenceUrl — required>"
      },
      "anchor_fact": "<the user's ground-truth anchor this contradicts>",
      "stale_claim": "<the specific stale phrasing in the answer>",
      "recommendation": "<update to match anchor, archive, or review>",
      "risk": "Critical"
    }
  ],
  "capability_contradictions": [
    {
      "capability": "<the capability being disputed, e.g. an integration name>",
      "negative_item": {
        "question": "<question text from the answer that denies the capability>",
        "answer_excerpt": "<first ~250 chars>",
        "file": "<fileName>",
        "date": "<YYYY-MM-DD>",
        "usage": 46,
        "reference_url": "<referenceUrl — required>"
      },
      "positive_item": {
        "question": "<question text from the answer that confirms the capability>",
        "answer_excerpt": "<first ~250 chars>",
        "file": "<fileName>",
        "date": "<YYYY-MM-DD>",
        "usage": 1,
        "reference_url": "<referenceUrl — required>"
      },
      "recommendation": "<which version to keep; usually archive the negative if the capability now exists>",
      "risk": "Critical"
    }
  ]
}
```

## Required fields per row type

**Every item reference (inside `conflicting_facts.item_a`, `conflicting_facts.item_b`, `outdated_facts.item`, `capability_contradictions.negative_item`, `capability_contradictions.positive_item`) must include:**

- `question` (string) — the human-readable question text
- `file` (string) — the fileName returned by `list_content`
- `date` (string, YYYY-MM-DD) — the item's `updatedAt`
- `usage` (integer) — the item's `usageCount`
- `reference_url` (string) — the `referenceUrl` returned by the MCP. **Mandatory.** Never substitute the `id` field.

Optional:

- `answer_excerpt` (string) — first 200-300 chars of the answer, helpful context for the reviewer

**Every contradiction row must include:**

- `risk` — one of "Critical" / "High" / "Medium" / "Verify"
- `recommendation` — short, actionable next step

## Filename convention for output

`content_library_contradictions_<YYYY-MM-DD>_<scope>.xlsx`

Example: `content_library_contradictions_2026-05-18_security.xlsx`
