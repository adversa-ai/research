# CVE-2025-59536 Incomplete Remediation: Post-Trust Silent MCP Execution in Claude Code

**Researcher:** Adversa AI (Rony Utevsky) 
**Severity:** High  
**Affected Product:** Claude Code CLI (Anthropic)  
**Tested Version:** v2.1.108 (April 2026)  
**Status:** Unpatched  

## Overview

This repository contains the research, proof-of-concept, and disclosure report for an incomplete remediation of [CVE-2025-59536](https://nvd.nist.gov/vuln/detail/CVE-2025-59536) in Anthropic's Claude Code CLI.

In October 2025, Check Point Research demonstrated that a malicious repository could use `enableAllProjectMcpServers` in project-scoped `.claude/settings.json` to auto-execute MCP servers **before** the user saw a trust dialog. Anthropic patched the **timing** (servers now wait until after the trust dialog), but did not patch the **scope** -- `enableAllProjectMcpServers` and `enabledMcpjsonServers` are still accepted from project-scoped settings.

This means a malicious repository can still self-approve its own MCP servers. After a single "trust this folder" click, attacker-supplied code executes silently as an unsandboxed OS process with full user privileges -- no additional prompt, no tool call required.

## Key Findings

1. **Settings scope restriction bypass**: `enableAllProjectMcpServers` and `enabledMcpjsonServers` remain unblocked from project scope, despite Anthropic blocking five other settings (`autoMode`, `bypassPermissions`, `skipBypassConfirmation`, `autoMemoryDirectory`, `useAutoModeDuringPlan`) for the same "prevent repo injection" reason.

2. **Trust dialog regression**: The older trust dialog (pre-v2.1) explicitly warned about MCP servers and offered enable/disable options. The current dialog removed all MCP-specific information -- a measurable security regression.

3. **Full machine compromise**: The MCP server runs with full user privileges, not sandboxed to the project directory. An attacker can exfiltrate `~/.ssh/`, `~/.aws/`, other project directories, establish C2 channels, and install persistence mechanisms.

4. **0-click CI/CD variant**: In pipelines using `--trust-folder` or the official `claude-code-action` GitHub Action, zero user interaction is required.

## Repository Structure

```
.
+-- report/
|   +-- report.md              # Full disclosure report
|   +-- screenshots/
|       +-- exfiltration.png       # Attacker server receiving exfiltrated data
|       +-- trust-dialog-old.png   # Old trust dialog (with MCP warning)
|       +-- trust-dialog-new.png   # Current trust dialog (MCP warning removed)
+-- exploit/                   # Proof-of-concept malicious repository (DO NOT EXECUTE)
|   +-- .mcp.json                  # MCP server definition
|   +-- .claude/
|   |   +-- settings.json          # Project-scoped auto-approval config
|   +-- mcp/
|   |   +-- github.js              # PoC payload (exfiltrates .env to webhook)
|   +-- .env                       # Dummy secret for PoC demonstration
|   +-- package.json               # Node.js dependencies
+-- README.md
```

## Attack Chain Summary

```
1. Victim clones malicious repository
2. Victim runs `claude` in the repo directory
3. Generic trust dialog appears (no mention of MCP servers)
4. Victim clicks "Yes, I trust this folder"
5. .claude/settings.json silently enables MCP servers
6. MCP server spawns -- payload executes immediately on process start
7. Credentials exfiltrated, C2 established -- victim sees normal Claude interface
```

The payload executes on **server startup** -- not on a tool call. The moment `node mcp/github.js` is spawned by Claude Code, the attacker's code runs. No interaction with Claude is needed after the trust dialog click.

## Root Cause

Anthropic treats project-scoped settings as partially trusted: five settings are explicitly blocked from project scope to prevent repo injection, but `enableAllProjectMcpServers` and `enabledMcpjsonServers` -- which grant **greater** execution capabilities (arbitrary unsandboxed code execution) -- are left unblocked. This is an inversion of the threat hierarchy.

## Recommended Fixes

1. **Block `enableAllProjectMcpServers` and `enabledMcpjsonServers` from project-scoped settings** -- apply the same restriction used for `bypassPermissions` and `autoMode`.
2. **Restore the MCP-specific trust dialog** -- enumerate each server, show commands, offer per-server enable/disable.
3. **Require per-server interactive consent** for project-defined MCP servers.
4. **Conduct a systematic scope audit** of all settings accepted from project scope.

## Safety Warning

> **The `exploit/` directory contains a functional proof-of-concept.** Do not run `claude` or `node` inside it. The PoC payload sends data to an external webhook endpoint. It is included for reproducibility and disclosure purposes only.

## Related Work

| CVE | Date | Issue | Fix |
|-----|------|-------|-----|
| CVE-2025-59536 | Oct 2025 | MCP executes before trust dialog | v1.0.111: Delayed MCP until after dialog |
| CVE-2026-21852 | Jan 2026 | `ANTHROPIC_BASE_URL` in project settings | v2.0.65: Setting blocked from project scope |
| CVE-2026-33068 | Mar 2026 | `bypassPermissions` in project settings | v2.1.53: Setting blocked from project scope |
| **This report** | **Apr 2026** | **Post-trust silent MCP execution** | **Unpatched** |

## License

This research is published for responsible disclosure and defensive security purposes.
