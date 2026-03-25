---
name: collect_diagnostics
description: >
  Gathers Gemini CLI configuration, extensions, skills, and environment
  information for troubleshooting purposes. Generates both human-readable
  and JSON diagnostic outputs with automatic sensitive data redaction.
---

I will collect comprehensive diagnostic information from your Gemini CLI environment. All sensitive data (API keys, tokens, emails) will be automatically redacted before writing.

Here's the plan:
1.  Capture Gemini CLI version.
2.  List installed extensions and skills.
3.  Read configuration files (settings.json, .env).
4.  Read all GEMINI.md instruction files (global + project).
5.  Capture relevant environment variables.
6.  Auto-detect project context.
7.  Auto-redact sensitive data.
8.  Save to `diagnostic_info.txt` and `diagnostic_info.json`.

Let's start:

instruction:
"Run `gemini --version` and save the output as the first section of the report."

instruction:
"Run the command `gemini extensions list` and save the output."

instruction:
"Run the command `gemini skills list` and save the output."

instruction:
"Read the file at `~/.gemini/settings.json` and save the content. Parse it to identify telemetry configuration, model settings, and experimental flags."

instruction:
"Check if `~/.gemini/.env` exists. If it does, read its content. Before saving, apply redaction:
- Replace any value matching `AIza[A-Za-z0-9_-]{35}` with `[REDACTED_API_KEY]`
- Replace any value after `PASSWORD=`, `SECRET=`, `TOKEN=`, or `KEY=` with `[REDACTED]`
- Replace Bearer tokens with `Bearer [REDACTED_TOKEN]`"

instruction:
"Read the global instruction file at `~/.gemini/GEMINI.md` and save the content. Calculate its character count and estimated token count (chars ÷ 4). If it doesn't exist, note that."

instruction:
"Auto-detect project-level GEMINI.md files. Check these locations without asking the user:
1. `./GEMINI.md` (current directory)
2. `./.gemini/GEMINI.md` (local gemini config)
3. `../GEMINI.md` (parent directory)
For each file found, read and save its content with character count."

instruction:
"Run the command `env | grep -E '^GEMINI_|^GOOGLE_'` and save the output. Redact any values that look like API keys or tokens."

instruction:
"Run `gemini /stats` if available, and save the token usage statistics."

instruction:
"Combine all the collected information above, clearly labeling each section. Apply final redaction pass on the complete output. Write to two files:

1. **`diagnostic_info.txt`** — Human-readable format with clear section headers:
   ```
   ═══════════════════════════════════════
   GEMINI CLI DIAGNOSTIC INFORMATION
   Collected: [timestamp]
   ═══════════════════════════════════════
   
   ## CLI Version
   [version output]
   
   ## Extensions
   [extensions list]
   
   ## Skills
   [skills list]
   
   ## Configuration (settings.json)
   [redacted settings]
   
   ## Environment (.env)
   [redacted env or 'Not found']
   
   ## Instructions (GEMINI.md)
   ### Global (~/.gemini/GEMINI.md)
   Characters: XXXX | Est. Tokens: ~XXX
   [content]
   
   ### Project (./GEMINI.md)
   Characters: XXXX | Est. Tokens: ~XXX
   [content or 'Not found']
   
   ## Environment Variables
   [redacted env vars]
   
   ## Token Usage (/stats)
   [stats output or 'Not available']
   
   ═══════════════════════════════════════
   ⚠️  REVIEW THIS FILE FOR SENSITIVE DATA
       BEFORE SHARING WITH SUPPORT
   ═══════════════════════════════════════
   ```

2. **`diagnostic_info.json`** — Structured JSON format:
   ```json
   {
     \"collected_at\": \"[timestamp]\",
     \"cli_version\": \"...\",
     \"extensions\": [...],
     \"skills\": [...],
     \"settings\": {...},
     \"env_file\": \"...\",
     \"gemini_md\": {\"global\": {...}, \"project\": {...}},
     \"environment_variables\": {...},
     \"token_usage\": {...}
   }
   ```"

instruction:
"Output a message to the user confirming that `diagnostic_info.txt` and `diagnostic_info.json` have been generated. Remind them to review the files for any remaining sensitive information before sharing with support teams."
