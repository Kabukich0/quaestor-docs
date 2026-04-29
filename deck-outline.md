# Quaestor — Seed Deck v0 (slide outline)

This is the slide-by-slide outline a designer will turn into the actual deck.
Each slide names its purpose, lists the 3–5 bullets it must land, and points
to the supporting artifact in the repo (or flags where production data is
still missing).

Theme (per game plan): **Federal Reserve meets Stripe** — off-white
background, oxblood accent, serif headlines, monospace code. Restrained.
Skim-readable in 90 seconds, deep on demand.

---

## 1. Title

**"Quaestor: The Mandate & Audit Layer for the Machine Economy"**

- **Purpose:** Anchor the category before the audience invents one for us.
- Subtitle (one line): "Sovereign agent payments. Keys never leave the device. Five protocols, one mandate."
- Founder name + role (Connor Hearts, founder)
- Contact + date in footer
- One QR code linking to the MCP demo (when the video exists)
- **Artifact:** none — title slide.

## 2. The 402 Problem

**Purpose:** Make the audience feel agent-payment friction before we sell the cure.

- HTTP 402 Payment Required has gone from a 25-year-old curiosity to the load-bearing handshake for AI agents.
- Four competing wire formats live in production today: `WWW-Authenticate: x402`, `Authorization: Payment scheme=mpp`, `WWW-Authenticate: A2A extension="ap2"`, `WWW-Authenticate: ACP`.
- Every agent that pays anything has to route between them by hand — and decide who holds the keys.
- The unsexy infra layer (the part that turns "the agent has authority" into "the merchant gets paid") is currently nobody's problem. Until it's a breach.
- **Artifact:** `protocol-bridge.md` — header table at top of doc.

## 3. The protocol explosion

**Purpose:** Show this isn't one protocol winning, it's five protocols all shipping in 12 months.

- **MPP** (Stripe + Tempo, March 2026) — Authorization-header payment scheme over fiat rails.
- **x402 v2** (Coinbase, December 2025) — base64url envelope on `X-PAYMENT` for stablecoin settlement.
- **AP2** (Google, late 2025) — A2A extension wrapping x402 inside an agent-to-agent envelope.
- **ACP** (Stripe + OpenAI, late 2025) — Agentic Commerce Protocol for marketplace flows.
- **MCP** (Anthropic, 2024 → mainstream 2026) — the tool surface every agent already speaks.
- The bet is *integration*, not *picking a winner* — and that's exactly what Quaestor sells.
- **Artifact:** `protocol-bridge.md` — full protocol-by-protocol breakdown.

## 4. The trust-model gap

**Purpose:** Establish that every funded competitor solves the *protocol* problem the wrong way (custodial), and that's an unfixable cap-table problem for them.

- Crossmint, Skyfire, Nekuda, Coinbase Agentic Wallets — all ship with the user's keys on their server.
- That's not a bug they can patch out. It's their entire business model: take-rate per transaction, KYC moat, custody float.
- The moment one of them gets breached, the whole category becomes "sovereign or nothing."
- Quaestor's structural position is the only one a security-conscious enterprise will sign off on.
- **Artifact:** `competitive-matrix.md` — at-a-glance table + the "Where Quaestor's structural position is genuinely defensible" section.

## 5. Quaestor — Sovereign by design

**Purpose:** The product, in 30 seconds.

- **Keys never leave the device.** Ed25519 HD keys live in the OS keychain. Mandates are JWTs the device signs.
- **Hash-chained ledger.** Every issuance, redemption, revocation is a BLAKE3-chained row. Tamper-evident, replayable.
- **Five protocols, one mandate.** Same JWT pays an MPP merchant, an x402 merchant, an AP2 agent, an ACP marketplace.
- **Open core.** The daemon and bridge are MIT. The audit-grade, multi-machine, and compliance pieces are paid SaaS.
- **Local-first.** No mandatory cloud. Loopback-only HTTP. The agent process never holds a key, even in memory.
- **Artifact:** `threat-model.md` (trust boundaries section), `quaestor-core/README.md` (architecture diagram).

## 6. Architecture — the flow

**Purpose:** Show the system on one page so a technical reviewer can verify the claims before the meeting ends.

- Agent (Claude Code, Cursor, Cline) → MCP tool call.
- `quaestor-mcp` server → forwards to `quaestor-core` daemon over loopback.
- `quaestor-bridge` translates the redeemed credential into the protocol-native header (MPP / x402 / AP2 / ACP).
- `quaestor-core` daemon: vault (keychain) + signer + BLAKE3 ledger + policy engine.
- Mandate JWT round-trips through the protocols; the private key never moves.
- **Artifact:** `quaestor-bridge/README.md` mermaid (the agent → bridge → core → vault+ledger flow).

## 7. Why now

**Purpose:** Anchor the timing — this is a 12-month window, not a 5-year one.

- **December 2025:** Coinbase ships x402 v2 — first credible standard with mainnet rails.
- **March 2026:** Stripe + Tempo ship MPP — first credible Authorization-header scheme on fiat.
- **Late 2025:** Google launches AP2 wrapping x402 inside A2A — first agent-to-agent payment envelope.
- **Now (Q2 2026):** Every framework (LangChain, AutoGen, OpenAI Assistants) is shipping payment hooks. None is shipping a *vault*.
- The protocols converged faster than the trust layer could. We're the trust layer.
- **Artifact:** `protocol-bridge.md` — release-date references; primary sources cited inline.

## 8. Wedge — the DevOps Agent Wallet

**Purpose:** Show we have a beachhead, not an addressable-market chart.

- First customer is the AI coding agent, not a human user.
- Claude Code / Cursor / Cline already speaks MCP. We register six tools (`request_mandate`, `pay_with_mandate`, `query_ledger`, `get_balance`, `revoke_mandate`, `list_active_mandates`) and the agent has a wallet.
- The agent now buys the things it already needs to buy: GPU credits, Vercel deploys, Stripe-hosted APIs, Coinbase x402 endpoints.
- This is the wedge: a developer types `/install quaestor-mcp` and an agent that can't be embezzled is online.
- From there: every other agent shape (browser, support, ops) is the same architecture.
- **Artifact:** `quaestor-mcp/examples/programmatic-mcp-client.ts` — runs all six tools end-to-end.

## 9. Traction

**Purpose:** Honest snapshot. Avoid the temptation to dress up a pre-launch repo.

- _Needs production data — fill in before the deck ships:_
  - GitHub stars (today: ~0; check `gh repo view PawCheck1/quaestor-core --json stargazerCount`)
  - MCP installs / Claude Code skill installs
  - Paying users (today: 0)
  - Discord members
  - Demo video views
- What we *do* have on Day Zero: three repos shipped, 36-hour Phase-1 cycle, three protocols (MPP / x402 / AP2) end-to-end through a live core daemon, 43 vitest tests on mcp + 36 on bridge passing.
- **Artifact:** `quaestor-core/STATUS.md`, `quaestor-bridge/STATUS.md`, `quaestor-mcp/STATUS.md`.

## 10. Competition + Moat

**Purpose:** Pre-empt the "who else is doing this" question and convert it into a moat slide.

- **Custodial wallets** (Crossmint, Skyfire, Nekuda, Coinbase Agentic Wallets): take-rate model, can't pivot to sovereignty without breaking their unit economics.
- **Bring-your-own-key SDKs**: protocol-specific, nobody covers MPP + x402 + AP2 + ACP from one mandate.
- **Quaestor's moat:**
  1. Local-first sovereignty — structurally hard to copy without a cap-table reset.
  2. Protocol-agnostic — five wire formats from one credential issuance.
  3. Open-source core — defensive against a Stripe / Coinbase clone.
  4. Audit IP — BLAKE3-chained ledger as a paid product (Audit, Compliance) is the upside on the open core.
- **Artifact:** `competitive-matrix.md` — "Where Quaestor's structural position is genuinely defensible" section.

## 11. Business model

**Purpose:** Show how this becomes a real company without a take-rate.

- **Open core:** `quaestor-core` + `quaestor-bridge` + `quaestor-mcp` — MIT, free forever.
- **Paid tiers (SaaS, no transaction fees):**
  - **Cloud sync — $29/mo** — multi-machine mandate sync (Iroh P2P backend).
  - **Audit — $79/mo** — exportable hash-chained audit logs, retention, SOC2-ready exports.
  - **Compliance — $499/mo per seat** — KYB attestation, regulatory adapters, dispute tooling.
- **Why not take-rate:** sovereign keys + take-rate is incoherent. Custodial wallets do take-rate because they hold the float. We don't, by design.
- **Artifact:** game plan doc (internal).

## 12. Team + Ask

**Purpose:** Make the ask explicit and defensible.

- **Founder:** Connor Hearts — solo technical founder. Three repos shipped end-to-end in 36 hours. Background in systems / cryptography / agent infra.
- **No co-founder yet.** Looking for a protocols-and-cryptography-savvy CTO post-seed.
- **First hires (12-month plan):**
  1. Protocols engineer (x402 / MPP / AP2 / ACP wire-format depth)
  2. Cryptography engineer (HD-key + multi-device)
  3. Founding GTM — developer-relations-shaped, not enterprise-sales-shaped
- **Ask:** $1.5–2.5M seed @ $10–14M post, pre-traction. 18 months runway.
- **Use of funds:** ~70% headcount (3 hires × 12 months), ~20% infra + GTM, ~10% legal + reserve.
- **Artifact:** none — slide is the slide.

---

## Visual artifacts needed

What the deck designer will need before the visual pass:

- **Architecture mermaid** — already in `quaestor-core/README.md` and `quaestor-bridge/README.md`. Exportable to PNG via mermaid-cli; pick the bridge's version (has the agent → bridge → core flow most clearly).
- **Competitive matrix table** — extract the at-a-glance table from `competitive-matrix.md`. Trim to top 6 competitors + Quaestor; one row per competitor; columns: name / custody model / protocol coverage / open source / per-tx fee.
- **Two demo videos:**
  - Bridge demo: `~/code/quaestor-bridge/docs/demo.mp4` (already recorded, screen-recordable, ~30 s).
  - MCP demo: **not yet recorded.** Highest-leverage missing asset for fundraising. (See `TOMORROW.md`.)
- **Logo / wordmark — not yet designed.** Workshop with a designer; theme is "Federal Reserve meets Stripe" so think Cooper Hewitt + a serif display face like GT Sectra or Tiempos. Oxblood accent, off-white background.
- **Theme stylesheet** — single page documenting the colors, type, and grid so every slide stays consistent. Off-white #F8F4EC background, oxblood #6E0E0A accent, ink #1A1A1A type, Berkeley Mono for code, GT Sectra (or similar) for headlines.

---

## Risks honestly disclosed

These show up in the deck explicitly — diligence reveals them anyway, and disclosing first builds trust.

- **Solo founder.** No co-founder yet. Hire dependency for technical depth in protocols + cryptography.
- **v0.1.1 alpha.** Three repos shipped, no production deployments yet. Real users = zero on Day Zero.
- **Real-network testnet integration not yet validated against a real x402 facilitator.** Bridge uses the in-house wire-format builders; we have not posted a live envelope to a Coinbase/Stripe facilitator and watched it accept. Mock 402 fixtures only.
- **`mppx@0.6.7` and `x402@1.2.0` ship empty `dist/` on npm.** Bridge auto-falls back to in-house builders. Correct on the wire; not exercising the published SDKs. Tracked in bridge `STATUS.md → Spec drift log`.
- **No mandate revocation endpoint yet.** `revoke_mandate` returns a documented 404 until core ships POST `/mandate/revoke` (Phase 1.4).
- **No multi-machine sync yet.** Iroh P2P integration is on the roadmap but not built. Single-device today.
- **Open-core monetization is a bet.** Hash-chained audit logs as a paid product is unproven; could end up looking like Sentry's audit add-on rather than a standalone tier.
- **Solo founder + alpha + open core** is what makes the round actually fundable at $10–14M post — anyone willing to underwrite *now* is buying the bet that the trust layer is structurally won early. Anyone wanting traction should pass.
