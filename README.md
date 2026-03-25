# Gemini CLI Diagnostic Toolkit

### *Troubleshooting Quota Exhaustion, Configuration Overheads, and Agentic Loops.*

<p align="center">
  <strong>Find the Smoking Gun behind your unexpected token consumption.</strong>
</p>

---

> **Your 1 prompt ≠ 1 API call.**  
> In agentic mode, a single user prompt can trigger 5–15 hidden backend requests through agent reasoning chains, failing MCP servers, and instruction bloat. This is the **Prompt Amplification Factor (PAF)**.

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                     AgentSkill Toolkit                           │
├──────────┬──────────┬──────────┬──────────┬─────────────────────┤
│  collect │ forensic │   cost   │  config  │     watchdog        │
│ -diagnos │  -audit  │-estimator│-optimizer│                     │
│   tics   │          │          │          │                     │
├──────────┴──────────┴──────────┴──────────┴─────────────────────┤
│                    Gemini CLI Skills API                         │
├─────────────────────────────────────────────────────────────────┤
│  telemetry.log  │  settings.json  │  GEMINI.md  │  MCP Servers │
└─────────────────┴─────────────────┴─────────────┴──────────────┘
```

---

## Skill Catalog

| Skill | Purpose | Output | API Cost |
|:------|:--------|:-------|:---------|
| **`collect-diagnostics`** | Full environment snapshot for support teams | `diagnostic_info.txt` + `.json` | Low |
| **`forensic-audit`** | Deep-state PAF analysis with severity scoring | `gemini_diagnostic_report.md` + `.json` | Medium |
| **`cost-estimator`** | Token-to-dollar cost breakdown | `cost_report.md` | Medium |
| **`config-optimizer`** | Configuration bloat detection & optimization | `optimization_report.md` | Low |
| **`watchdog`** | Real-time PAF monitoring & loop detection | `watchdog_report.md` | Medium |
| **Manual Steps** | Zero-API-cost shell-based diagnostics | Terminal output | **Zero** |

---

## Skills in Detail

### 🔍 `collect-diagnostics`
A lightweight diagnostic data collector for support escalations.
- Captures CLI version, extensions, skills, configs, and env vars
- Auto-redacts API keys, tokens, and sensitive data
- Outputs both human-readable `.txt` and structured `.json`

### 🔬 `forensic-audit`
The flagship deep-state diagnostic skill.
- **PAF Calculation**: Measures hidden backend prompts vs your actual inputs
- **MCP Circuit Breaker**: Detects retry storms per server (not just health pings)
- **Historical PAF Tracking**: Saves readings to `~/.gemini/paf_history.json` for trend analysis
- **Severity Scoring**: Assigns CRITICAL/HIGH/MEDIUM/LOW to each finding
- **Enhanced Redaction**: Masks API keys, emails, tokens, private keys, and file paths
- **Dual Output**: Generates both Markdown report and structured JSON for the dashboard

### 💰 `cost-estimator`
Maps token consumption to actual dollar costs.
- Applies published Gemini pricing tiers (3.0/3.1 and 2.5)
- Breaks down cost: **your prompts** vs **agent overhead** vs **retry waste**
- Calculates cost per PAF-adjusted prompt
- Provides optimization recommendations with estimated monthly savings

### ⚙️ `config-optimizer`
Proactive configuration analysis before issues occur.
- **Instruction Bloat Score**: Measures all GEMINI.md files and skill descriptions
- **Token Budget Calculator**: Shows per-prompt overhead in tokens and dollars
- **Settings Risk Matrix**: Flags dangerous experimental features
- **Extension Audit**: Identifies unused or conflicting extensions

### 🐕 `watchdog`
Real-time monitoring for active sessions.
- **Agent Loop Detection**: Identifies repeating prompt patterns
- **PAF Trend Analysis**: Shows PAF across multiple time windows (10/50/200 entries)
- **MCP Health Monitor**: Server uptime, error rates, retry storm detection
- **Token Velocity**: Tracks rate against Gemini Code Assist daily quotas
- **Historical Tracking**: Appends readings to `paf_history.json` for long-term trends

---

## Getting Started

### Installation
1. Ensure you have the latest Gemini CLI:
    ```bash
    brew upgrade gemini-cli
    ```

2. Clone this repo:
    ```bash
    git clone https://github.com/phoenixiyer/AgentSkill.git
    ```

3. Copy all skills to your Gemini skills directory:
    ```bash
    cp -r AgentSkill/forensic-audit ~/.gemini/skills/
    cp -r AgentSkill/collect-diagnostics ~/.gemini/skills/
    cp -r AgentSkill/cost-estimator ~/.gemini/skills/
    cp -r AgentSkill/config-optimizer ~/.gemini/skills/
    cp -r AgentSkill/watchdog ~/.gemini/skills/
    ```

### Usage

**Run a specific skill:**
```bash
gemini --skill forensic-audit
gemini --skill cost-estimator
gemini --skill config-optimizer
gemini --skill watchdog
gemini --skill collect-diagnostics
```

**Or in Gemini CLI interactive mode:**
```
Help me run a forensic audit
Help me estimate my costs
Optimize my configuration
Run the watchdog monitor
Collect diagnostic information
```

### Visual Dashboard
After running `forensic-audit` (which generates `gemini_diagnostic_report.json`), open the visual dashboard:
```bash
open templates/report_dashboard.html
```
Load your JSON report file into the dashboard for interactive visualization of findings, PAF trends, MCP health, and cost breakdowns.

---

## Project Structure

```
AgentSkill/
├── README.md                          # This file
├── manual_diagnostics.md              # Zero-API-cost manual steps
│
├── collect-diagnostics/
│   └── SKILL.md                       # Environment data collector
├── forensic-audit/
│   └── SKILL.md                       # Deep-state PAF analysis
├── cost-estimator/
│   └── SKILL.md                       # Token-to-dollar cost mapping
├── config-optimizer/
│   └── SKILL.md                       # Configuration optimization
├── watchdog/
│   └── SKILL.md                       # Real-time monitoring
│
├── docs/
│   ├── PAF_EXPLAINED.md               # Deep-dive on Prompt Amplification Factor
│   └── TROUBLESHOOTING.md             # Troubleshooting guide with decision tree
│
├── examples/
│   ├── sample_report.md               # Example forensic audit report
│   └── sample_telemetry.log           # Example telemetry log (PAF ~6:1)
│
└── templates/
    └── report_dashboard.html          # Interactive HTML dashboard
```

---

## Key Concept: Prompt Amplification Factor (PAF)

The PAF is the ratio of total backend API requests to user-initiated prompts:

```
PAF = Total Backend Prompts / Your Actual Prompts
```

| PAF | Severity | Meaning |
|:---:|:--------:|:--------|
| 1–2 | 🟢 Low | Normal operation |
| 2–4 | 🟡 Medium | Some agent overhead, worth investigating |
| 4–8 | 🟠 High | Significant hidden activity, action needed |
| 8+ | 🔴 Critical | Active agent loop, immediate action required |

**Three root causes of high PAF:**
1. 🔁 **Agentic Loops** — internal reasoning chains that retry with refined prompts
2. 💀 **MCP Retry Storms** — failing tool connections triggering silent retries
3. 📜 **Instruction Bloat** — oversized GEMINI.md files prepended to every call

→ Read [PAF_EXPLAINED.md](docs/PAF_EXPLAINED.md) for the full deep-dive.

---

## Disclaimer

> [!IMPORTANT]
> **Experimental Status:** These skills are **experimental** and intended for diagnostic/troubleshooting purposes on non-production environments.
>
> **Not an Official Product:** This is **not** an official Google product. Provided "as-is" without warranties. Use at your own risk.
>
> **Data Privacy:** While skills include auto-redaction for API keys and tokens, **always review generated reports** for sensitive information before sharing.
>
> **Support:** Maintained on a best-effort basis. No SLA or official Google Cloud support.

---

## Troubleshooting

If any skill **hangs or loops**, it's likely because the `LocalAgentExecutor` is blocking shell tools. To bypass:

1. Press `Ctrl+C` to interrupt
2. Follow [manual_diagnostics.md](manual_diagnostics.md) for zero-API-cost diagnosis
3. Run the shell commands from the SKILL.md files directly in your terminal

See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for the complete troubleshooting guide.
