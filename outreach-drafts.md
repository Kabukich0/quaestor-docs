# Quaestor — Cold-Outreach Drafts

Three first-touch templates. Each is under 150 words, references one (and
only one) artifact, and ends with a concrete ask. Don't fabricate metrics
— if a number isn't real, the line gets cut.

Personalize before sending: the first line of every draft should reference
something the recipient has shipped, said, or written publicly in the last
60 days. The version below gives you the structure; the personalization
line is your job.

---

## Draft A — Tier-1 VC (Conviction / Sarah Guo style)

**Subject:** the trust gap in agent payments

Hi Sarah,

[Personalization line — reference a specific recent post or talk on agent
infra; one sentence.]

Every funded agent wallet shipping today (Crossmint, Skyfire, Nekuda,
Coinbase Agentic Wallets) is custodial. They hold the user's keys. That
isn't a bug they can patch out — it's the whole business model, take-rate
on float plus KYC moat. The first breach in this category collapses the
whole approach.

I've been building Quaestor — sovereign keys (never leave the device),
hash-chained ledger, five protocols (MPP / x402 / AP2 / ACP / MCP) from one
mandate JWT. Open core, MIT, MCP-native. Phase 1 shipped this week.

Demo: https://github.com/Kabukich0/quaestor-mcp/blob/main/examples/programmatic-mcp-client.ts
(in-process MCP server drives all 6 tools end-to-end against a live core
daemon — exit 0 in ~2 seconds).

20 minutes this week or next? Or if it's not a fit — what would change
your mind?

— Kabukich0

---

## Draft B — Strategic angel (Coinbase / Stripe / Anthropic engineer)

**Subject:** five payment protocols, one mandate JWT

Hi [Name],

[Personalization line — reference something they've shipped or commented on
publicly: "Saw your x402 v2 thread on the envelope shape" / "Your MPP
session-mode write-up was the cleanest I've read"; one sentence.]

I built a bridge that takes one Quaestor-issued mandate JWT and translates
it into the protocol-native payment header for MPP, x402, and AP2 from a
single redeem call. The bridge holds no keys — every payment round-trips
through a local daemon (loopback only) that signs with an Ed25519 HD key
in the OS keychain. Hash-chained ledger underneath for audit.

The technical claim that's hard to make work: AP2's settlement delegates
to x402's adapter rather than reimplementing — verified by a `vi.spyOn`
test in CI.

Repo: https://github.com/Kabukich0/quaestor-bridge

Open to your read on the wire format. 15 minutes? Happy to demo against a
local merchant on Zoom.

— Kabukich0

---

## Draft C — Canadian fund / pre-seed (Inovia / Real Ventures style)

**Subject:** sovereign agent wallet — Phase 1 shipped in 36 hrs

Hi [Name],

[Personalization line — reference a specific portfolio company or thesis post;
one sentence.]

Quick context: I'm a solo founder building Quaestor — the trust &
audit layer for AI agent payments. Local-first sovereign keys, five
protocols (MPP / x402 / AP2 / ACP / MCP), open-core (MIT) + paid SaaS for
multi-machine sync, audit, and compliance.

Phase 1 — four repos (`quaestor-core`, `quaestor-bridge`, `quaestor-mcp`,
`quaestor-docs`) end-to-end with green CI — shipped in a 36-hour focused
build cycle this week. Test suites, integration demos, full STATUS.md per
repo. Solo founder shipping at exceptional velocity; not seeking a
co-founder for the round; first hires (protocols engineer, cryptography
engineer, founding GTM) planned post-funding.

Phase 1.3 status doc:
https://github.com/Kabukich0/quaestor-mcp/blob/main/STATUS.md

Raising $1.5–2.5M seed at $10–14M post. 20-min intro call this week or next?

— Kabukich0

---

## Discipline notes

- **One link per email.** A second link costs a reply. The recipient will
  click zero or one — make it count.
- **Don't fabricate.** No fake user counts, no inflated stars, no "we've
  been talking to..." unless we have. Investors compare notes.
- **Personalize the opener.** The structure above is reusable; the first
  sentence is not. Pull from their recent X posts, a podcast, a recent
  portfolio announcement. Show you read.
- **Keep follow-ups shorter than the original.** Day 4 follow-up: two
  sentences max. Day 11: one sentence. After that, stop — silence is an
  answer.
- **Reply rate target:** ~25% on Tier-1 cold (with personalization), ~40%
  on warm intros, ~10% on Canadian-fund cold. Below those bars, the email
  is broken — rewrite, don't push harder.
