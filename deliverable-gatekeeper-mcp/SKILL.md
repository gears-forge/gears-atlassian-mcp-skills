---
name: deliverable-gatekeeper-mcp
description: Audit Jira engineering deliverable review statuses and scan projects for unapproved or rejected release-gate attachments via Atlassian Rovo MCP.
---

# Deliverable Gatekeeper MCP Skill

Use this skill when connecting an MCP-compatible AI client (Claude Code, Claude Desktop, Cursor, Codex, or custom Rovo Studio agents) to the **Deliverable Gatekeeper MCP** server (`deliverable-mcp`) provided by **Attachment Manager & Folders for Jira**.

## Connection Setup

1. Install **Attachment Manager & Folders for Jira** (`v2.2.0+` or the Codegeist Judge Edition) on your Jira Cloud site and ensure **Atlassian Rovo** is enabled.
> **Consent first:** Before calling tools from an external client, the same user opens the app once in Jira, selects **Allow access**, and runs one prompt with **Deliverable Quality Gatekeeper** in Rovo Chat. Forge needs this app consent entry for remote `asUser()` calls; without it, tool calls can fail.

2. In **Atlassian Administration → Apps → Sites → [Your Site] → Connected apps**, open the app and enable external MCP exposure for:
   - `check-issue-deliverable-readiness`
   - `scan-project-pending-deliverables`
3. Connect your MCP client to:
   - Commercial Marketplace app endpoint: `https://mcp.atlassian.com/forge/2deded4c-e285-4b40-b57a-d4e46190de04`
   - Codegeist 2026 Judge Edition endpoint: `https://mcp.atlassian.com/forge/7a3d024a-c457-462c-8391-e1826e87f0ed`

> **Zero File Egress:** Tools return attachment review metadata only (`approved`, `pending`, `changes_requested`, `not_requested`) using the caller's `asUser` Jira permissions. Raw binary file contents are never transmitted.

---

## Standard Operating Procedures (SOPs)

### 1. Single-Issue Release Gate Sign-Off (`APQP` / `ISO 26262` / Hardware & Firmware Releases)
1. Call `check-issue-deliverable-readiness(issueKey)` with an explicit Jira issue key (e.g., `"CAR-12"`).
2. State the release verdict clearly: an issue is ready to ship **only** when `isReadyToShip === true` (every attachment is `approved`).
3. Group non-approved files by status (`changes_requested`, `pending`, `not_requested`), quote each `reviewerComment` verbatim, and include the returned `nextStep` and `reviewRules` (including whether independent review prevents the submitter from self-approving).

### 2. Project-Wide Milestone Readiness Sweep
1. Call `scan-project-pending-deliverables(projectKey)` with an explicit Jira project key (e.g., `"CAR"`).
2. List every blocked issue with its clickable `url`, assignee `mention`, and blocking filenames + reviewer feedback.
3. If `truncated === true`, state explicitly that the scan inspected the 50 most recently updated issues with attachments and direct the user to **Project Sidebar → Attachment Center** for the full project inventory.
