---
name: gears-diagram-mcp
description: Search, inspect, lint, and turn Confluence architecture diagrams and flowcharts (Gears Draw) into code, API contracts, or test cases via Atlassian Rovo MCP.
---

# Gears Diagram MCP Skill

Use this skill when connecting an MCP-compatible AI client (Claude Code, Claude Desktop, Cursor, Codex, or custom Rovo Studio agents) to the **Gears Diagram MCP** server (`gears-diagram-mcp`).

## Connection Setup

1. Ensure the **Gears Draw for Confluence** app is installed on your Confluence Cloud site and **Atlassian Rovo** is enabled.
> **Consent first:** Before calling tools from an external client, the same user opens the app once in Confluence, selects **Allow access**, and runs one prompt with **Gears Diagram Assistant** in Rovo Chat. Forge needs this app consent entry for remote `asUser()` calls; without it, tool calls can fail.

2. In **Atlassian Administration → Apps → Sites → [Your Site] → Connected apps**, open **Gears Draw** and enable external MCP exposure for:
   - `gears-search-diagrams`
   - `gears-read-diagrams`
3. Connect your MCP client to:
   - Commercial app endpoint: `https://mcp.atlassian.com/forge/6e422ada-f9e0-4586-8df0-dcab43785836`
   - Codegeist 2026 Judge Edition endpoint: `https://mcp.atlassian.com/forge/9a9b5ba3-89c8-4923-b7da-ad5d931773b3`

> **Permission Boundary:** Every tool call executes `asUser` using your OAuth 2.1 identity. You will never read pages or diagrams you do not have permission to view in Confluence.

---

## Core Operating Rules (External MCP Clients)

- **No implicit page context:** External IDE/CLI agents are not inside a Confluence browser tab. Always pass `pageId` (numeric Confluence content ID) when calling `gears-read-diagrams`.
- **Search first when `pageId` is unknown:** If the user names a system, service, topic, or page title rather than a numeric `pageId`, call `gears-search-diagrams` first to locate matching pages and obtain their numeric `id`.
- **Never hallucinate topology:** Base all architectural conclusions strictly on `diagrams[].pages[].nodes`, `edges`, `inside` (swimlane/container hierarchy), and `checks` returned by `gears-read-diagrams`.

---

## Standard Workflows (SOPs)

### 1. Architecture Review & Spec-to-Diagram Drift Check
1. If no `pageId` is given, call `gears-search-diagrams` with the system or feature keyword (optionally scoped by `spaceKey`).
2. Call `gears-read-diagrams` with `pageId` (and `macroId` if the user targets one specific diagram).
3. Report deterministic graph lint issues from `checks` first (`isolatedNodes`, `unlabeledEdges`, `danglingEdges`, `duplicateLabels`, `emptyContainers`).
4. Cross-check `page.text` (written Confluence specification) against `nodes` and `edges` to flag components or flows mentioned in prose but missing from the diagram (or vice versa).
5. Evaluate architectural risks (single points of failure, missing auth/trust boundaries, unbuffered synchronous chains) and rate findings `High` / `Medium` / `Low`.

### 2. Diagram-to-Code & Test Case Generation
1. Fetch the target flowchart or sequence/architecture diagram via `gears-read-diagrams(pageId)`.
2. Trace every directed edge (`from` → `to`, including branch `label` conditions) and container boundary (`inside`).
3. When generating **test cases**: produce one happy-path test per end-to-end path and one edge-case test per decision branch (`rhombus` / labeled outgoing edges), citing the exact node labels.
4. When scaffolding **code or API contracts**: map containers/swimlanes to modules or services, nodes to handlers/stores, and labeled edges to typed request/event interfaces.

### 3. Dependency Blast-Radius Discovery Across Confluence
1. Call `gears-search-diagrams(query: "<component or datastore>", spaceKey: "<optional>")` (for example `"Kafka"`, `"Redis"`, `"Auth Service"`).
2. Present matching pages with links and `matchingLabels`. If zero results return, remind the user that only diagrams saved after content indexing was enabled are indexed.
3. For each critical hit, call `gears-read-diagrams(pageId)` to list upstream callers and downstream dependencies of that component.
