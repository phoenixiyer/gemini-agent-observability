---
name: config-optimizer
description: >
  Analyzes your Gemini CLI configuration for performance and cost issues.
  Detects instruction bloat, risky experimental flags, unused extensions,
  and token overhead. Generates an `optimization_report.md` with actionable fixes.
---

I am initiating a configuration optimization analysis. My goal is to identify configuration issues that inflate your token usage and increase costs.

instruction:
"Create or overwrite a file named `optimization_report.md` in the current directory. Start it with a header: '# Gemini CLI Configuration Optimization Report' and the current timestamp."

instruction:
"**Instruction Bloat Analysis**: Read all GEMINI.md files that contribute to the prompt context:
1. Global: `~/.gemini/GEMINI.md`
2. Project: `./GEMINI.md`
3. Local: `./.gemini/GEMINI.md`

For each file found, calculate and report:
- Character count
- Estimated token count (chars ÷ 4 as rough estimate)
- Number of lines
- Whether it exceeds the 3,000 character threshold

Create a '## Instruction Bloat Analysis' section with a table:

| File | Characters | Est. Tokens | Lines | Status |
|------|-----------|-------------|-------|--------|
| ~/.gemini/GEMINI.md | X,XXX | ~XXX | XX | ⚠️ BLOATED / ✅ OK |

Calculate the **total instruction overhead per prompt** = sum of all GEMINI.md tokens. Flag if total exceeds 1,000 tokens (this means ~$0.0005 wasted per prompt on 3.0 Flash, or ~$0.002 wasted per prompt on 3.1 Pro)."

instruction:
"**Settings Risk Analysis**: Read `~/.gemini/settings.json` and any local `./.gemini/settings.json`. Under '## Settings Risk Analysis', check for and flag:

| Setting | Value | Risk Level | Impact |
|---------|-------|-----------|--------|
| `experimental.enableAgents` | true/false | 🔴 HIGH if true | Enables hidden agent reasoning chains, major PAF driver |
| `telemetry.enabled` | true/false | ℹ️ INFO | Required for diagnostics |
| `telemetry.logPrompts` | true/false | ℹ️ INFO | Required for PAF calculation |
| `sandbox` mode | value | 🟡 MEDIUM | Affects tool execution behavior |
| `model` override | value | ℹ️ INFO | Cost varies 10x between Flash and Pro |

For each risky setting, provide a specific recommendation."

instruction:
"**Extension Audit**: Run `gemini extensions list` and list all active extensions. Under '## Extension Audit':
- List each extension with its type (MCP server, tool, etc.)
- Flag extensions that are known to be high-overhead (e.g., those that inject large system prompts)
- Identify any extensions that might conflict with each other
- Recommend removing unused extensions to reduce context size

Format as a table:
| Extension | Type | Status | Recommendation |
|-----------|------|--------|---------------|"

instruction:
"**Skills Audit**: Run `gemini skills list` and enumerate all loaded skills. Under '## Skills Audit':
- List each skill with its description length
- Flag skills with descriptions > 500 characters (they add to every prompt's context)
- Identify redundant skills (multiple skills doing similar things)
- Calculate total token overhead from all skill descriptions"

instruction:
"**Environment Variable Check**: Run `env | grep -E '^(GEMINI_|GOOGLE_)'` and analyze under '## Environment Configuration':
- Check if `GOOGLE_CLOUD_PROJECT` is set (billing project)
- Check if `GEMINI_MODEL` overrides the settings.json model
- Flag any unexpected or conflicting environment variables
- Verify the billing project matches the intended quota pool"

instruction:
"**Token Budget Calculator**: Under '## Token Budget Impact', calculate the per-prompt overhead:

```
╔══════════════════════════════════════════════════════════╗
║  PER-PROMPT OVERHEAD BREAKDOWN                          ║
║                                                         ║
║  GEMINI.md instructions:     ~XXX tokens                ║
║  Skill descriptions:         ~XXX tokens                ║
║  Extension contexts:         ~XXX tokens                ║
║  System prompt (estimated):  ~500 tokens                ║
║  ─────────────────────────────────────────               ║
║  TOTAL OVERHEAD PER PROMPT:  ~X,XXX tokens              ║
║                                                         ║
║  At 100 prompts/day:                                    ║
║    3.0 Flash: ~$X.XX/day ($XX/month)                   ║
║    3.1 Pro:   ~$X.XX/day ($XX/month)                   ║
╚══════════════════════════════════════════════════════════╝
```"

instruction:
"Generate a '## Optimization Recommendations' section, prioritized by impact:

1. **CRITICAL** items (fix immediately to stop quota drain)
2. **HIGH** items (significant cost savings)
3. **MEDIUM** items (nice-to-have optimizations)
4. **LOW** items (minor improvements)

For each recommendation, include:
- What to change
- Expected token savings per prompt
- Estimated monthly cost savings
- Risk of the change (e.g., 'safe', 'may reduce agent capability')"

instruction:
"Output a message to the user confirming that `optimization_report.md` has been generated. Include the headline: total per-prompt overhead in tokens and the number of critical/high recommendations found."
