# Competitive matrix

> **Honesty notes.** This matrix is authored by the Quaestor team. We tried not to puff our own column. Where we are not sure of a competitor's funding, custody model, or feature set as of 2026-04-29, the cell is marked `?` — we would rather be wrong-flagged-as-uncertain than wrong-while-asserting-certainty. Where Quaestor is *weaker* than a competitor on a given dimension (e.g. funding, distribution, productionization), we say so explicitly in the per-row "Known weakness vs Quaestor" column or the closing section.

## At a glance

| Project | Custodial? | Open source? | Local-first? | Multi-protocol (MPP / x402 / AP2)? | Audit ledger? | Funding round | Known weakness vs Quaestor |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Quaestor** (this project) | No — keys never leave device | Yes (MIT) | Yes — daemon binds `127.0.0.1` only | Yes — all three; AP2 composes over x402 | Yes — BLAKE3-chained SQLite WAL, `quaestor ledger verify` | None (alpha, v0.1.1) | n/a (see "Where Quaestor is weaker", below) |
| **Nekuda** | ? — likely SaaS-custodied; "Agentic Mandates" hosted | ? (no public OSS repo identified) | No — cloud service | ? — markets "mandates"; multi-rail not confirmed | ? — provider-side log; not user-verifiable chain | $5M seed (Madrona, Amex Ventures, Visa Ventures) | Custodian compromise = customer compromise; mandate semantics opaque to user; depends on Nekuda staying online |
| **Crossmint** | Yes by default (custodial-as-a-service); non-custodial smart-wallet option | Partial (SDKs open; backend infra closed) | No — managed wallet APIs | ? — web3-first; MPP and AP2 not confirmed | ? — provider dashboard; not cryptographic chain | ? (raised through 2024; latest round size not confirmed by us as of 2026-04-29) | Custodial default; web3-rail focus; not designed to span IETF MPP + x402 + AP2 in one credential |
| **Skyfire** | Yes — managed agent-payments wallet | ? | No | ? — has own rails for agent payments; multi-protocol parity not confirmed | ? | ? (publicly funded; round sizes not verified by us) | Custodial; provider-dependent settlement; key material outside user's host |
| **Coinbase Agentic Wallets** | Yes — Coinbase MPC custody | Partial (CDP SDK is OSS; custody infra is not) | No — keys on Coinbase infrastructure | x402 native (theirs); MPP and AP2 not native | Coinbase reporting; not a user-walkable chain | N/A — product of Coinbase (NASDAQ: COIN). Distinct from Coinbase Ventures, which invests in this category. | Centralized custodian; single-vendor risk; KYC/regulatory exposure on the funds themselves |
| **Basis Theory** | Yes — custodial vault for sensitive data (PCI tokenization) | ? — closed core | No | No — primary use is card-data tokenization, not agent mandates | Provider-side audit log; not cryptographic chain | ? (Series A reported; current size not verified by us) | Different focus (PCI tokens), not agent payment mandates; comparable mostly as a "vaulted secrets in someone else's cloud" alternative |
| **Keyban** | ? — wallet infrastructure; custody model not confirmed (likely MPC) | ? | ? — MPC schemes can mix device + cloud shares | ? | ? | €500K pre-seed | Earlier stage; feature parity with Quaestor's mandate / ledger primitives not demonstrated publicly as of 2026-04-29 |
| **Nevermined** | No — web3-native registry of priced AI services; payments are on-chain | Yes (most of it) | Partial — agent runs locally but settlement is on-chain | No — Nevermined's own payment paradigm; not an MPP / x402 / AP2 adapter | Yes — the chain itself | ? (raised seed in the 2021–2022 wave; latest round not verified by us) | Crypto-native only — no path to settle through Stripe / IETF MPP / Tempo through one mandate |

## Where Quaestor is weaker today

Pretending a v0.1.1 alpha is on parity with $5M-funded competitors is exactly the puffery this matrix is trying to avoid. Concrete weaknesses:

- **No production deployments.** Quaestor has not settled a single dollar through a real merchant in production. The demo runs against fixture 402s, not live facilitators.
- **No funding, no team beyond a single contributor.** Competitors with seed rounds have runway and account-management muscle Quaestor does not.
- **Upstream receipt verification is scaffolded, not enforced.** `POST /ledger/receipt` records what the bridge reports; it does not re-query the chain or facilitator to confirm `tx_hash`. (Listed as a Known gap in [`quaestor-core/STATUS.md`](https://github.com/PawCheck1/quaestor-core/blob/main/STATUS.md#known-gaps); v0.2 target.)
- **No mandate revocation endpoint.** Today the only way to invalidate a mandate is to let it expire or exhaust its `use_counter_max`. Competitors with a hosted control plane can revoke instantly.
- **No vault rotation or recovery flow.** Losing the OS-keychain KEK = losing the vault. Custodial competitors recover users via their support flow.
- **`mppx@0.6.7` and `x402@1.2.0` ship empty `dist/` on npm.** Quaestor's adapters fall back to in-house IETF/v2 builders, which are correct on paper but not yet validated by a real facilitator round-trip. (See [`protocol-bridge.md → SDK pinning`](./protocol-bridge.md).)
- **No multi-process / multi-machine story.** Single daemon, single device. Custodians can offer cross-device wallets out of the box.

## Where Quaestor's structural position is genuinely defensible

What the matrix above is meant to show, calibrated against the weaknesses:

- **Custody model.** Quaestor is the only row in the matrix where the answer to "what happens if the provider gets hacked" is "there is no provider." See [`threat-model.md`](./threat-model.md).
- **Multi-protocol surface.** Three live adapters (MPP, x402 v2, AP2-over-x402) from one mandate is, to our knowledge, unique. Competitors specialize in one rail. See [`protocol-bridge.md`](./protocol-bridge.md).
- **User-walkable audit chain.** A BLAKE3-chained SQLite WAL ledger that any user can `verify` from genesis is not the same primitive as a provider dashboard.
- **Open source.** MIT-licensed and inspectable end-to-end. The mandate semantics, vault encryption, and ledger format are all auditable code, not blog-post claims.

## Sources / verification notes

- **Nekuda funding** ($5M seed; Madrona, Amex Ventures, Visa Ventures): provided to the Quaestor team as context, not independently re-verified for this document. If the round size or syndicate is wrong, this cell should be corrected.
- **Keyban funding** (€500K pre-seed): same as above — provided as context.
- All other competitor cells: cross-checked against publicly known positioning as of 2026-04-29; uncertain entries marked `?` rather than guessed.

If you are reviewing this matrix and a cell is wrong, please open an issue on `PawCheck1/quaestor-docs` — being right matters more here than looking right.
