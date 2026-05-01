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
| [`artifacts/trustfall-claude-code-rce-mcp-settings/`](artifacts/trustfall-claude-code-rce-mcp-settings/) | Claude Code 1-click RCE via project-scoped MCP auto-approval (incomplete CVE-2025-59536 fix) | [TrustFall: Claude Code RCE via insecure MCP settings](https://adversa.ai/blog/trustfall-claude-code-rce-mcp-settings/) |

## Disclosure posture

All findings published here have been reported to the affected vendor first. Where a vendor has declined a finding as out-of-scope of their threat model, the corresponding write-up makes the vendor's position explicit and documents the residual risk for defenders. Proof-of-concept code is deliberately neutered: no PoC in this repo exfiltrates data, opens a network connection, or modifies anything outside the working directory. Read the per-artifact README before running anything.

## Contact

Security research and disclosure inquiries: [https://adversa.ai/](https://adversa.ai/)
