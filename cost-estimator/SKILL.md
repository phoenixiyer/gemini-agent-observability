---
name: cost-estimator
description: >
  Estimates the real dollar cost of your Gemini CLI sessions by analyzing 
  telemetry logs and /stats output. Breaks down spend into user prompts, 
  agent overhead, and retry waste. Generates a `cost_report.md`.
---

I am initiating a cost analysis of your Gemini CLI usage. My goal is to estimate your actual dollar spend and identify where tokens are being wasted.

instruction:
"Create or overwrite a file named `cost_report.md` in the current directory. Start it with a header: '# Gemini CLI Cost Estimation Report' and the current timestamp."

instruction:
"Execute `gemini /stats` to capture real-time token consumption. Parse the output to extract:
- Total input tokens
- Total output tokens
- Model variant in use (e.g., gemini-3.0-flash, gemini-3.1-pro, gemini-2.5-flash)
Append this raw data to `cost_report.md` under a '## Raw Token Usage' section."

instruction:
"Read `~/.gemini/settings.json` to determine the configured model. Check for `model` or `defaultModel` fields. Also check for the `GEMINI_MODEL` environment variable via `echo $GEMINI_MODEL`. Append the detected model to the report under '## Model Configuration'."

instruction:
"Calculate estimated costs using these published pricing tiers (per 1M tokens). Create a pricing table in the report under '## Pricing Reference':

| Model | Input (per 1M tokens) | Output (per 1M tokens) | Notes |
|-------|----------------------|------------------------|-------|
| Gemini 3.1 Pro | $2.00 | $12.00 | Thinking tokens billed as output |
| Gemini 3.1 Flash-Lite | $0.25 | $1.50 | |
| Gemini 3.0 Flash | $0.50 | $3.00 | |
| Gemini 2.5 Pro | $1.25 | $10.00 | |
| Gemini 2.5 Flash | $0.15 | $0.60 | |
| Gemini 2.0 Flash | $0.10 | $0.40 | Legacy |

Apply the appropriate pricing tier based on the detected model to calculate:
- **Gross session cost**: total input tokens × input rate + total output tokens × output rate
Append the calculation to the report."

instruction:
"Forensic Log Analysis for Cost Attribution: Locate the telemetry log at `~/.gemini/telemetry.log` (or the path configured in settings.json). Use shell commands to:
1. Count total `\"role\": \"user\"` entries in the log (these represent ALL prompts including hidden agent prompts)
2. Estimate user-initiated prompts vs agent-generated prompts
3. Calculate the Prompt Amplification Factor (PAF) = total_backend_prompts / estimated_user_prompts

Create a '## Cost Attribution' section with:
- **User prompt cost**: gross_cost / PAF (your actual useful work)
- **Agent overhead cost**: gross_cost × (1 - 1/PAF) (hidden agent reasoning)
- **Estimated waste**: if PAF > 3, flag the agent overhead as 'excessive'

Format this as a clear breakdown table."

instruction:
"Analyze MCP server retry patterns in the telemetry log. Count entries that suggest retries or failures (look for 'error', 'retry', 'failed', 'timeout' patterns). Under '## Retry Waste Analysis':
- Estimate tokens wasted on failed/retried tool calls
- List which MCP servers contributed the most retries
- Calculate retry waste as a percentage of total cost"

instruction:
"**Gemini Code Assist Quota Analysis**: Under '## Request Quota Impact', determine the user's license tier and calculate the PAF-adjusted daily request budget:

| Tier | Daily Request Limit | With PAF X.X:1 | Effective User Prompts |
|------|:-------------------:|:--------------:|:---------------------:|
| Free (Google account login) | 1,000 RPD | ~1,000 / PAF | ~XXX prompts |
| Free (API Key, Flash only) | 250 RPD | ~250 / PAF | ~XXX prompts |
| Code Assist Standard | 1,500 RPD | ~1,500 / PAF | ~XXX prompts |
| Code Assist Enterprise | 2,000 RPD | ~2,000 / PAF | ~XXX prompts |

Highlight: 'At your current PAF of X.X:1, your effective daily prompt budget is only ~Y prompts before hitting quota (not the advertised Z).' Flag if the effective budget is less than 200 prompts."

instruction:
"Generate a '## Cost Optimization Recommendations' section with specific, actionable advice based on the findings:
1. If PAF > 5: 'CRITICAL — Disable experimental agent features or reduce GEMINI.md size'
2. If retry waste > 10%: 'HIGH — Disable or fix failing MCP server: [server_name]'
3. If using 3.1 Pro model: 'Consider switching to 3.0 Flash for routine tasks (4x cheaper input, 4x cheaper output)'
4. If using 2.5 Pro model: 'Consider switching to 3.0 Flash or 3.1 Flash-Lite for significant savings'
5. If GEMINI.md > 3000 chars: 'Reduce instruction context to save ~X tokens per prompt'
6. If effective daily prompt budget < 200: 'CRITICAL — Your PAF is consuming your daily request quota. Reducing PAF from X to 2 would give you Y more usable prompts per day.'
Include estimated monthly savings for each recommendation."

instruction:
"Add a '## Session Summary' section at the top of the report (after the header) with a quick-glance box:
```
╔══════════════════════════════════════════╗
║  ESTIMATED SESSION COST: $X.XXXX        ║
║  PROMPT AMPLIFICATION FACTOR: X.X:1     ║
║  WASTE RATIO: XX%                       ║
║  SEVERITY: [LOW/MEDIUM/HIGH/CRITICAL]   ║
╚══════════════════════════════════════════╝
```
Set severity based on: LOW (PAF<2), MEDIUM (PAF 2-4), HIGH (PAF 4-8), CRITICAL (PAF>8)."

instruction:
"Output a message to the user confirming that `cost_report.md` has been generated. Include the headline numbers: estimated cost, PAF, and severity level."
