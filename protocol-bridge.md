# Protocol bridge — speaking MPP, x402 v2, and AP2 from one mandate

> **What this document is.** A reference for *how* [`quaestor-bridge`](https://github.com/PawCheck1/quaestor-bridge) translates a single Quaestor mandate credential into the wire format three different agent-payment ecosystems insist on, *why* the SDK versions are pinned exactly, and *what* spec / SDK drift we have already absorbed. The companion in-repo doc is [`quaestor-bridge/ARCHITECTURE.md`](https://github.com/PawCheck1/quaestor-bridge/blob/main/ARCHITECTURE.md); this document is the externally-readable summary for people who do not yet want to clone the repo.

## The shape of the problem

There is no agreed wire format for "an autonomous agent paying a merchant" in 2026-04. There are at least three live candidates, each with its own constituency:

| Protocol | Origin | Header | Use case |
| --- | --- | --- | --- |
| **MPP** (IETF Payment) | Stripe + Tempo, IETF draft | `Authorization: Payment scheme=mpp; v=1; …` | One-shot HTTP payments + the March 2026 sessions primitive for streaming micropayments |
| **x402 v2** | Coinbase | `X-PAYMENT: <base64url(envelope)>` | Multi-chain stablecoin settlement (`base-sepolia`, `solana-devnet`, `tempo-testnet`, mainnets behind a flag) |
| **AP2** | Google A2A extension | `X-A2A-Payment: <base64url(envelope)>` | A2A 402 challenge → AP2 envelope wrapping x402 settlement |

Each protocol expects payment-instrument primitives that look superficially similar but differ in detail (recipient field name, asset specification, session semantics, envelope keying). A naive integration ships three full payment libraries and three custody stories. Quaestor ships **one mandate primitive plus three thin adapters**.

## How Quaestor speaks all three from one credential

Single redeem path:

```
Agent ──(mandate JWT)──▶ Bridge
                         │
                         ▼
                 POST /mandate/redeem  (X-Local-Auth)
                         │
                         ▼
                 quaestor-core verifies signature, exp, amount_max,
                 use_counter_max, then returns a Credential
                         │
                         ▼
                 Adapter builds the protocol-native header
                 from the same Credential — no second redeem
```

The same credential can be projected into any of the three formats; the adapter is a pure function of the credential plus a network/policy context. The bridge does **not** verify the mandate (that is core's job), does **not** hold keys, and does **not** persist state — every adapter call is one redeem followed by one header construction.

### MPP adapter — `src/adapters/mpp.ts`

[`MppAdapter`](https://github.com/PawCheck1/quaestor-bridge/blob/main/src/adapters/mpp.ts) emits the IETF Payment header. One-shot:

```
Authorization: Payment scheme=mpp; v=1; network=<n>; currency=<c>;
                       amount=<a>; recipient=<r>; ref=<credential_id>
```

The March-2026 sessions primitive is supported as well, with `kind=session; session=mpp_sess_<credential_id>; amount_max=<n>; expires=<epoch>` for streaming micropayments where the agent and merchant agree to a cap rather than a per-call amount. Verified by [`test/mpp.test.ts`](https://github.com/PawCheck1/quaestor-bridge/blob/main/test/mpp.test.ts) and the runtime-resilience regression in [`test/mpp-runtime.test.ts`](https://github.com/PawCheck1/quaestor-bridge/blob/main/test/mpp-runtime.test.ts).

### x402 adapter — `src/adapters/x402.ts`

[`X402Adapter`](https://github.com/PawCheck1/quaestor-bridge/blob/main/src/adapters/x402.ts) emits the Coinbase x402 v2 envelope:

```json
{ "v": 2, "scheme": "exact", "network": "<n>", "payTo": "<r>",
  "asset": "<c>", "maxAmountRequired": "<a>", "resource": "<u>",
  "reference": "<credential_id>", "extras": {} }
```

base64url-encoded as the `X-PAYMENT` header value. Verified for `base-sepolia`, `solana-devnet`, and `tempo-testnet` parametrically in [`test/x402.test.ts`](https://github.com/PawCheck1/quaestor-bridge/blob/main/test/x402.test.ts) (`it.each`). Mainnet is refused unless `QUAESTOR_MAINNET=1`.

### AP2 adapter — `src/adapters/ap2.ts`

AP2 is an A2A extension, not a separate settlement protocol. [`Ap2Adapter`](https://github.com/PawCheck1/quaestor-bridge/blob/main/src/adapters/ap2.ts) parses the A2A 402 challenge, validates that the `accepts` list contains an `ap2` payload, then **calls `X402Adapter.pay` to actually settle** and wraps the resulting `X-PAYMENT` in an AP2 envelope. The `vi.spyOn(x402, 'pay')` assertion in [`test/ap2.test.ts`](https://github.com/PawCheck1/quaestor-bridge/blob/main/test/ap2.test.ts) ensures AP2 *delegates* to x402 rather than reimplementing settlement — if AP2's spec evolves the way it references settlement, that change happens in `adapters/x402.ts`, not in two parallel code paths.

## SDK pinning rationale

| Package | Pinned version | Why exact, not a range |
| --- | --- | --- |
| `x402` (Coinbase) | `1.2.0` | Wire format changed between 1.x minors during the December 2025 modular rewrite (`recipient` → `payTo`, asset specification semantics). We need byte-for-byte determinism against published facilitators. |
| `mppx` | `0.6.7` | Pre-1.0; the `sessions` primitive landed in 0.6.x and the 0.7.x line is expected to rename `openSession` → `startStream`. Pinning to `0.6.7` prevents silent contract drift on `pnpm update`. |

These are deliberately *exact* version pins, not `^` or `~` ranges.

## The 2026-04-28 spec-drift log entry — and why both adapters carry an in-house fallback

Both pinned tarballs — `mppx@0.6.7` and `x402@1.2.0` — ship with **broken or empty `dist/`** on npm as of 2026-04-28. Three failure modes were observed in production:

1. **Reject** — `import('mppx')` or `import('x402')` throws synchronously; caught and the in-house builder engages.
2. **Empty module** — the import resolves but the default export is `undefined`; detected at the call site and the in-house builder engages.
3. **Hang** — `import()` never settles. This was reproduced specifically against the published `mppx` tarball on Node 22 + pnpm strict store, and it was the root cause of the 2026-04-28 demo hang.

The fix is identical in both adapters: race `import()` against a 2-second `Promise.race` timer. If the timer wins we use the in-house builder. The spec-drift log captures this exactly:

| Date       | Spec / SDK              | Drift                                     | Resolution                                  |
| ---------- | ----------------------- | ----------------------------------------- | ------------------------------------------- |
| 2025-12-15 | Coinbase x402 v2        | `payTo` renamed from `recipient`          | Fallback envelope uses `payTo`              |
| 2026-03-01 | MPP IETF draft          | `sessions` primitive added                | `kind=session` + `amount_max` in header     |
| 2026-04-12 | AP2 (A2A 1.0 ext)       | Envelope key `extension` (was `ext`)      | AP2 envelope uses `extension: 'ap2'`        |
| 2026-04-28 | mppx@0.6.7 / x402@1.2.0 | Both ship empty/broken `dist/` on npm     | 2 s timeout-race + per-process fallback warn|

The in-house builders produce the same wire format the published SDKs are *supposed* to produce. Fallback engagement logs one warning per process (not per call) so an operator running this in production sees exactly one signal that the SDK is broken — the demo continues. The honest caveat: because we currently fall back, **we are not exercising the official SDKs in CI**. When upstream ships fixed tarballs we should remove the fallback warning, bump the pin, and run a contract test against the real wire format.

## What the bridge does not do

By construction, not by oversight:

- **Does not hold keys.** Every committed file is greppped for private-key-shaped strings on every CI run by [`test/ap2.test.ts → "no key material leaks"`](https://github.com/PawCheck1/quaestor-bridge/blob/main/test/ap2.test.ts). If anyone ever introduces a key to the bridge, the build fails.
- **Does not verify mandate JWTs.** Every redeem is a `POST /mandate/redeem` to core. Core decides validity; the bridge transports the verdict.
- **Does not pick policy.** Network classification (testnet vs mainnet), mainnet opt-in (`QUAESTOR_MAINNET=1`), and ledger writes are all gated by env vars or proxied through core. The bridge has no admin surface.
- **Does not persist state.** Restarting the bridge loses nothing. The ledger is core's job.

## What's next on the protocol surface

Documented in [`quaestor-bridge/STATUS.md → Next phase`](https://github.com/PawCheck1/quaestor-bridge/blob/main/STATUS.md): expose `pay_mpp`, `pay_x402`, `pay_ap2`, and a routing helper `pay_from_402` as MCP tools so an agent can pay through stdio MCP without ever touching HTTP directly. The trust boundary stays the same — the MCP transport reuses core's `X-Local-Auth` bearer.

When upstream `mppx` and `x402` ship fixed tarballs, we drop the per-process fallback warning and add a contract-test layer that round-trips our envelopes against a mock facilitator. This is the most important "known gap" to close on the bridge — until then we have correct-on-paper wire format that has not been validated against a real facilitator response.
