---
name: watchdog
description: >
  Real-time monitoring skill that watches telemetry logs for signs of
  quota abuse, agent loops, and MCP failures. Provides live alerts
  when the Prompt Amplification Factor exceeds safe thresholds.
  Generates a `watchdog_report.md` with findings.
---

I am initiating the Watchdog monitoring skill. I will analyze your recent Gemini CLI telemetry for signs of runaway token consumption and agent loops.

instruction:
"Create or overwrite a file named `watchdog_report.md` in the current directory. Start it with a header: '# Gemini CLI Watchdog Report' and the current timestamp."

instruction:
"**Telemetry Log Location**: Check for the telemetry log file at these locations in order:
1. The path specified in `~/.gemini/settings.json` under `telemetry.outfile`
2. `~/.gemini/telemetry.log`
3. `./.gemini/telemetry.log`

If no telemetry log is found, write a '## ⚠️ Telemetry Not Configured' section in the report with instructions on how to enable it:
```json
{
  \"telemetry\": {
    \"enabled\": true,
    \"target\": \"local\",
    \"outfile\": \".gemini/telemetry.log\",
    \"logPrompts\": true
  }
}
```
If found, note the file path, size, and last modified time."

instruction:
"**Agent Loop Detection**: Analyze the telemetry log for repeating patterns that indicate infinite or excessive agent loops. Use shell commands to:

1. Extract the last 200 lines of the telemetry log
2. Count occurrences of `\"role\": \"user\"` entries — these represent ALL prompts (user + hidden agent)
3. Count occurrences of `\"role\": \"model\"` entries — these represent model responses
4. Look for repeating sequences: if the same tool call or prompt pattern appears more than 3 times consecutively, flag it as a potential loop

Under '## Agent Loop Detection':
- Report the user:model ratio (healthy is ~1:1, unhealthy is >3:1)
- List any detected repeating patterns with their count
- Assign a status: ✅ HEALTHY, ⚠️ ELEVATED, 🔴 LOOPING"

instruction:
"**PAF Trend Analysis**: Calculate the Prompt Amplification Factor across different time windows in the telemetry log. Under '## PAF Trend Analysis':

Analyze the log in chunks (if large enough) to show how PAF changes over time:
- Last 10 entries: PAF = X.X
- Last 50 entries: PAF = X.X  
- Last 200 entries: PAF = X.X

Show the trend direction:
- 📈 INCREASING (getting worse)
- 📉 DECREASING (improving)  
- ➡️ STABLE (consistent)

If PAF exceeds 5:1 at any window, output a prominent warning:
```
🚨 ALERT: Prompt Amplification Factor exceeded 5:1 threshold!
   Current PAF: X.X:1
   This means for every 1 prompt you type, the agent is generating X.X hidden backend requests.
```"

instruction:
"**MCP Server Health Monitor**: Check the telemetry log for MCP-related entries. Under '## MCP Server Health':

For each MCP server referenced in the log:
1. Count successful calls vs errors/timeouts
2. Calculate uptime percentage
3. Identify retry storms (>3 consecutive failures to the same server)

Format as a status dashboard:
| MCP Server | Calls | Errors | Uptime | Status |
|-----------|-------|--------|--------|--------|
| server_name | XX | XX | XX% | ✅/⚠️/🔴 |

Flag any server with uptime < 90% as a retry storm risk."

instruction:
"**Token Velocity Monitor**: Analyze the rate of token consumption from the telemetry log. Under '## Token Velocity':
- Calculate tokens consumed per minute (if timestamps are available in the log)
- Estimate daily consumption rate based on the observed velocity
- Compare against known quota limits:
  - Free tier (Google login): 1,000 requests/day
  - Code Assist Standard: 1,500 requests/day
  - Code Assist Enterprise: 2,000 requests/day

If the observed velocity would exhaust the daily quota in less than 8 hours of active use, flag it:
```
⚠️ At current velocity, you will exhaust your daily quota in ~X.X hours
```"

instruction:
"**Historical PAF Tracking**: Check if `~/.gemini/paf_history.json` exists. If it does, read it and append the current PAF reading with a timestamp. If it doesn't, create it with the current reading.

The format should be:
```json
{
  \"readings\": [
    {\"timestamp\": \"2025-01-15T10:30:00Z\", \"paf\": 4.2, \"window\": 50},
    {\"timestamp\": \"2025-01-15T14:00:00Z\", \"paf\": 2.1, \"window\": 50}
  ]
}
```

Under '## Historical PAF Trend' in the report, show the last 10 readings (if available) and indicate whether the overall trend is improving or degrading."

instruction:
"Generate a '## Watchdog Summary' section at the top of the report (after the header) with a quick-glance status:

```
╔══════════════════════════════════════════════════════════╗
║  WATCHDOG STATUS DASHBOARD                              ║
║                                                         ║
║  Agent Loops:     ✅ HEALTHY / ⚠️ ELEVATED / 🔴 LOOPING  ║
║  Current PAF:     X.X:1                                 ║
║  PAF Trend:       📈/📉/➡️                                ║
║  MCP Health:      X/Y servers healthy                   ║
║  Token Velocity:  ~XXX tokens/min                       ║
║  Quota Risk:      LOW / MEDIUM / HIGH / CRITICAL        ║
╚══════════════════════════════════════════════════════════╝
```"

instruction:
"Output a message to the user with the Watchdog summary status. If any alerts were triggered (PAF > 5, loop detected, MCP failures), highlight them prominently. Confirm that `watchdog_report.md` has been generated and that the PAF reading has been saved to `~/.gemini/paf_history.json` for trend tracking."
