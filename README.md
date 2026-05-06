# Adversa AI Research

Public attachments for [Adversa AI](https://adversa.ai/) security research and disclosures. Each subdirectory under `artifacts/` corresponds to a single published advisory or article and contains the full technical report, proof-of-concept code, and supporting materials.

This repository is intended for security practitioners who want the technical detail behind a public Adversa AI write-up — readers who only need the narrative version should start with the linked blog post inside each artifact.

## Layout

```
artifacts/
└── <article-slug>/
    ├── README.md      # Short index: what this advisory is and what's in the directory
    ├── report/        # Full technical report and screenshots
    └── poc/           # Safe, reproducible proof-of-concept (where applicable)
```

## Published research

| Artifact | Topic | Blog post |
|---|---|---|
| [`artifacts/trustfall-mcp-settings-rce/`](artifacts/trustfall-mcp-settings-rce/) | Project-scoped MCP auto-approval enables 1-click RCE across four agentic CLIs (Claude Code, Gemini CLI, Cursor CLI, Copilot CLI); 0-click in Claude Code CI via `claude-code-action`. Claude Code deep-dive — incomplete CVE-2025-59536 fix. | [TrustFall: Claude Code RCE via insecure MCP settings](https://adversa.ai/blog/trustfall-claude-code-rce-mcp-settings/) |

## Disclosure posture

All findings published here have been reported to the affected vendor first. Where a vendor has declined a finding as out-of-scope of their threat model, the corresponding write-up makes the vendor's position explicit and documents the residual risk for defenders.

Proof-of-concept code is deliberately scoped to the minimum that demonstrates the vulnerability. PoCs intended for general execution on a developer machine do not exfiltrate data, open network connections, or modify anything outside the working directory. A small number of PoCs are scoped to **self-testing on infrastructure the reader owns** — for example, a CI fixture that requires the reader to point an exfiltration target at their own collector URL on a repository they control; nothing leaves the runner until the reader supplies that URL. These are flagged in their per-artifact README and require explicit setup steps. Read the per-artifact README before running anything.

## Contact

Security research and disclosure inquiries: [https://adversa.ai/](https://adversa.ai/)
