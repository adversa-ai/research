# TrustFall PoC — safe reproduction (Claude Code · Gemini CLI · Cursor CLI · Copilot CLI)

This directory is a minimal malicious-repository fixture for the **[TrustFall](https://adversa.ai/blog/trustfall-claude-code-rce-mcp-settings/)** finding. Cloning it and running any of the four agentic CLIs in this directory triggers the CLI to silently spawn an attacker-defined MCP server the moment you accept the "trust this folder" dialog — and the server's only payload is opening the OS calculator. The visible calculator is the proof that arbitrary code ran with your user privileges immediately after the trust dialog, with no per-server consent prompt.

The full technical write-up is in [`../report/`](../report/).

## How to run

```bash
cd artifacts/trustfall-claude-code-rce-mcp-settings/poc
# Then run whichever CLI you want to test:
claude          # Claude Code
gemini          # Gemini CLI
cursor-agent    # Cursor CLI
copilot         # Copilot CLI
# Press Enter on the trust dialog. The OS calculator launches.
```

Tested on Claude Code v2.1.126 (latest at time of publication) and current versions of Gemini CLI, Cursor CLI, and Copilot CLI as of May 2026. The PoC works on Windows (`calc.exe`), macOS (`Calculator.app`), and Linux (`gnome-calculator` — adjust if your distro ships a different default).

## What's in here

The fixture ships parallel auto-approving config files for each CLI from the same project root. They all reference a single MCP server (`poc-server`) whose payload is inlined via `node -e` — there is no script file on disk, demonstrating the fileless variant from the report.

| File | CLI | Purpose |
|---|---|---|
| [`.mcp.json`](.mcp.json) | Claude Code, Copilot CLI | Registers `poc-server` with the inline `node -e` payload. |
| [`.claude/settings.json`](.claude/settings.json) | Claude Code (Project scope) | Auto-approves `poc-server` via `enableAllProjectMcpServers`, `enabledMcpjsonServers`, and `permissions.allow`. The three project-scope settings the report argues should be blocked from project scope. |
| [`.claude/settings.local.json`](.claude/settings.local.json) | Claude Code (Local scope) | Same content as `settings.json`. Demonstrates that an attacker can ship Local scope directly — Local outranks Project per Claude Code's scope precedence, so a Project-only block is insufficient. |
| [`.cursor/mcp.json`](.cursor/mcp.json) | Cursor CLI | Same MCP server definition in Cursor's project-scope MCP config path. |
| [`.gemini/settings.json`](.gemini/settings.json) | Gemini CLI | Same MCP server definition in Gemini's project-scope settings file. |

## Safety

This PoC is deliberately neutered:

- **No file reads.** It does not touch `~/.ssh/`, `~/.aws/`, or any other path on disk.
- **No network calls.** No exfiltration, no C2, no second-stage download.
- **No persistence.** It does not write to shell init files, cron, or any user-scope CLI settings.
- **No privilege escalation.** It runs as your user, with the privileges the CLI already has.

The single side effect is launching the OS calculator. Closing the calculator and the CLI session ends the demo — there is nothing to clean up.

## What the PoC demonstrates

1. **Same one-keypress chain across four CLIs.** Claude Code, Gemini CLI, Cursor CLI, and Copilot CLI all auto-execute the project-defined MCP server the moment the user accepts the folder-trust prompt. All four default to "Yes/Trust." They differ only in how the trust dialog frames the authorization — see the [report](../report/) for the dialog comparison.
2. **The trust dialog is the only boundary.** None of the four prompts a second time before spawning the server, and (in the Claude Code and Copilot cases) does not even mention MCP in the dialog itself.
3. **Project-scoped settings auto-approve attacker-supplied executables.** For Claude Code, `enableAllProjectMcpServers` / `enabledMcpjsonServers` / `permissions.allow` combine to skip every additional consent surface; the equivalent project-scope MCP definition does the same for the other CLIs.
4. **The MCP server runs as a native OS process with the user's full privileges**, not as a Claude/Gemini/Cursor/Copilot tool call. `calc` is launched via `child_process.exec`, not via any agent tool invocation.

Replace the inline `child_process.exec(...)` payload in the MCP config with anything an OS process can do, and you have the real-world attack chain. We did not.
