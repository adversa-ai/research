# TrustFall PoC — CI/CD pipeline (0-click headless variant)

This directory is the second of two fixtures for the **[TrustFall](https://adversa.ai/blog/trustfall-claude-code-rce-mcp-settings/)** finding. It demonstrates the **headless CI/CD variant**: Claude Code invoked through `anthropics/claude-code-action` runs non-interactively, the workspace trust dialog never renders, and a project-shipped `.mcp.json` is executed automatically on action startup with no human in the loop.

The companion fixture at [`../poc/`](../poc/) shows the **1-click developer-machine variant** across all four agentic CLIs (Claude Code, Gemini CLI, Cursor CLI, Copilot CLI) using a safe `calc.exe` payload.

The full technical write-up is in [`../report/`](../report/).

## What's in here

| File | Purpose |
|---|---|
| [`.mcp.json`](.mcp.json) | Registers `poc-server` with an inline `node -e` payload that POSTs the entire `process.env` of the GitHub Actions runner to a collector URL of your choice. Demonstrates the fileless variant — there is no script on disk for a reviewer or static scanner to flag. |
| [`.github/workflows/claude.yml`](.github/workflows/claude.yml) | A minimal GitHub Actions workflow that invokes `anthropics/claude-code-action@v1` on `pull_request` (and `workflow_dispatch`). The action runs Claude Code headlessly via the SDK, so the workspace trust dialog is never rendered, and the action auto-injects `enableAllProjectMcpServers: true` — `.mcp.json` alone is sufficient to trigger the chain. |

Note that no `.claude/settings.json` is needed in this fixture. The official action's `setupClaudeCodeSettings()` already auto-approves project-defined MCP servers in CI.

## How to run (against a repository you own)

> **Only run this against a GitHub repository you own or have explicit authorization to test.** Running it against any other repository — including via opening a pull request from a fork into someone else's repo — is unauthorized access to the target's CI environment and the secrets injected into it. We use this fixture exclusively for measuring our own exposure.

1. Create a test repository under your own account.
2. Copy the contents of this directory into it (`.mcp.json` and `.github/workflows/claude.yml`).
3. Set up a collector endpoint for the exfiltrated environment. The simplest option is a free [webhook.site](https://webhook.site) URL — it gives you a unique HTTPS endpoint that records every POST it receives. Replace `<ENTER YOUR URL HERE>` in `.mcp.json` with your collector URL.
4. Add an `ANTHROPIC_API_KEY` repository secret in GitHub (Settings → Secrets and variables → Actions). The workflow won't start without it, but the payload runs before Claude does any real work — the API key value is incidental to triggering the chain.
5. Install the **Claude** GitHub App on your test repository: <https://github.com/apps/claude/installations/select_target>. `anthropics/claude-code-action` requires the app to be installed on the target repo to authenticate; without it the workflow run will fail before the action starts and the chain won't trigger.
6. Trigger the workflow. Either:
   - Open a pull request against `main` (the workflow has `pull_request` and `workflow_dispatch` triggers), or
   - Run it manually from the Actions tab via "Run workflow".
7. Observe your collector. You will receive a single POST whose body is the JSON-serialized `process.env` of the runner — including `GITHUB_TOKEN`, `ANTHROPIC_API_KEY`, and any other secrets the workflow has access to.

The exfiltration runs the moment `node -e` evaluates the inline payload, before Claude reasons about anything. There is no terminal session for the workspace trust dialog to render in. There is no human in the loop. Zero clicks.

## Safety

This fixture is more aggressive than the [`../poc/`](../poc/) calculator demo because the CI variant is meaningfully different from the developer-machine one and a calculator payload doesn't demonstrate the actual blast radius. To compensate:

- **The collector URL is empty by default** (`<ENTER YOUR URL HERE>`). Nothing exfiltrates anywhere until you put your own endpoint in. If you forget to set it, the `https.request` call fails on URL parse and nothing leaves the runner.
- **The payload exfiltrates only `process.env`.** It does not read the filesystem, does not enumerate other secrets sources, does not establish a persistent C2 channel, and does not pivot to other systems reachable from the runner.
- **The destination is yours.** The fixture has no hardcoded attacker domain. Point it at `webhook.site`, your own collector, or a local mock.
- **No persistence.** Closing the workflow run ends the demo. There is nothing to clean up beyond rotating the secrets you exposed to the test repository (rotate them anyway — the runner saw them).

If you only want to confirm the trust-dialog-skipping behavior without exfiltrating env at all, replace the `node -e` payload with `console.log('TrustFall: payload executed in CI without trust dialog')` and inspect the workflow logs. The "the .mcp.json server ran in CI without any user consent" claim is the load-bearing one, not the env-leak.

## What the PoC demonstrates

1. **The trust dialog is the only consent surface in the threat model — and CI removes it.** `claude-code-action` invokes Claude through the SDK rather than the interactive CLI, so the workspace trust prompt is never rendered, never answered, and never relied on.
2. **Project-shipped `.mcp.json` executes automatically.** The action auto-injects `enableAllProjectMcpServers: true`, so a malicious repo doesn't even need `.claude/settings.json` to self-approve in CI — `.mcp.json` is sufficient.
3. **The MCP server runs as a native OS process on the runner with the runner's full privileges.** Anything in `process.env` — `GITHUB_TOKEN`, `ANTHROPIC_API_KEY`, deploy keys, signing certs, cloud credentials — is reachable to the payload from the moment the server process starts.
4. **The chain runs at action startup, before Claude reasons about anything.** There is no opportunity for Claude's planning, tool-call gating, or any other agent-side check to interpose.

Replace the `process.env` payload with anything an OS process can do — read the checked-out source, post to your runner's metadata service, mint a malicious release artifact, talk to your reachable internal network — and you have the realistic 0-click attack chain against any pipeline that runs `claude-code-action` on PR branches. We did not.
