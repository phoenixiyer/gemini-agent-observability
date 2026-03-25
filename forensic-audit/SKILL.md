---
name: forensic-audit
description: >
  Performs a deep-state forensic audit to identify token leaks and hidden 
  backend prompt amplification. Generates a comprehensive `gemini_diagnostic_report.md` 
  and `gemini_diagnostic_report.json` containing global/local configs, MCP health, 
  usage stats, PAF trends, and severity-scored findings.
---

I am initiating a deep-state forensic audit of the Gemini CLI environment. My goal is to determine the "Prompt Amplification Factor" and identify why your quota is being exhausted.

instruction:
"Create or overwrite a file named `gemini_diagnostic_report.md` in the current directory. Start it with a header: '# Gemini CLI Forensic Diagnostic Report' and the current timestamp. Also initialize an empty JSON structure in memory that we will write to `gemini_diagnostic_report.json` at the end."

instruction:
"Run `gemini --version` first. Then run `gemini extensions list`, `gemini skills list`, and `gemini /mcp list`. Append the output of all these commands to `gemini_diagnostic_report.md` under a '## System Components' section. Store each output in the JSON structure under `system_components`."

instruction:
"Read the global `~/.gemini/settings.json` and `~/.gemini/GEMINI.md`. Append their contents to the report. Specifically, check if `experimental.enableAgents` is true, as this is the primary driver of hidden prompts. Calculate the character count and estimated token count (chars ÷ 4) for GEMINI.md. Add to JSON under `global_config`."

instruction:
"Check for local project overrides: Read `./.gemini/settings.json`, `./GEMINI.md`, and `./.env`. Before appending to the report, apply enhanced redaction using these patterns:
- API keys: replace any string matching `AIza[A-Za-z0-9_-]{35}` with `[REDACTED_API_KEY]`
- Bearer tokens: replace `Bearer [A-Za-z0-9._-]+` with `Bearer [REDACTED_TOKEN]`
- Email addresses: replace `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` with `[REDACTED_EMAIL]`
- GCP project IDs in sensitive contexts: flag but don't auto-redact (user may want these)
- File paths containing usernames: replace `/Users/[username]/` with `/Users/[REDACTED]/`
- Private keys: replace any multi-line block starting with `-----BEGIN` with `[REDACTED_PRIVATE_KEY]`

If a local `GEMINI.md` exists, calculate its character count and flag if it exceeds 3,000 characters. Add to JSON under `local_config`."

instruction:
"Execute `gemini /stats` to get real-time token consumption. Append this to a '## Usage Statistics' section in the report. Parse the output to extract input tokens, output tokens, and model name. Add to JSON under `usage_stats`."

instruction:
"Capture relevant environment variables by running `env | grep -E '^(GEMINI_|GOOGLE_)'`. Apply redaction to any values that look like API keys or tokens before appending to the report. Add to JSON under `environment`."

instruction:
"**MCP Circuit Breaker Analysis**: Go beyond simple health pings. For each MCP server found in the extensions list:
1. Attempt to ping/connect to the server
2. Check the telemetry log for historical failure patterns for this server
3. Count: total calls, successful calls, errors, timeouts, consecutive failures

Under '## MCP Circuit Breaker Analysis', create a table:
| MCP Server | Total Calls | Successes | Errors | Timeouts | Consecutive Failures | Status |
|-----------|------------|-----------|--------|----------|---------------------|--------|
| server_name | XX | XX | XX | XX | XX | ✅ HEALTHY / ⚠️ DEGRADED / 🔴 OPEN (tripped) |

A server is 'OPEN' (circuit tripped) if it has > 5 consecutive failures. This is a major retry storm risk.
Add to JSON under `mcp_circuit_breaker`."

instruction:
"**Forensic Log Analysis**: Locate the telemetry log path from settings.json (typically `~/.gemini/telemetry.log`). Use shell commands to:
1. Count the number of `\"role\": \"user\"` entries in the last 100 lines
2. Count the number of `\"role\": \"model\"` entries
3. Count the number of tool call entries
4. Identify any repeating prompt patterns (potential loops)
5. Calculate the Prompt Amplification Factor (PAF) = total_user_roles / estimated_real_user_prompts

If PAF > 3:1, explicitly write a prominent warning in the report about 'Hidden Agent Looping'.
If PAF > 8:1, escalate to 'CRITICAL: Severe Agent Loop Detected'.
Add to JSON under `forensic_analysis`."

instruction:
"**Historical PAF Tracking**: Check if `~/.gemini/paf_history.json` exists. If it does, read it. Append the current PAF reading with timestamp to the history:
```json
{\"timestamp\": \"[current_time]\", \"paf\": [calculated_paf], \"severity\": \"[severity]\"}
```
Write the updated history back to `~/.gemini/paf_history.json`.

Under '## PAF Trend History' in the report, show the last 10 readings and indicate whether PAF is improving (📉), degrading (📈), or stable (➡️). Add to JSON under `paf_history`."

instruction:
"**Severity Scoring**: Assign a severity level to EACH finding in the report. Under '## Severity Summary', create a findings table:

| # | Finding | Severity | Impact | Recommendation |
|---|---------|----------|--------|---------------|
| 1 | [finding description] | 🔴 CRITICAL / 🟠 HIGH / 🟡 MEDIUM / 🟢 LOW | [impact] | [fix] |

Severity criteria:
- 🔴 CRITICAL: PAF > 8, MCP circuit breaker tripped, active agent loop
- 🟠 HIGH: PAF 4-8, instruction bloat > 5K chars, experimental agents enabled
- 🟡 MEDIUM: PAF 2-4, instruction bloat 3-5K chars, unused extensions
- 🟢 LOW: PAF < 2, minor config issues

Add to JSON under `findings`."

instruction:
"Finalize the report by adding a '## Debugging Insights' section. Based on ALL gathered data, provide a prioritized summary:

```
╔══════════════════════════════════════════════════════════╗
║  FORENSIC AUDIT SUMMARY                                ║
║                                                         ║
║  Overall Severity:  [CRITICAL/HIGH/MEDIUM/LOW]          ║
║  PAF Score:         X.X:1                               ║
║  PAF Trend:         📈/📉/➡️                              ║
║  MCP Health:        X/Y servers healthy                 ║
║  Findings:          X critical, Y high, Z medium        ║
║                                                         ║
║  Root Cause Assessment:                                 ║
║  1. [Primary cause]                                     ║
║  2. [Secondary cause]                                   ║
║  3. [Contributing factor]                               ║
╚══════════════════════════════════════════════════════════╝
```

Then write the complete JSON structure to `gemini_diagnostic_report.json`."

instruction:
"Output a message to the user confirming that both `gemini_diagnostic_report.md` and `gemini_diagnostic_report.json` have been generated. Show the overall severity, PAF score, and number of critical findings. Remind the user to review the report for any remaining sensitive information before sharing."
