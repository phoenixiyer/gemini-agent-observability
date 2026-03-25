# Troubleshooting Guide

## Decision Tree: Which Skill Do I Use?

```
Start Here: What's your situation?
│
├─ "My quota is being exhausted and I don't know why"
│   └─→ Run `forensic-audit` first
│       └─ If forensic-audit hangs → Use `manual_diagnostics.md` instead
│
├─ "I want to understand my ongoing costs"
│   └─→ Run `cost-estimator`
│
├─ "I want to optimize my setup before issues occur"  
│   └─→ Run `config-optimizer`
│
├─ "I suspect an agent loop is happening right now"
│   └─→ Run `watchdog`
│       └─ If CLI is unresponsive → Use manual steps: Ctrl+C, then check telemetry.log
│
├─ "Support asked me to collect diagnostic info"
│   └─→ Run `collect-diagnostics`
│
└─ "The CLI itself is hanging/broken"
    └─→ Follow `manual_diagnostics.md` (zero API cost)
```

---

## Common Issues and Solutions

### Issue 1: "Quota Exhausted" with Low Prompt Count

**Symptoms**: You've only typed ~50-100 prompts today but hit the rate limit.

**Root Cause**: High PAF — hidden backend requests are consuming your quota.

**Solution**:
1. Run `gemini --skill forensic-audit`
2. Check the PAF score in the report
3. If PAF > 3:1, look at:
   - MCP server health (failing servers = retry storms)
   - `experimental.enableAgents` setting (agent loops)
   - GEMINI.md file sizes (instruction bloat)

---

### Issue 2: CLI Hangs or Becomes Unresponsive

**Symptoms**: Gemini CLI freezes, doesn't respond to input.

**Root Cause**: Usually an agent loop or blocked MCP server connection.

**Solution**:
1. Press `Ctrl+C` to interrupt
2. Do NOT run another agent skill (it may make it worse)
3. Follow `manual_diagnostics.md` for zero-cost diagnosis
4. Check `~/.gemini/telemetry.log` for the last entries
5. Look for repeating patterns in the log

---

### Issue 3: "Blocked Call" Errors

**Symptoms**: Repeated "blocked call" messages in the CLI output.

**Root Cause**: The `LocalAgentExecutor` is blocking shell tools, causing the agent to retry.

**Solution**:
1. Run the diagnostic shell commands manually (see `manual_diagnostics.md`)
2. Check if your sandbox settings are too restrictive
3. Consider adjusting the sandbox mode in `settings.json`

---

### Issue 4: High Costs Despite Low Usage

**Symptoms**: Billing shows higher-than-expected charges.

**Root Cause**: Could be model selection (Pro vs Flash) × PAF amplification.

**Solution**:
1. Run `gemini --skill cost-estimator` to get a cost breakdown
2. Check if you're using Pro when Flash would suffice
3. Run `gemini --skill config-optimizer` for targeted recommendations

---

### Issue 5: Skill or Extension Not Loading

**Symptoms**: A skill fails to execute or isn't recognized.

**Root Cause**: Usually a file path or format issue.

**Solution**:
1. Verify the skill file is in `~/.gemini/skills/` or linked via `gemini skills link`
2. Check the SKILL.md has valid YAML frontmatter with `name` and `description`
3. Run `gemini skills list` to confirm it's registered
4. Ensure there are no syntax errors in the frontmatter separators (`---`)

---

### Issue 6: Telemetry Log Not Being Generated

**Symptoms**: No `telemetry.log` file exists, can't calculate PAF.

**Root Cause**: Telemetry is not configured.

**Solution**:
Add this to your `~/.gemini/settings.json`:
```json
{
  "telemetry": {
    "enabled": true,
    "target": "local",
    "outfile": ".gemini/telemetry.log",
    "logPrompts": true
  }
}
```
Run a few prompts, then re-run the diagnostic skills.

---

## FAQ

**Q: Will running diagnostic skills consume more of my quota?**  
A: Yes, each skill invocation uses API calls. If you're already near your limit, use the `manual_diagnostics.md` steps instead — they require zero API calls.

**Q: Is my data safe when running these skills?**  
A: The skills include auto-redaction for API keys and tokens. However, **always review the generated report** before sharing it with anyone. The auto-redaction is regex-based and may miss some sensitive patterns.

**Q: Can I run these on Windows?**  
A: The shell commands use Unix syntax (`grep`, `sed`, `tail`). On Windows, use WSL or Git Bash, or adapt the commands for PowerShell.

**Q: How often should I run the watchdog?**  
A: Run it whenever you suspect unusual quota consumption, or periodically (e.g., weekly) to track your PAF trend. The historical readings in `paf_history.json` will show your trend over time.

**Q: What's a "good" PAF score?**  
A: Anything under 2.0 is healthy. Between 2-4 is worth monitoring. Above 4 needs attention. Above 8 is critical and indicates active loops or severe configuration issues.
