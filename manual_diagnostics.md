# Manual Diagnostic Steps

### *Zero API Cost : Zero Agent Loop Risk*

Use these steps when the Gemini CLI is hanging, you've exhausted your quota, or you want to avoid any additional API calls.

---

## Prerequisites
- Access to the terminal where Gemini CLI is installed
- Basic familiarity with shell commands

---

## Quick Script (One-Command Version)

Copy and paste this entire block into your terminal to generate a complete diagnostic report:

```bash
#!/bin/bash
REPORT="gemini_manual_diagnostic_$(date +%Y%m%d_%H%M%S).txt"

echo "═══════════════════════════════════════════════" > "$REPORT"
echo "  GEMINI CLI MANUAL DIAGNOSTIC REPORT" >> "$REPORT"
echo "  Generated: $(date -u +"%Y-%m-%dT%H:%M:%SZ")" >> "$REPORT"
echo "  Host: $(hostname)" >> "$REPORT"
echo "═══════════════════════════════════════════════" >> "$REPORT"

echo -e "\n## 1. CLI Version" >> "$REPORT"
gemini --version 2>&1 >> "$REPORT" || echo "  [ERROR: gemini command not found]" >> "$REPORT"

echo -e "\n## 2. Extensions" >> "$REPORT"
gemini extensions list 2>&1 >> "$REPORT" || echo "  [ERROR: could not list extensions]" >> "$REPORT"

echo -e "\n## 3. Billing Project" >> "$REPORT"
echo "  GOOGLE_CLOUD_PROJECT=${GOOGLE_CLOUD_PROJECT:-[NOT SET]}" >> "$REPORT"
echo "  GEMINI_MODEL=${GEMINI_MODEL:-[NOT SET]}" >> "$REPORT"

echo -e "\n## 4. Global Settings" >> "$REPORT"
if [ -f ~/.gemini/settings.json ]; then
    # Redact API keys before writing
    sed 's/AIza[A-Za-z0-9_-]\{35\}/[REDACTED_API_KEY]/g' ~/.gemini/settings.json >> "$REPORT"
else
    echo "  [NOT FOUND: ~/.gemini/settings.json]" >> "$REPORT"
fi

echo -e "\n## 5. Global GEMINI.md" >> "$REPORT"
if [ -f ~/.gemini/GEMINI.md ]; then
    CHARS=$(wc -c < ~/.gemini/GEMINI.md | tr -d ' ')
    echo "  Character count: $CHARS" >> "$REPORT"
    echo "  Est. tokens: ~$((CHARS / 4))" >> "$REPORT"
    [ "$CHARS" -gt 3000 ] && echo "  ⚠️  WARNING: Exceeds 3,000 char threshold (instruction bloat risk)" >> "$REPORT"
    echo "  ---content---" >> "$REPORT"
    cat ~/.gemini/GEMINI.md >> "$REPORT"
else
    echo "  [NOT FOUND: ~/.gemini/GEMINI.md]" >> "$REPORT"
fi

echo -e "\n## 6. Local Project Config" >> "$REPORT"
for f in ./.gemini/settings.json ./GEMINI.md ./.gemini/GEMINI.md ./.env; do
    if [ -f "$f" ]; then
        echo "  --- $f ---" >> "$REPORT"
        sed 's/AIza[A-Za-z0-9_-]\{35\}/[REDACTED_API_KEY]/g; s/Bearer [A-Za-z0-9._-]*/Bearer [REDACTED_TOKEN]/g' "$f" >> "$REPORT"
    fi
done

echo -e "\n## 7. Environment Variables" >> "$REPORT"
env | grep -E "^GEMINI_|^GOOGLE_" | sed 's/=.*KEY.*/=[REDACTED]/; s/=.*SECRET.*/=[REDACTED]/' >> "$REPORT"

echo -e "\n## 8. Telemetry Log Analysis" >> "$REPORT"
TLOG=""
[ -f ~/.gemini/telemetry.log ] && TLOG=~/.gemini/telemetry.log
[ -f ./.gemini/telemetry.log ] && TLOG=./.gemini/telemetry.log

if [ -n "$TLOG" ]; then
    echo "  Log file: $TLOG" >> "$REPORT"
    echo "  File size: $(ls -lh "$TLOG" | awk '{print $5}')" >> "$REPORT"
    
    TOTAL_USER=$(tail -200 "$TLOG" | grep -c '"role": "user"' || echo 0)
    TOTAL_MODEL=$(tail -200 "$TLOG" | grep -c '"role": "model"' || echo 0)
    AGENT_GEN=$(tail -200 "$TLOG" | grep -c '"agentGenerated":true' || echo 0)
    ERRORS=$(tail -200 "$TLOG" | grep -c '"level":"error"' || echo 0)
    
    echo "  Last 200 lines analysis:" >> "$REPORT"
    echo "    Total 'user' roles:     $TOTAL_USER" >> "$REPORT"
    echo "    Total 'model' roles:    $TOTAL_MODEL" >> "$REPORT"
    echo "    Agent-generated prompts: $AGENT_GEN" >> "$REPORT"
    echo "    Errors:                 $ERRORS" >> "$REPORT"
    
    if [ "$TOTAL_MODEL" -gt 0 ]; then
        PAF=$(echo "scale=1; $TOTAL_USER / $TOTAL_MODEL" | bc 2>/dev/null || echo "N/A")
        echo "    Estimated PAF:          $PAF:1" >> "$REPORT"
        
        if [ "$PAF" != "N/A" ] && [ "$(echo "$PAF > 5" | bc 2>/dev/null)" = "1" ]; then
            echo "    🚨 WARNING: PAF > 5:1 — Possible Hidden Agent Loop!" >> "$REPORT"
        fi
    fi
else
    echo "  [NOT FOUND: telemetry.log]" >> "$REPORT"
    echo "  Enable telemetry by adding to ~/.gemini/settings.json:" >> "$REPORT"
    echo '  { "telemetry": { "enabled": true, "target": "local", "outfile": ".gemini/telemetry.log", "logPrompts": true } }' >> "$REPORT"
fi

echo -e "\n═══════════════════════════════════════════════" >> "$REPORT"
echo "  ⚠️  REVIEW THIS FILE FOR SENSITIVE DATA" >> "$REPORT"
echo "      BEFORE SHARING WITH SUPPORT" >> "$REPORT"
echo "═══════════════════════════════════════════════" >> "$REPORT"

echo "✅ Diagnostic report saved to: $REPORT"
echo "📊 Review the file and redact any remaining sensitive data before sharing."
```

---

## Step-by-Step Manual Process

If you prefer to run each step individually:

### 1. Get Gemini CLI Version
```bash
gemini --version
```

### 2. List Installed Extensions
```bash
gemini extensions list
```

### 3. Check Billing Project
```bash
echo "Project: ${GOOGLE_CLOUD_PROJECT:-[NOT SET]}"
echo "Model: ${GEMINI_MODEL:-[NOT SET]}"
```

### 4. Capture Gemini CLI Stats
```bash
# Start Gemini CLI
gemini
# Inside the CLI, run:
/stats
# Exit with:
/quit
```

### 5. Check Configuration Files
```bash
# Global settings
cat ~/.gemini/settings.json 2>/dev/null || echo "Not found"

# Global instructions
cat ~/.gemini/GEMINI.md 2>/dev/null || echo "Not found"

# Local overrides
cat ./.gemini/settings.json 2>/dev/null || echo "Not found"
cat ./GEMINI.md 2>/dev/null || echo "Not found"
cat ./.env 2>/dev/null || echo "Not found"
```

### 6. Check Instruction Bloat
```bash
# Measure GEMINI.md sizes
wc -c ~/.gemini/GEMINI.md 2>/dev/null
wc -c ./GEMINI.md 2>/dev/null
# Flag anything over 3,000 characters
```

### 7. Check Environment Variables
```bash
env | grep -E "^GEMINI_|^GOOGLE_|^NODE_"
```

### 8. Enable Telemetry (If Not Active)
Add to `~/.gemini/settings.json`:
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

### 9. Analyze Telemetry Log
```bash
# Count user vs model entries
tail -200 ~/.gemini/telemetry.log | grep -c '"role": "user"'
tail -200 ~/.gemini/telemetry.log | grep -c '"role": "model"'

# Check for agent-generated prompts (PAF indicator)
tail -200 ~/.gemini/telemetry.log | grep -c '"agentGenerated":true'

# Check for errors (MCP failures)
tail -200 ~/.gemini/telemetry.log | grep -c '"level":"error"'
```

### 10. CRITICAL: Review and Redact
Before sharing **any** output, remove:
- ❌ API keys, passwords, tokens, credentials
- ❌ Private keys or certificates
- ❌ Internal hostnames or IP addresses
- ❌ Confidential project names or code
- ❌ Personal identifiable information (PII)

**When in doubt, redact it.**

### 11. Secure Submission
Share the redacted information through your secure Google Cloud Support case. Label each section clearly.
