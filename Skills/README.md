# AutoRFP.ai Skills

Skills for Claude that work against the AutoRFP.ai content library MCP. Drop these into a Claude Code plugin marketplace, a Cowork plugin, or wherever your team consumes skills from. Designed to work with any AutoRFP.ai customer instance — point at your MCP and go.

## Skills

### `content-library-contradictions`

Audits your AutoRFP.ai content library for three classes of contradiction:

- **Conflicting facts** — two answers state different values for the same thing (e.g. one answer lists one supported language count, another lists a different one)
- **Outdated facts** — old answers contradict the company's current state (e.g. an answer references a single-region hosting footprint when the company now has multi-region deployments)
- **Capability contradictions** — mutually exclusive claims (e.g. one answer says the company doesn't integrate with a platform, another confirms the integration is live)

Output is an Excel file with one sheet per contradiction type. Every flagged row includes a clickable URL straight to the item in AutoRFP.ai. The skill flags only — it never auto-edits content.

**When it triggers**: any time the user mentions auditing the library, checking for conflicting answers, finding inconsistencies, or surfacing contradictions. Not to be confused with `rfp-contradiction-checker`, which audits a finished RFP response rather than the library itself.

**What it needs**: the AutoRFP.ai MCP for your instance (provides `list_tags`, `list_content`, `search_content`, `list_content_usage`, `list_projects`, `get_project`).

## Structure

```
skills/
└── content-library-contradictions/
    ├── SKILL.md                          # Main skill instructions
    ├── scripts/
    │   └── build_audit_xlsx.py           # Builds the Excel output
    └── references/
        ├── methodology.md                # Deep-dive on audit strategy
        └── output_format.md              # JSON schema for the build script
```

## Installation

The exact install path depends on your environment.

- **Claude Code plugin marketplace**: place this `skills/` folder at the root of your marketplace repo.
- **Cowork**: copy each skill folder into your Cowork plugins directory.
- **Claude.ai**: package each skill as a `.skill` file using the skill-creator's `package_skill.py` and upload.

Whichever path you use, make sure the AutoRFP.ai MCP for your instance is connected before running the skill.
