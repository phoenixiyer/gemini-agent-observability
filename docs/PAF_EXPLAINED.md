# Understanding the Prompt Amplification Factor (PAF)

## What is PAF?

The **Prompt Amplification Factor (PAF)** is the ratio of total backend API requests to user-initiated prompts in an agentic AI session.

```
PAF = Total Backend Prompts / User-Initiated Prompts
```

A PAF of 1.0 means every backend call was directly triggered by you. A PAF of 6.0 means for every prompt you typed, the system generated **5 additional hidden requests** behind the scenes.

---

## Why Does PAF Matter?

When you use Gemini CLI with agentic features, your single prompt often triggers a chain of events:

```
   You type:              "Refactor this function"
                                    │
                          ┌─────────┼─────────┐
                          ▼         ▼         ▼
                     Agent       Tool      Context
                    Reasoning    Calls    Assembly
                     (2-3x)    (1-5x)     (1-2x)
                          │         │         │
                          └────┬────┘         │
                               ▼              │
                         Backend API ◄────────┘
                          Requests
                          (5-15x)
```

Your **1 prompt** became **5-15 API calls**, each consuming tokens against your quota.

---

## The Three Root Causes of High PAF

### 1. Agentic Reasoning Loops

When `experimental.enableAgents` is set to `true` in your settings, the CLI enters an agentic mode where it:
- Plans a multi-step approach to your request
- Executes each step as a separate API call
- Reflects on results and adjusts its plan
- Sometimes **retries steps** when results are unsatisfactory

**Typical PAF contribution**: 2-5x

**Detection**: Look for multiple consecutive `"role": "user"` entries in `telemetry.log` that contain system-generated content rather than your actual input.

**Mitigation**:
- Disable `experimental.enableAgents` when not needed
- Use specific, well-scoped prompts to reduce reasoning steps
- Consider using the `--skill` flag for targeted tasks

---

### 2. MCP Server Retry Storms

MCP (Model Context Protocol) servers extend Gemini CLI's capabilities by providing tools like GitLab integration, database access, or Google Maps. When these servers are:
- **Unreachable**: The CLI retries the connection, each retry consuming tokens
- **Slow**: Timeouts trigger retries with increasing delay
- **Misconfigured**: Authentication failures are retried before surfacing the error

**Typical PAF contribution**: 1-10x (can be extreme during outages)

**Detection**: Run the `forensic-audit` skill and check the MCP Circuit Breaker Analysis section. Look for servers with > 3 consecutive failures.

**Mitigation**:
- Disable MCP servers you're not actively using
- Check server health before starting work sessions
- Use the `watchdog` skill to monitor MCP health in real-time

---

### 3. Instruction Bloat

Every API call includes your full instruction context — all GEMINI.md files (global + project), skill descriptions, and system prompts. If your GEMINI.md files are large:

| GEMINI.md Size | Est. Tokens per Call | Cost at 100 calls/day (3.0 Flash) | Cost at 100 calls/day (3.1 Pro) |
|:-:|:-:|:-:|:-:|
| 1,000 chars | ~250 tokens | ~$0.012/day | ~$0.050/day |
| 3,000 chars | ~750 tokens | ~$0.037/day | ~$0.150/day |
| 10,000 chars | ~2,500 tokens | ~$0.125/day | ~$0.500/day |
| 30,000 chars | ~7,500 tokens | ~$0.375/day | ~$1.500/day |

Multiply these numbers by your PAF to get the **real** daily cost of instruction bloat.

**Detection**: Run the `config-optimizer` skill to measure your actual instruction overhead.

**Mitigation**:
- Keep GEMINI.md under 3,000 characters
- Move specialized instructions into skills (loaded on-demand, not every call)
- Use project-level GEMINI.md only for project-specific context
- Avoid duplicating instructions across global and project files

---

## How to Calculate Your PAF

### Method 1: Using the Forensic Audit Skill (Recommended)
```bash
gemini --skill forensic-audit
```
The generated report includes the PAF calculation automatically.

### Method 2: Manual Calculation
```bash
# Count total "user" role entries in last 100 lines of telemetry
TOTAL_USER=$(tail -100 ~/.gemini/telemetry.log | grep -c '"role": "user"')

# Count your actual prompts (estimate from your session)
ACTUAL_PROMPTS=5  # adjust to your actual count

# Calculate PAF
echo "PAF = $TOTAL_USER / $ACTUAL_PROMPTS"
```

### Method 3: Using the Watchdog Skill
```bash
gemini --skill watchdog
```
The watchdog provides PAF trend analysis across multiple time windows and tracks historical readings.

---

## PAF Severity Scale

| PAF Range | Severity | What It Means |
|:-:|:-:|:-:|
| 1.0 - 2.0 | 🟢 **LOW** | Normal operation. Minimal amplification. |
| 2.0 - 4.0 | 🟡 **MEDIUM** | Some agent overhead. Worth investigating. |
| 4.0 - 8.0 | 🟠 **HIGH** | Significant hidden activity. Action recommended. |
| 8.0+ | 🔴 **CRITICAL** | Agent is likely in a loop. Immediate action required. |

---

## Real-World PAF Benchmarks

Based on observed patterns:

| Scenario | Typical PAF | Primary Driver |
|:-:|:-:|:-:|
| Simple Q&A, no tools | 1.0 - 1.5 | Minimal overhead |
| Code editing with tools | 2.0 - 3.0 | Tool call chains |
| Agentic mode, healthy MCP | 3.0 - 5.0 | Agent reasoning |
| Agentic mode, failing MCP | 5.0 - 15.0 | Retry storms |
| Bloated config + failing MCP | 10.0 - 30.0+ | Compound amplification |

---

## Key Takeaway

**Token cost ≠ Prompt count.**

Observability into the *agent layer* — not just the model layer — is the next critical capability for anyone running agentic AI in production. The PAF is your primary metric for understanding the true cost of your AI agent usage.

Use the AgentSkill toolkit to measure, monitor, and manage your PAF.
