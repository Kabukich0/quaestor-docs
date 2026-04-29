# HANDOFF — Phase 1.5b

You (fresh Claude Code session) are picking up after Phase 1.5a shipped real
on-chain x402 settlement on Base Sepolia. Read this before doing anything.

## Phase 1.5a — what was shipped

End-to-end live: agent → bridge → core (signs EIP-3009) → bridge formats
X-PAYMENT envelope → demo-seller → x402.org facilitator → on-chain.

**Proof tx (Base Sepolia):**
`https://sepolia.basescan.org/tx/0xbaeb93856c4a660e06d0733b53f67c60d1fa59667fd5810e0c82cbf86ba8ac56`
ledger receipt entry_id `121`.

### Repo state

| Repo | tip of `main` | latest CI run |
|---|---|---|
| `Kabukich0/quaestor-core` | `6ff567c style(eip3009): biome formatting` | https://github.com/Kabukich0/quaestor-core/actions/runs/25132469872 (green) |
| `Kabukich0/quaestor-bridge` | `eb68801 test(live): add on-chain x402 settlement live test + readme` | https://github.com/Kabukich0/quaestor-bridge/actions/runs/25132212639 (green) |
| `Kabukich0/quaestor-mcp` | `d5796f0 chore: identity cleanup …` (untouched in 1.5a) | green |

Phase 1.5a commits, in order:

- core `9bdfc40` `feat(vault): add per-mandate settlement key derivation`
- core `c420cf5` `feat(daemon): add /mandate/:jti/settlement-address and /mandate/sign-x402`
- core `02b7df3` `test(daemon): cover sign-x402 policy + atomicity paths; add no-key-leak grep`
- core `6be8323` `fix(eip3009): match Base Sepolia USDC's domain.name "USDC" + actually compare DOMAIN_SEPARATOR`
- core `6ff567c` `style(eip3009): biome formatting`
- bridge `2a85e02` `feat(adapters): wire x402 to core sign-x402 endpoint`
- bridge `8b66cc8` `feat(examples): add minimal x402 demo seller`
- bridge `eb68801` `test(live): add on-chain x402 settlement live test + readme`

## Funded settlement state

- **mandate jti:** `a47e75e02bfc7792656ab63a3952dc9c`
- **mandate_index:** `1`
- **derived settlement address:** `0xf3F00F230aD037cA3b730b5E229495c085578c17`
- **JTI/JWT files (still on disk):** `/tmp/jti-15a.txt`, `/tmp/jwt-15a.txt`
- **Original mandate response JSON:** `/tmp/mandate-15a-raw.json`
- **Mandate cap:** $0.10, `use_counter_max=3`, ttl 1h from ~14:08 UTC
  2026-04-29 (likely **expired** by now — re-issue if needed).
- **Use counter:** 1 redeem consumed (the live test). Two uses remain on the
  same mandate IF it hasn't expired.
- **Funding (last verified pre-test):** ~5 USDC + small ETH on
  `0xf3F00F230aD037cA3b730b5E229495c085578c17`. Live test consumed `0.001 USDC`
  → user's payout `0xe5146463c01Ed787A410b6434b3cd8D25eE9CCD0`. Verify
  current balance:
  https://sepolia.basescan.org/address/0xf3F00F230aD037cA3b730b5E229495c085578c17

If you need a fresh mandate against the **same funded address**, that's
impossible — `mandate_index` advances per issuance. Either reuse this
mandate (if not expired) or fund the next derived address.

## Daemon state

- **Not running.** Killed at end of 1.5a.
- pidfile path: `/tmp/quaestor-core-15a.pid` (removed)
- log path: `/tmp/quaestor-core-15a.log`
- Restart: `cd ~/code/quaestor-core && pnpm build && node ./bin/run.js start`
  (build first — dist/ may not match HEAD if you skipped that step in 1.5a).
- Auth token: `~/.config/quaestor/auth.token`
- DB: `~/Library/Application Support/quaestor/{vault,ledger}.db` (macOS).
  The funded mandate row is still in the `mandates` SQLite table — survives
  restart.

## In-flight gotchas — load-bearing context

1. **Base Sepolia USDC `name()` is `"USDC"`, NOT `"USD Coin"`.** This was
   the live-test failure. Fixed in core `6be8323`. `verifyDomainSeparator`
   now actually compares the on-chain DOMAIN_SEPARATOR against a locally-
   recomputed one (keccak256 over EIP712Domain typehash + name + version
   + chainId + verifyingContract). Don't undo this. If you ever add
   another network, query its `name()` first.

2. **`@coinbase/x402` exports `facilitator` pointing at
   `api.cdp.coinbase.com/platform/v2/x402` which requires CDP API auth.**
   Demo seller defaults to `https://x402.org/facilitator` (the public
   testnet endpoint, no creds). Override with `DEMO_SELLER_FACILITATOR_URL`.
   `examples/demo-seller/index.ts:13` has the constant + comment. Don't
   "fix" the import to use `@coinbase/x402`'s default — that broke the
   first live run.

3. **`LIVE_MANDATE_JTI` + `LIVE_MANDATE_JWT` env override pattern.** Live
   test default is "issue a fresh mandate", but every fresh mandate
   advances `mandate_index` → new derived address → unfunded. The two
   env vars short-circuit issuance and reuse a pre-funded mandate. This
   is how the live test ran today. Without it, the test fails on
   "insufficient funds" until the operator funds the new address.

4. **pnpm 9 + `file:../quaestor-bridge` virtual-store gotcha.** mcp's CI
   patches bridge's `package.json` exports + rsyncs bridge dist into
   `node_modules` because pnpm packs file: deps with `.gitignore` semantics
   and drops dist/. This still applies — Phase 1.5b cleanup item is to
   push the bridge `package.json` exports fix (`./dist/src/...` paths,
   bump 0.1.0 → 0.1.2) and to remove dist from bridge's `.gitignore`
   whitelist. See `mcp/.github/workflows/test.yml` "Patch quaestor-bridge
   package.json exports" + "Materialize bridge into mcp/node_modules".

5. **The bridge's local `package.json` is uncommitted-edit-different from
   remote main.** Local has `version: "0.1.2"` and `exports: ./dist/src/...`;
   remote has `0.1.0` and `./dist/...`. mcp CI clones remote and patches
   in-place. If you push the bridge package.json, the mcp CI patch step
   becomes a no-op and can be removed.

## File paths that matter for 1.5b

**Core (signing path):**
- `quaestor-core/src/lib/eth-key.ts` — BIP-32 secp256k1 derivation, `m/44'/60'/0'/0/<mandate_index>`
- `quaestor-core/src/lib/eip3009.ts` — EIP-712 typed-data signing + DOMAIN_SEPARATOR check
- `quaestor-core/src/lib/mandate-store.ts` — `mandates` + `audit_signatures` SQLite tables
- `quaestor-core/src/lib/server.ts` — `/mandate/:jti/settlement-address`, `/mandate/sign-x402`
- `quaestor-core/test/sign-x402.test.ts` — 11 cases covering happy path + rejections
- `quaestor-core/test/no-key-leak.test.ts` — architectural-invariant grep

**Bridge (settlement path):**
- `quaestor-bridge/src/adapters/x402.ts` — `pay()` (legacy) + `payOnChain()` (new) + `encodeX402Header`
- `quaestor-bridge/src/core-client.ts` — `getSettlementAddress`, `signX402`, `writeReceipt`
- `quaestor-bridge/src/types.ts` — `Eip3009Payload`, `SettlementAddress`, `SignX402Response`
- `quaestor-bridge/examples/demo-seller/index.ts` — Express + x402-express + facilitator URL
- `quaestor-bridge/test/live/x402-onchain.test.ts` — RUN_LIVE-gated end-to-end
- `quaestor-bridge/test/ap2.test.ts` — `no key material leaks` grep with `/0x[0-9a-fA-F]{64}\b/`
- `quaestor-bridge/.env.example`, `examples/demo-seller/.env.example`

**mcp (untouched in 1.5a — picks up new bridge behavior transparently):**
- `quaestor-mcp/.github/workflows/test.yml` — has the patch + rsync steps
  that 1.5b cleanup should remove

## `pay()` vs `payOnChain()` — keep them straight

`X402Adapter` has two payment surfaces:

- **`pay({ mandate, amount, recipient, resource?, extras? })`** — legacy
  composition path. Calls `core.redeem()` (NOT `signX402`), builds an
  X-PAYMENT envelope from the credential, returns `{ header, credential }`.
  Used by:
  - `Ap2Adapter.pay` — proven by `vi.spyOn(x402, 'pay')` in
    `test/ap2.test.ts → "delegates settlement to the x402 adapter"`. AP2
    must continue to compose over this; do NOT delete `pay()`.
  - All offline tests (`test/x402.test.ts` 10 cases, `test/ap2.test.ts` 5)
- **`payOnChain({ mandate, mandate_jti, resource_url, value, pay_to, ... })`**
  — Phase 1.5a addition. Calls `core.getSettlementAddress` →
  `core.signX402` → POSTs to seller → captures tx_hash from
  `X-PAYMENT-RESPONSE` → writes ledger receipt. Returns
  `{ tx_hash, basescan_url, ledger_seq, credential_id, from, facilitator_response }`.
  Used by:
  - `test/live/x402-onchain.test.ts` (RUN_LIVE only)
  - The demo (manual / future demo.mp4 re-record)

If 1.5b migrates AP2 to core-signs, that's the place to deprecate `pay()`.
Until then, both coexist.

There is no `payOnCore()` — only the two above.

## What 1.5b is for (per Phase 1.5a spec's "non-goals" / `TOMORROW.md`)

Cleanup, not new features:

1. Push bridge `package.json` exports fix + version bump 0.1.0 → 0.1.2
   to remote, remove the corresponding mcp CI patch step.
2. Fix bridge `.gitignore` (remove dist/ or add `.npmignore` whitelisting
   dist) so pnpm 9 packs file: deps correctly, remove the mcp CI rsync
   materialize step.
3. Phase 1.4 territory (NOT 1.5b): POST `/mandate/revoke` in core. Bridge
   already has revoke wiring (`quaestor-mcp/src/tools/revoke_mandate.ts`
   surfaces 404 verbatim).
4. Re-record `~/code/quaestor-bridge/docs/demo.mp4` to show the live
   on-chain flow. README "Run the live demo" section already has the
   recipe.
5. Phase 1.5b is also the right place to add a unit test that proves
   `verifyDomainSeparator` actually compares on-chain vs local rather
   than just shape-checking. Currently only the live test exercises that
   path.

## What you'd lose if context cleared NOW

- The funded address (`0xf3F00F230aD037cA3b730b5E229495c085578c17`) is in
  this file, the mandate JTI is in this file, the JWT is at
  `/tmp/jwt-15a.txt`. **DO NOT lose track of these — funding is real
  testnet money even if testnet money is free.**
- The USDC `name()` finding (#1 above) is the kind of thing that costs an
  hour to rediscover from a cryptic `invalid_exact_evm_signature` error.
- The facilitator URL override (#2) — same.
- The `LIVE_MANDATE_*` env pattern (#3) — same.

Read the in-flight-gotchas section twice before touching anything.
