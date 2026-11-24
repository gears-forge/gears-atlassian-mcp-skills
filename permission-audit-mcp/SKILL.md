---
name: permission-audit-mcp
description: Audit Confluence space permissions, inspect 6 security risk categories, and evaluate employee offboarding or role-change access blast radius via Atlassian Rovo MCP.
---

# Confluence Space Permission Audit MCP Skill

Use this skill when connecting an MCP-compatible AI client (Claude Code, Claude Desktop, Cursor, Codex, or custom Rovo Studio agents) to the **Permission Audit MCP** server (`permission-audit-mcp`).

## Connection Setup

1. A Confluence/site administrator installs **Space Permission Audit for Confluence**, opens it from **Apps**, and selects **Refresh index**. Wait for the index to finish and check its coverage.
> **Consent first:** Before calling tools from an external client, the same user opens the app once in Confluence, selects **Allow access**, and runs one prompt with **Permission Audit Copilot** in Rovo Chat. Forge needs this app consent entry for remote `asUser()` calls; without it, tool calls can fail.

2. In **Atlassian Administration → Apps → Sites → [Your Site] → Connected apps**, a site or organization administrator opens the app and enables external MCP exposure for the desired tools:
   - `get-security-insights-summary`
   - `inspect-principal-blast-radius`
   - `check-space-permission-summary`
3. Connect a compatible remote MCP client to `https://mcp.atlassian.com/forge/<appId>` (replace `<appId>` with the installed Forge app UUID). Complete OAuth 2.1 consent and select the site on which exposure was enabled. Atlassian lists Rovo MCP and external connections as Preview; see its [Rovo MCP documentation](https://developer.atlassian.com/platform/forge/manifest-reference/modules/rovo-mcp/).

> **Authorization Boundary:** These tools require Confluence/site administrator authorization (`requireLicensedAdmin`). A separately authorized space administrator page is scoped to the current space and does not confer permission to use the site-wide tools.

> **Evidence Boundary:** Results use indexed space grants and indexed group memberships. They do not account for page restrictions or prove complete live access. Check timestamps and coverage, and refresh the index after group membership changes. Report response limits and truncation instead of treating a partial list as complete.

---

## Standard Security & Governance SOPs

### 1. Daily / Weekly Confluence Posture Check
1. Call `get-security-insights-summary` first.
2. If `outcome` is `"not-indexed"`, instruct the administrator to open **Apps → Space Permission Audit for Confluence** and select **Refresh index** before proceeding.
3. Group findings by severity:
   - **Critical:** `no_admin` (spaces with no indexed administrator grant) and `over_granted` (site-wide groups or access classes holding space `administer` or `export` rights).
   - **Warning:** `stale_grant` (deactivated/former users retaining grants) and `empty_group` (grants assigned to 0-member groups).
   - **Info / Exposure:** `anonymous` (publicly readable spaces) and `island` (single-user spaces at risk of knowledge loss).
4. Include `indexedAt`, `groupsIndexedAt`, index status, and any available coverage warnings. A timestamp identifies an observation; it does not make every underlying record a simultaneous live snapshot.

### 2. Employee Offboarding / Role-Transfer Blast-Radius Audit
1. Call `inspect-principal-blast-radius` with `principalQuery` set to the employee's email, display name, or Atlassian `accountId` (or a group name when auditing a group).
2. Handle disambiguation safely:
   - If `outcome` is `"not-found"`, report clearly that no user/group matched—**never** claim the user has access to 0 spaces.
   - If `outcome` is `"ambiguous"`, present the returned `candidates` (display name, available email, `principalId`) and ask the administrator to confirm the intended `principalId` before continuing. At most 10 candidates are returned.
3. For a resolved user, review `spaces` (`effective` mode combines direct user grants + inherited group grants + site-wide access classes):
   - Flag spaces where the user has `administer` rights; inspect other administrator grant principals with `check-space-permission-summary(spaceKey)`. `adminCount` counts indexed grant principals, including groups, and does not prove how many active people can administer the space.
   - Identify direct and inherited sources where available, and verify group membership and remaining access before making any separate change.
   - The action returns at most 50 spaces; if its response contains a truncation note, direct the administrator to the app for the full indexed list.

### 3. Pre-Audit Space Sign-Off (`ISO 27001` / `SOC 2`)
1. Given one or more sensitive space keys (e.g., `HR`, `FIN`, `SEC`, `LEGAL`), call `check-space-permission-summary` for each exact `spaceKey`.
2. Review the returned evidence against the organization's policy:
   - `space.anonymousReadAccess` is `"enabled"` or `"disabled"` in the indexed data.
   - `space.adminCount` is the count of administrator grant principals. Independently verify active individuals and group membership before assessing administrator coverage.
   - Examine each returned user grant's `accountStatus` where available; a missing status is unknown, not evidence of inactivity or activity.
   - Inspect whether site-wide access classes have unintended write/admin/export permissions.
3. Review the response's truncation note: at most 60 grants are returned. If more exist, continue in the app. State that these checks support review preparation; they do not establish regulatory compliance.
