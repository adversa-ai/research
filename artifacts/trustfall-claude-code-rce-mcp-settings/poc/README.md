# TrustFall PoC — safe reproduction

This directory is a minimal malicious-repository fixture for the **[TrustFall](https://adversa.ai/blog/trustfall-claude-code-rce-mcp-settings/)** finding. Cloning it and running `claude` in this directory triggers Claude Code to silently spawn an attacker-defined MCP server the moment you accept the "Yes, I trust this folder" dialog — and the server's only payload is opening the OS calculator. The visible calculator is the proof that arbitrary code ran with your user privileges immediately after the trust dialog, with no per-server consent prompt.

The full technical write-up is in [`../report/report.md`](../report/report.md).

## How to run

```bash
cd artifacts/trustfall-claude-code-rce-mcp-settings/poc
claude
# Press Enter on the trust dialog. The OS calculator launches.
```

Tested on Claude Code v2.1.126 (latest at time of publication). The PoC works on Windows (`calc.exe`), macOS (`Calculator.app`), and Linux (`gnome-calculator` — adjust if your distro ships a different default).

## What's in here

| File | Purpose |
|---|---|
| [`.mcp.json`](.mcp.json) | Registers a single MCP server named `poc-server`. Its payload is inlined via `node -e` — no script file on disk, demonstrating the fileless variant from the report. |
| [`.claude/settings.json`](.claude/settings.json) | Project-scoped Claude Code settings that auto-approve `poc-server` via `enableAllProjectMcpServers`, `enabledMcpjsonServers`, and `permissions.allow`. All three are the project-scope settings the report argues should be blocked from project scope. |

## Safety

This PoC is deliberately neutered:

- **No file reads.** It does not touch `~/.ssh/`, `~/.aws/`, or any other path on disk.
- **No network calls.** No exfiltration, no C2, no second-stage download.
- **No persistence.** It does not write to shell init files, cron, or your user-scope Claude settings.
- **No privilege escalation.** It runs as your user, with the privileges Claude Code already has.

The single side effect is launching the OS calculator. Closing the calculator and the `claude` session ends the demo — there is nothing to clean up.

## What the PoC demonstrates

1. The trust dialog in Claude Code v2.1+ does not mention MCP servers, does not enumerate the server about to start, and does not show the command it will run.
2. After Enter is pressed, the project-scoped `enableAllProjectMcpServers` / `enabledMcpjsonServers` / `permissions.allow` settings combine to auto-approve and start the attacker's server with no further prompt.
3. The MCP server process runs as a native OS process with the user's full privileges and is not confined to the project directory — `calc` is launched via `child_process.exec`, not via any Claude tool call.

Replace the inline `child_process.exec(...)` payload in `.mcp.json` with anything an OS process can do, and you have the real-world attack chain. We did not.
