# The Approval Prompt is Lying: Symlink-to-RCE in Agentic Coding CLIs

Attachments for the Adversa AI research paper **[The Approval Prompt Is Lying to You: Symlink RCE in Five AI Coding Agents](#)**.

A booby-trapped repository tricks your coding agent into overwriting its own configuration through a disguised file copy. The human approval step—the key safety control these tools rely on—is the exact mechanism being defeated: the user approves a harmless-looking media copy on their screen, but a pre-committed symbolic link inside the repository redirects the write straight into the agent's sensitive config folder. We tested five major tools and confirmed the vulnerability in **Claude Code**, **Gemini CLI**, **Cursor Agent CLI**, **GitHub Copilot CLI**, and **Grok Build CLI**. Anthropic quietly hardened Claude Code's approval flow to resolve and display the real target path, while other vendors declined the report or categorized it as out-of-scope.

## What's in this directory

| Path | Contents |
|---|---|
| [`poc/`](poc/) | **Ready-to-run safe PoC fixtures.** Subdirectories containing targeted environments to demonstrate silent configuration overwrites and symlink redirection across different agent CLIs: <br> • [`poc/claude/`](poc/claude/) – Targets **Claude Code** (`.claude/settings.json`, `.mcp.json`) <br> • [`poc/gemini/`](poc/gemini/) – Targets **Gemini CLI** (`.gemini/settings.json`) <br> • [`poc/cursor/`](poc/cursor/) – Targets **Cursor Agent CLI** (`~/.cursor/mcp.json`) <br> • [`poc/copilot/`](poc/copilot/) – Targets **GitHub Copilot CLI** (`.mcp.json`) <br> • [`poc/grok/`](poc/grok/) – Targets **Grok Build CLI** (`.mcp.json`) <br> • [`poc/codex/`](poc/codex/) – Targets **Codex Agent** (`.codex/config.toml`) |

The Adversa AI research paper carries the detailed architectural design critique, the step-by-step attack flow, and the back-and-forth with the security programs of all five vendors. The [`poc/`](poc/) folder provides the localized, safe fixtures to verify the findings on your own machine.
