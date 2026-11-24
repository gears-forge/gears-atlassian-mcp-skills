---
name: field-history-mcp
description: Detect Jira scope creep, compare live Jira issues against saved baseline snapshots, and inspect cross-product Teamwork Graph impact across Goals, Teams, PRs, Deployments, and Designs via Atlassian Rovo MCP.
---

# Field History, Release Baseline & Teamwork Graph Radar MCP Skill

Use this skill when connecting an MCP-compatible AI client (Claude Code, Claude Desktop, Cursor, Codex, or custom Rovo Studio agents) to the **Field History MCP** server (`field-history-mcp`).

## Connection Setup

1. Install **Field History for Jira (Judge Edition)** on your Jira Cloud site and ensure **Atlassian Rovo** is enabled.
> **Consent first:** Before calling tools from an external client, the same user opens the app once in Jira, selects **Allow access**, and runs one prompt with **Release Baseline Copilot** in Rovo Chat. Forge needs this app consent entry for remote `asUser()` calls; without it, tool calls can fail.

2. In **Atlassian Administration → Apps → Sites → [Your Site] → Connected apps**, open **Field History** and enable external MCP exposure for:
   - `analyze-issue-field-churn`
   - `compare-issue-with-baseline`
   - `inspect-issue-graph-impact`
3. Connect your MCP client to `https://mcp.atlassian.com/forge/<appId>` (replace `<appId>` with the installed Forge app UUID).

> **Permission & Privacy Boundary:** All three tools verify app access and Jira issue visibility as the authenticated caller (`asUser`) before reading snapshot storage or calling `requestTeamworkGraph`. Every Teamwork Graph query runs as `asUser`, enforcing object-level ACLs across Goals, Teams, Pull Requests, Deployments, and Designs. Comment bodies are never exposed—only comment counts.

---

## Standard Operating Procedures (SOPs)

### 1. Cross-Product Teamwork Graph Blast-Radius & Release Gate Audit
1. Always pass an explicit `issueKey` (e.g., `"PROJ-123"`), since external MCP clients do not carry Jira browser page context.
2. Call `inspect-issue-graph-impact(issueKey, snapshotName?)` before signing off on a release candidate or sprint scope change.
3. Review `overallRiskPosture` and `impactSignals`:
   - **`POST_DEPLOYMENT_SPEC_DRIFT` (`critical` / `warning`)**: Scope-sensitive Jira fields changed *after* a successful Production or Staging deployment, or currently diverge from the saved baseline. Escalate immediately and evaluate whether a hotfix issue is required.
   - **`PR_CODE_REVIEW_SPEC_DRIFT` (`critical` / `warning`)**: Requirements drifted after a pull request was merged, or while open pull requests are in code review. Notify the PR author and reviewers listed in `teamworkGraph.linkedPullRequests`.
   - **`STRATEGIC_GOAL_DRIFT_EXPOSURE` (`critical` / `warning`)**: Issue baseline drift compounds an `AT_RISK`, `OFF_TRACK`, or overdue Atlassian Goal in `teamworkGraph.linkedGoals`. Alert the Goal owner and `teamworkGraph.impactedTeams`.
   - **`DESIGN_HANDOFF_SPEC_DRIFT` (`warning` / `info`)**: Summary or description diverged from linked Figma / external design specifications marked `READY_FOR_DEVELOPMENT`.

### 2. Pre-Release Baseline Drift Sign-Off
1. Call `compare-issue-with-baseline(issueKey)` (optionally passing `snapshotName` when verifying against a named milestone such as `"Sprint 18 Release Baseline"`).
2. Interpret `outcome` accurately:
   - `"compared"`: report the summary verdict and list every entry in `differences` (`field`, `baseline`, `current`). Flag changes to `Summary`, `Description`, `Story Points`, `Assignee`, `Priority`, or `Due date`.
   - `"not-found"`: present the returned `availableSnapshots` list (`name`, `kind`, `created`) and ask the user which named baseline to compare against.
   - `"no-snapshot"`: explain that no baseline exists yet and instruct the user to create one in **Apps → Field History Admin → Baselines → Create Baseline**.

### 3. Sprint Scope Creep & Requirement Instability Triage
1. For candidate issues in a sprint or release, call `analyze-issue-field-churn(issueKey)`.
2. Inspect `topChangedFields` and `recentChanges`:
   - High churn (3+ edits) on `Summary`, `Description`, or acceptance-criteria fields after sprint start indicates requirement instability.
   - Combine with `inspect-issue-graph-impact(issueKey)` to quantify which Atlassian Goals, Teams, and DevOps pipelines are exposed to that instability.
