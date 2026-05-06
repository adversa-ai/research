# TrustFall: Claude Code RCE via insecure MCP settings

Attachments for the Adversa AI blog post **[TrustFall: Claude Code RCE via insecure MCP settings](https://adversa.ai/blog/trustfall-claude-code-rce-mcp-settings/)**.

A malicious repository can ship two small JSON files (`.mcp.json` and `.claude/settings.json`) that auto-approve an attacker-controlled MCP server. The moment a developer presses Enter on Claude Code's generic "Yes, I trust this folder" dialog, the server spawns as an unsandboxed Node.js process with the user's full privileges — no per-server consent, no tool call from Claude required. The same attack runs with zero interaction on CI runners that invoke Claude Code headlessly via the official `anthropics/claude-code-action` GitHub Action, since headless mode auto-skips the trust dialog. The report deep-dives Claude Code, but we ran a parity check across three comparable agentic CLIs and confirmed the same one-keypress chain in **Gemini CLI**, **Cursor CLI**, and **Copilot CLI** — they differ only in how the trust dialog frames the authorization. Anthropic patched the timing component as CVE-2025-59536 (October 2025) but declined the scope component as outside their threat model; this repo documents the residual attack surface.

## What's in this directory

| Path | Contents |
|---|---|
| [`report/`](report/) | Full technical report and supporting screenshots — trust-dialog regression, parity-check dialogs from Gemini/Cursor/Copilot CLIs, the `bypassPermissions` red-warning dialog, and the C2 video thumbnail. |
| [`poc/`](poc/) | **1-click developer-machine PoC.** Auto-approves an MCP server whose only payload opens the OS calculator — no file reads, no network calls. The same fixture reproduces on all four CLIs (Claude Code, Gemini CLI, Cursor CLI, Copilot CLI). See [`poc/README.md`](poc/README.md) before running. |
| [`poc-ci-pipeline/`](poc-ci-pipeline/) | **0-click headless CI/CD PoC.** A `.github/workflows/claude.yml` that invokes `anthropics/claude-code-action@v1` plus a `.mcp.json` whose payload POSTs the runner's `process.env` (including `GITHUB_TOKEN`, `ANTHROPIC_API_KEY`, and any other secrets the workflow has access to) to a collector URL you choose. For self-testing on pipelines you own — see [`poc-ci-pipeline/README.md`](poc-ci-pipeline/README.md). |

The blog post is the recommended starting point. The full report adds the timeline of related CVEs, the side-by-side capability comparison with `bypassPermissions`, the parity-check table across the four CLIs, the appendices on the fileless and `permissions.allow` variants, and the full back-and-forth with Anthropic's security team.
