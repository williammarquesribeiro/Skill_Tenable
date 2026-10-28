---
name: tenable-shadow-ai-discovery
description: Find, classify and prioritize shadow AI (unapproved AI tools, LLM API usage, AI agents, AI browser extensions) in an environment monitored by Tenable One, using the Tenable MCP server. Use this skill whenever the user mentions shadow AI, unapproved or unsanctioned AI usage, AI inventory, AI governance evidence, AI exposure, or asks what AI tools are running in their Tenable environment, even if they do not say "shadow AI" explicitly. Also use it when the user wants a risk-ranked report of AI usage or wants to tag assets or open tickets for unapproved AI.
---

# Tenable Shadow AI Discovery

Discover which AI tools are actually in use across the environment, separate the approved ones from the rest, and rank what is left by risk. The goal is evidence a security team can act on, not a list of every AI domain ever seen.

## Prerequisites

This skill works through the Tenable MCP server (hosted by Tenable, exposes ~90 tools). Before anything else, check that Tenable tools are available in the session.

If they are not, stop and tell the user how to connect. Example for Claude Desktop (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "tenable": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://cloud.tenable.com/mcp/",
        "--header",
        "X-ApiKeys: accessKey=...;secretKey=..."
      ]
    }
  }
}
```

The API keys come from Tenable: Settings > My Account > API Keys. Tell the user to use a dedicated, least-privilege key. Never ask the user to paste keys into the chat.

## Step 1: Confirm the approved list

Ask once which AI tools are officially approved (for example: a cloud LLM service, an enterprise copilot, an internal AI gateway). This list decides what counts as "shadow". If the user does not know, proceed and mark every finding as "needs owner decision" instead of guessing.

## Step 2: Discover available tools

List the Tenable MCP tools and pick the ones related to AI exposure, assets, findings, cloud, attack surface, identity and tagging. Do not assume tool names. Names seen in Tenable's own demos include `workbenches_list_vulnerabilities`, `tagging_add_tags_assets`, `ticket_create_issue`, `dashboard_create_dashboard_from_template`, `scan_create` and `scan_launch`, but the catalog can change between releases.

Tenable One has an AI Exposure area that covers enterprise chat assistants, Microsoft 365 Copilot, AI agents and shadow AI. Prefer it whenever the tools are available, because it is the most direct source. If those tools are missing (module not licensed or not exposed through MCP), say so and use the fallback signals in Step 3.

## Step 3: Collect evidence

Use as many of these sources as the available tools allow, and record which ones were actually queried:

1. **AI Exposure inventory**: AI tools, agents and shadow AI already identified by Tenable.
2. **Asset and software inventory**: endpoints or servers with AI clients, local LLM runtimes (for example anything listening on port 11434), AI-related packages or browser extensions.
3. **Attack surface**: external domains and services that point to AI platforms.
4. **Cloud security**: accounts, roles and services that call AI provider APIs outside the approved path.
5. **Identity**: applications granted access through the identity provider (Entra ID, Okta) that are AI products.

Provider domains that are useful as signals when matching: `api.openai.com`, `chatgpt.com`, `api.anthropic.com`, `claude.ai`, `generativelanguage.googleapis.com`, `gemini.google.com`, `api.mistral.ai`, `api.cohere.ai`, `api-inference.huggingface.co`, `api.deepseek.com`, `perplexity.ai`, `copilot.microsoft.com`, `bedrock-runtime.*.amazonaws.com`. Treat this list as a starting point; new tools appear constantly.

## Step 4: Classify each finding

| Class | Meaning |
|---|---|
| Approved | Matches the approved list and the approved access path |
| Shadow | Known AI tool, not on the approved list |
| Bypass | Approved tool, but used outside the approved path (for example direct API calls that skip the gateway) |
| Suspicious | Traffic or software that looks like AI usage but cannot be confirmed |

Keep "Suspicious" honest. If the evidence is only a domain match, say that, and do not present it as confirmed usage.

## Step 5: Prioritize

Rank Shadow, Bypass and Suspicious findings by combining what the tools return:

- Asset criticality and exposure (internet-facing assets rank higher)
- Data sensitivity on the asset or account involved
- Privilege of the identity or role involved
- Exploitable vulnerabilities on the same asset (VPR, CISA KEV tagging, exploitability verification)

If a signal is not available in the tenant, leave it out and do not invent a value.

## Step 6: Report

Use this structure:

```
# Shadow AI report - [date]

## Executive summary
[3-4 sentences: how many findings, how many are high risk, biggest exposure]

## Coverage
[Sources actually queried, sources unavailable, known blind spots]

## Findings (highest risk first)
| # | AI tool | Class | Asset / account / identity | Evidence | Risk | Suggested action |

## Approved AI observed
[Short list, for the inventory]

## Recommendations
[Governance actions: onboard, block, or investigate]
```

State the blind spots plainly. Tenable sees assets, cloud, identity and the attack surface. It does not see AI use in a browser on an unmanaged device or on a personal phone, so recommend correlating with proxy, SASE or DNS logs when that matters.

## Actions (write operations)

The default is read-only. Only after the user explicitly asks, and after showing exactly what will change, the skill may:

- Tag affected assets (for example `shadow-ai`) so they can be tracked
- Open tickets in the connected ticketing tool for the top findings
- Create a dashboard for ongoing tracking

Tenable also asks for confirmation before write actions. Never bypass that step, and never run a new scan without the user's approval.

## Rules of evidence

- Report only what a tool returned. If a query returned nothing, say "no data returned", not "no shadow AI exists".
- Separate what was observed from what was inferred.
- Do not include secrets, API keys or personal data from the environment in the report.

## Language

Write this skill in English, but answer the user in the language they are writing in.