# quaestor-docs

Internal docs, handoffs, deck outlines, and threat models for the
Quaestor protocol stack.

## Repos

| Repo | Purpose |
|---|---|
| [`quaestor-core`](https://github.com/Kabukich0/quaestor-core) | Local daemon: vault, mandate signing, ledger, sign-x402 / sign-ap2 / sign-mppx, JWKS |
| [`quaestor-bridge`](https://github.com/Kabukich0/quaestor-bridge) | Protocol translation: x402 / AP2 / MPP-X adapters via core-signs |
| [`quaestor-mcp`](https://github.com/Kabukich0/quaestor-mcp) | MCP server exposing mandate + payment + ledger surface to Claude Desktop, Cursor, Cline |
| [`quaestor-policy`](https://github.com/Kabukich0/quaestor-policy) | Local policy LLM plugin (Phase 2.0) — intent enforcement via on-device Qwen 2.5 3B |
| [`quaestor-demos`](https://github.com/Kabukich0/quaestor-demos) | Phase walkthroughs. Latest: [v0.2.0](https://github.com/Kabukich0/quaestor-demos/releases/tag/v0.2.0) |

## Active handoffs

- [`HANDOFF-1.5b.md`](HANDOFF-1.5b.md) — Phase 1.5a → 1.5b carryover (now closed)
- [`HANDOFF-2.0.md`](HANDOFF-2.0.md) — Phase 1.5b → 2.0 carryover (active; signMandate + policy LLM)
- [`TOMORROW.md`](TOMORROW.md) — chronological work log

## Reference

- [`deck-outline.md`](deck-outline.md) — investor pitch structure
- [`competitive-matrix.md`](competitive-matrix.md) — landscape table
- [`investor-pipeline.md`](investor-pipeline.md) — outreach status
- [`outreach-drafts.md`](outreach-drafts.md) — cold-email templates
- [`protocol-bridge.md`](protocol-bridge.md) — x402 / AP2 / MPP-X protocol notes
- [`threat-model.md`](threat-model.md) — current trust-boundary analysis
