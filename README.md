# AutoRFP.ai — MCP Server

The Model Context Protocol (MCP) server that lets AI assistants like Claude and ChatGPT connect directly to your AutoRFP.ai workspace. Ask questions about your RFP projects, search your approved content library, and check requirement status without leaving the chat.

This repo holds the public `server.json` manifest. The server itself is hosted by AutoRFP.ai.

## What it does

Once connected, your AI assistant has **read-only** access to:

| What it can access | What it can do |
| --- | --- |
| **Tags** | See your organisation's tag categories and vocabulary |
| **Projects** | List projects, filter by status, due date, or tags |
| **Requirements** | Read questions and answers within a project, filter by status or tag |
| **Content Library** | Search approved content by keyword or semantic meaning, browse by tag or file type |

The connection cannot create, edit, or delete anything in AutoRFP.ai.

## Servers

AutoRFP.ai runs three regional MCP endpoints. Pick the one that matches your AutoRFP.ai data residency region:

| Region | URL |
| --- | --- |
| APAC | `https://api.autorfp.ai/mcp` |
| EU | `https://api.eu.autorfp.ai/mcp` |
| US | `https://api.us.autorfp.ai/mcp` |

If you do not know your region, check the URL you log in with, or ask your AutoRFP.ai admin.

## Setup

There are two parts: an org-level admin sets up the connection once, then each user authorises it for their own chat.

### Part 1: Add the connector (Claude org admin, one-off)

1. Log in to Claude and open **Settings**.
2. Go to **Connectors**, then **Add**, then **Add Custom Connector**.
3. Enter the AutoRFP.ai MCP server URL for your region (see table above).
4. Name the integration (e.g. "AutoRFP.ai") and click **Add**.

### Part 2: Authorise the connection (each user)

1. In Claude, go to **Settings**, then **Connectors**.
2. Find the AutoRFP.ai connector and click **Connect**.
3. Sign in to AutoRFP.ai when prompted and approve the three permission scopes:
   - `tags:read` — see your organisation's tag categories
   - `projects:read` — read your RFP projects and requirements
   - `content:read` — search your approved content library
4. Make sure the new connection is **Enabled** in your next conversation.

Each user inherits their own AutoRFP.ai permissions, so people only see what they would see logged into the app directly.

### ChatGPT and other MCP clients

ChatGPT and any other MCP-compatible client uses the same server URLs and OAuth flow. Add a custom connector or integration in your client's settings and point it at the URL for your region.

## Example prompts

Once connected, try:

- *"List my open RFP projects in AutoRFP.ai."*
- *"What RFPs are due in the next two weeks and how many unanswered requirements do they each have?"*
- *"Search our content library for responses about GDPR compliance."*
- *"List all projects tagged 'Healthcare' and 'Security' that are still in progress."*
- *"Find everything we've ever said about our implementation methodology and summarise the key themes."*
- *"Which content items have been reused the most across projects?"*

### Tips

- **Be specific.** "In-progress projects tagged Security" returns sharper results than "show me projects."
- **Combine filters.** Status, tags, and due dates all stack.
- **Use semantic search.** The content library understands meaning, not just keywords.

## Requirements

- An **AutoRFP.ai** account with access to the projects and content you want to query.
- A **Claude Pro, Team, or Enterprise** plan (the Integrations and Connectors option is plan-gated). Equivalent paid tier for other MCP clients.
- For the initial org-level setup in Claude, a **Claude organisation admin**.

## Troubleshooting

**"I don't see the Connectors option in Settings."** You need a Claude Pro, Team, or Enterprise plan.

**"A colleague added the integration but I still can't see my data."** Each user connects their own AutoRFP.ai credentials. Run Part 2 of the setup above.

**"I'm getting permission errors on certain projects."** The connector inherits your AutoRFP.ai role-based permissions. If you cannot see a project in the app, the assistant cannot either.

**"Which region am I on?"** Look at the URL you use to log into AutoRFP.ai, or ask your AutoRFP.ai admin.

## More resources

- [How to connect to AI Assistants (MCP Server)](https://learn.autorfp.ai/en/articles/15029444-how-to-connect-to-ai-assistants-mcp-server)
- [How to Integrate with Claude](https://learn.autorfp.ai/en/articles/15031130-how-to-integrate-with-claude)
- [AutoRFP.ai Learning Centre](https://learn.autorfp.ai/en)
- Live chat support is available in-app.

## License

MIT
