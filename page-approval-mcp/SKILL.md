---
name: page-approval-mcp
description: Query Confluence page approval statuses, review decision comments, list personal pending sign-offs, and inspect space-level approval bottlenecks via Atlassian Rovo MCP.
---

# Gears Content Approval MCP Skill

Use this skill when connecting an MCP-compatible AI client (Claude Code, Claude Desktop, Cursor, Codex, or custom Rovo Studio agents) to the **Page Approval MCP** server (`approval-workflow-mcp`).

## Connection Setup

1. Install **Gears Content Approval** on your Confluence Cloud site and ensure **Atlassian Rovo** is enabled.
> **Consent first:** Before calling tools from an external client, the same user opens the app once in Confluence, selects **Allow access**, and runs one prompt with **Approval Assistant** in Rovo Chat. Forge needs this app consent entry for remote `asUser()` calls; without it, tool calls can fail.

2. In **Atlassian Administration → Apps → Sites → [Your Site] → Connected apps**, open **Gears Content Approval** and enable external MCP exposure for its four read-only tools:
   - `get-page-approval`
   - `get-approval-comments`
   - `list-my-pending-approvals`
   - `get-approval-overview`
3. Connect your MCP client to:
   - Codegeist 2026 Judge Edition endpoint: `https://mcp.atlassian.com/forge/88454548-147f-4b32-bd16-2c8a3afae65e`

> **Safety by Design:** Only read-only (`GET`) actions are exposed over MCP. Write/trigger actions (`request-page-approval`, `submit-approval-decision`, `nudge-page-approvers`) remain gated inside Confluence Rovo Chat with interactive confirmation cards.

---

## Standard Operating Procedures (SOPs)

### 1. Daily Personal Sign-Off Triage
1. Call `list-my-pending-approvals` to retrieve all Confluence pages currently waiting on the authenticated user's decision.
2. Sort items by overdue status and earliest due date first.
3. For any urgent or overdue page, call `get-page-approval` and `get-approval-comments` (passing the page ID/URL/title explicitly) so the user can review the policy (`ALL`, `ANY`, `QUORUM`), fellow approvers' decisions, and prior review notes before opening the page to sign off.

### 2. Space Governance & SLA Bottleneck Review (Admins)
1. Call `get-approval-overview` (optionally scoped to `spaceKey`) to inspect turnaround metrics, overdue pending pages, and workload distribution across approvers.
2. Identify bottleneck pages (longest pending or past due date) and call `get-page-approval` on the top blockers to report exactly which pending approvers are holding up completion.

### 3. Audit Trail & Rejection Post-Mortem
1. Given a Confluence page ID, URL, or title, call `get-page-approval` and `get-approval-comments`.
2. Summarize the approval rule, requester, timestamps (already formatted in the caller's time zone), and quote each approver's decision comment verbatim without inventing links or status codes.
