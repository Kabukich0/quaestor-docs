# Quaestor — Investor Pipeline (v0)

Tracker template for the seed round. Every entry starts at **not contacted**.
Update the status field as conversations move; do not pre-fill anything we
haven't earned. Honest tracking is the only kind that compounds.

**Status values (in order of progression):**

`not contacted` → `warm intro identified` → `first email drafted` → `first email sent` → `first meeting` → `diligence` → `soft commit` → `term sheet` → `passed` → `wired`

A "passed" entry stays in the doc — losing tracker memory is how you re-pitch
a fund six months later and look like you weren't paying attention.

---

## A. Tier-1 lead candidates

Target: 1 of these to lead a $2–3M seed at $10–14M post.

### Paradigm

- **Partner most likely to engage:** Matt Huang (co-founder, also Tempo co-founder) or Dan Robinson (research partner, protocol-fluent).
- **Why they fit:** Paradigm incubated Tempo. They know the MPP wire format better than the people building merchants on it. Quaestor's "five protocols, one mandate" pitch lands without translation.
- **Warm-intro path:** Through anyone on the Tempo team — protocol engineers especially. Secondary: any existing Paradigm portfolio founder shipping payment infra.
- **First-email hook:** "We're building the trust layer Tempo's MPP standard assumes exists. Sovereign mandates, no custody, five protocols from one JWT. 30s demo here."
- **Status:** not contacted

### a16z crypto

- **Partner most likely to engage:** Chris Dixon (general thesis) or Ali Yahya (protocol depth + dev tools).
- **Why they fit:** They've been backing the x402 / Tempo ecosystem aggressively. Quaestor is the missing local-first piece that makes their stack defensible against custodial competitors.
- **Warm-intro path:** Through their portfolio founders — anyone shipping on x402 or building agent infra (Browserbase, Replit Agent team, etc.).
- **First-email hook:** "Custodial agent wallets are the Coinbase 2014 mistake replayed. We're shipping the local-first alternative — open-core, MIT, MCP-native. Demo + repo below."
- **Status:** not contacted

### Coinbase Ventures

- **Partner most likely to engage:** whoever owns the agent-payments thesis there (CDP devrel knows the names).
- **Why they fit:** They built x402. Quaestor expands its addressable surface (every agent that wants x402 + something else benefits). Strategic, not just financial.
- **Warm-intro path:** Through the CDP devrel team — Chris Hay's network is the obvious door. Secondary: anyone at Coinbase already publicly building on x402 v2.
- **First-email hook:** "Built the first MCP server that pays via x402 — but also MPP and AP2 from the same mandate. Want to show you the bridge before we ship it loudly."
- **Status:** not contacted

### Conviction (Sarah Guo)

- **Partner most likely to engage:** Sarah Guo herself — small fund, founder-led.
- **Why they fit:** Sarah's written publicly about agent payments and the missing trust layer. Conviction is dedicated AI infra, top-of-stack.
- **Warm-intro path:** Cold email is fine here — she answers founders directly. Bring the demo video, not the pitch deck.
- **First-email hook:** "You wrote about the agent-payments trust gap. We built it. 30-second demo: Claude Code holding a sovereign mandate and paying three protocols from one JWT."
- **Status:** not contacted

### Boldstart Ventures (Ed Sim)

- **Partner most likely to engage:** Ed Sim — infra/dev-tools-first thesis.
- **Why they fit:** Bottom-up developer wedge is exactly Boldstart's pattern. Quaestor's wedge is "developer types `/install quaestor-mcp` and the agent has a wallet" — this is their kind of deal.
- **Warm-intro path:** Any Boldstart portfolio founder in dev tools.
- **First-email hook:** "MCP-first agent wallet, open-core, sovereign keys. Phase 1 shipped solo in 36 hours. Looking for an infra-thesis lead — would love 20 minutes."
- **Status:** not contacted

---

## B. Strategic angels

Target: 8–12 SAFE checks of $50–250K each. Pick for *signal*, not just dollars
— the right angel list closes the lead.

| Name | Why them | Status |
|---|---|---|
| Matt Huang (Paradigm / Tempo co-founder) | Protocol depth; MPP first-party | not contacted |
| Cuy Sheffield (Visa head of crypto) | Payments infra credibility, regulatory perspective | not contacted |
| Brian Armstrong angels list (Coinbase ecosystem) | Coinbase-orbit signal; warm path to CDP | not contacted |
| Anthropic engineers building MCP | First-party MCP credibility; product-feedback loop | not contacted |
| Stripe agent-team alums (MPP authors) | MPP wire-format depth; merchant-side perspective | not contacted |
| Browserbase founders | Agent-infra peer; same buyer | not contacted |
| Crossmint founders (intel only — competitor caveat) | Direct competitor; bring only as informational coffee, not check | not contacted |
| Skyfire founders (intel only — competitor caveat) | Direct competitor; same caveat | not contacted |
| Chris Hay (outspoken x402 builder, X) | Public-facing x402 evangelist; signal amplification | not contacted |
| Other outspoken x402 / AP2 builders on X | Same; pick 2–3 | not contacted |
| A100 angels (Calgary tech network) | Local-network depth; soft warm intros to Canadian funds | not contacted |
| Startup Edmonton angel network | Local-network depth; first-check culture | not contacted |

**Caveat noted in row:** Crossmint and Skyfire are *competitors*. Coffee for
intel is fine — never ask for a check, never share material non-public info.
Treat the meeting as competitive research; expect the same in return.

---

## C. Canadian-specific funds

Target: 1–2 of these to fill the round if a Tier-1 lead doesn't materialize
fast enough. Most write smaller checks ($250K–$1M); useful as round-fillers
or as a pre-seed lead with a follow-on tier.

| Fund | Stage / fit | Status |
|---|---|---|
| A100 (Calgary) | Pre-seed; Alberta network; founder-led | not contacted |
| BCF Ventures (Toronto) | Pre-seed/seed; tech-broad; Quebec → ON expansion | not contacted |
| Garage Capital (Waterloo) | Seed; B2B SaaS / dev-tools shaped | not contacted |
| Inovia Capital (Montreal) | Seed → Series A; multi-fund; pan-North America | not contacted |
| Real Ventures (Montreal) | Seed; founder-friendly; pre-product OK | not contacted |
| Luge Capital (Montreal) | Seed; fintech-specific; payments adjacency | not contacted |
| Panache Ventures (Vancouver) | Pre-seed/seed; tech-broad; CDN coverage | not contacted |

---

## Discipline notes

- Update **only** with status changes — don't fluff "warm intro identified" into "diligence" because you sent a follow-up.
- Each meeting → write a 5-line note (date, who attended, the one objection raised, the next step, the follow-up date). Keep notes in `~/code/quaestor-docs/private/investor-notes/` (gitignored — don't commit).
- "Passed" is fine. Note the reason in one line. Re-pitch when the reason no longer applies (e.g. "passed because pre-product" → revisit at Phase 2 GA).
- Lead first, fillers second. Don't fill the round with $50K SAFEs and then try to find a lead — that's the cap-table version of squatter's rights.
