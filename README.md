# Gears Atlassian MCP Skills

Open-source `SKILL.md` playbooks for connecting third-party AI coding and governance agents (Claude Code, Codex, Cursor, Claude Desktop, and Rovo Studio) to the five **Gears Forge** Atlassian Rovo MCP servers built for **Codegeist 2026**.

## Included Skills

| Skill | Product | MCP Server Key | Guide |
|---|---|---|---|
| `gears-diagram-mcp` | Confluence | `gears-diagram-mcp` | [`gears-diagram-mcp/SKILL.md`](./gears-diagram-mcp/SKILL.md) |
| `permission-audit-mcp` | Confluence | `permission-audit-mcp` | [`permission-audit-mcp/SKILL.md`](./permission-audit-mcp/SKILL.md) |
| `page-approval-mcp` | Confluence | `approval-workflow-mcp` | [`page-approval-mcp/SKILL.md`](./page-approval-mcp/SKILL.md) |
| `deliverable-gatekeeper-mcp` | Jira | `deliverable-mcp` | [`deliverable-gatekeeper-mcp/SKILL.md`](./deliverable-gatekeeper-mcp/SKILL.md) |
| `field-history-mcp` | Jira | `field-history-mcp` | [`field-history-mcp/SKILL.md`](./field-history-mcp/SKILL.md) |

## Prerequisites (Atlassian Cloud + Forge `rovo:mcp`)

1. Install the corresponding Forge app on an Atlassian Cloud site with **Atlassian Rovo** enabled.
2. **Consent first:** Before calling tools from an external MCP client, open the app once inside Jira/Confluence, click **Allow access**, and run one prompt with the app's Rovo Agent in Rovo Chat so Forge records your user-level `asUser()` consent.
3. **Enable external MCP exposure:** In **Atlassian Administration → Apps → Sites → [Your Site] → Connected apps**, open the app and enable external MCP exposure for its tools.
4. Connect your MCP client to `https://mcp.atlassian.com/forge/<appId>`, complete OAuth 2.1 consent, and load the corresponding `SKILL.md`.

## License

Licensed under the [MIT License](./LICENSE).
