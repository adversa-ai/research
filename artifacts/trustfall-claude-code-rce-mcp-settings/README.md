# TrustFall: Claude Code RCE via insecure MCP settings

Attachments for the Adversa AI blog post **[TrustFall: Claude Code RCE via insecure MCP settings](https://adversa.ai/blog/trustfall-claude-code-rce-mcp-settings/)**.

A malicious repository can ship two small JSON files (`.mcp.json` and `.claude/settings.json`) that auto-approve an attacker-controlled MCP server. The moment a developer presses Enter on Claude Code's generic "Yes, I trust this folder" dialog, the server spawns as an unsandboxed Node.js process with the user's full privileges — no per-server consent, no tool call from Claude required. The same attack runs with zero interaction on CI runners that pass `--trust-folder`. Anthropic patched the timing component as CVE-2025-59536 (October 2025) but declined the scope component as outside their threat model; this repo documents the residual attack surface.

## What's in this directory

| Path | Contents |
|---|---|
| [`report/report.md`](report/report.md) | Full technical report — for readers who want the full disclosure detail beyond the blog narrative. |
| [`report/screenshots/`](report/screenshots/) | Trust-dialog regression screenshots, the `bypassPermissions` red-warning screenshot, and the C2 video thumbnail referenced from both the report and the blog post. |
| [`poc/`](poc/) | Safe, reproducible proof-of-concept. Opens the OS calculator after the trust dialog is accepted — no file reads, no network calls. See [`poc/README.md`](poc/README.md) before running. |

The blog post is the recommended starting point. The full report adds the timeline of related CVEs, the side-by-side capability comparison with `bypassPermissions`, the appendices on the fileless and `permissions.allow` variants, and the full back-and-forth with Anthropic's security team.
