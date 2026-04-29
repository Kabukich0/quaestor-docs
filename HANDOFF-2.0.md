# HANDOFF — Phase 2.0 (local policy LLM)

You (fresh Claude Code session) are picking up after Phase 1.5b shipped:
the generic `signMandate` pipeline, three core signing endpoints
(x402 / ap2 / mppx), JWKS exposure, mandate revocation, and the bridge
migration to a uniform core-signs architecture across all three
protocols. Read this before doing anything for 2.0.

## Phase 1.5b — final state

End-to-end live across three protocols, one trust boundary. Bridge holds
zero keys for x402, ap2, and mppx alike — every signing path goes
through quaestor-core's `signMandate` pipeline.

### Repo tips

| Repo | tip of `main` | latest CI |
|---|---|---|
| `Kabukich0/quaestor-core` | `2de9aa8 feat(daemon): generic signMandate pipeline + sign-ap2 + sign-mppx + jwks` | https://github.com/Kabukich0/quaestor-core/actions/runs/25133645877 (green) |
| `Kabukich0/quaestor-bridge` | `a8f8614 chore(deps): add jose to devDeps for ap2 live test JWS verification` | https://github.com/Kabukich0/quaestor-bridge/actions/runs/25134529864 (green) |
| `Kabukich0/quaestor-mcp` | `ea3d5e0 ci(mcp): build bridge before tsc — pnpm 9 file: dir-mode skips prepare` | https://github.com/Kabukich0/quaestor-mcp/actions/runs/25134324099 (green) |

### Phase 1.5b commits, in order

- core `f3442c6` `feat(daemon): add POST /mandate/revoke with audit trail`
- core `2de9aa8` `feat(daemon): generic signMandate pipeline + sign-ap2 + sign-mppx + jwks`
- bridge `c968f01` `feat(adapters): migrate ap2 + mppx to core-signs via payOnCore`
- bridge `ffa098d` `chore(pkg): add prepare hook + .npmignore so consumers get a built dist`
- mcp `a2073aa` `ci(mcp): drop bridge package.json patch + rsync materialize hacks`
- mcp `ea3d5e0` `ci(mcp): build bridge before tsc — pnpm 9 file: dir-mode skips prepare`
- bridge `ef60f7d` `test(live): add ap2 + mppx RUN_LIVE round-trip tests`
- bridge `a8f8614` `chore(deps): add jose to devDeps for ap2 live test JWS verification`

### RUN_LIVE artifacts (this phase)

- **x402**: deferred — 1.5a Basescan tx
  `0xbaeb93856c4a660e06d0733b53f67c60d1fa59667fd5810e0c82cbf86ba8ac56`
  remains the standing on-chain artifact. The 1.5a-funded mandate
  (jti `a47e75e02bfc7792656ab63a3952dc9c`) expired ~1h after issuance.
  Re-running x402 live needs fresh funding because mandate_index
  advances per issuance → new derived address → unfunded. Bridge
  x402 paths weren't touched in 1.5b, so this is intentionally
  not re-proven this phase.
- **AP2**: fresh proof via `test/live/ap2-onchain.test.ts` (RUN_LIVE=1).
  - kid: `ap2-2-587620f73f5039cf`
  - JWS verifies against `GET /mandate/<jti>/jwks` via `jose.compactVerify`.
  - Decoded payload bytes-equal `canonicalJSON(vc-without-proof)`.
- **MPP-X**: fresh proof via `test/live/mppx-onchain.test.ts` (RUN_LIVE=1).
  - signer: `0x24c38c46BAdd0b92a168c0E3ce71bD483fF37f91`
  - `viem.recoverMessageAddress(canonical, sig)` matches that same
    address as served by `GET /mandate/<jti>/settlement-address`.
  - secp256k1 settlement-key family is shared with x402 (same
    `m/44'/60'/0'/0/<mandate_index>` derivation).

## The signMandate pipeline — load-bearing context for 2.0

This is the architectural artifact 2.0 needs to understand. It lives at
`quaestor-core/src/lib/sign-pipeline.ts` and is the single trust-
boundary funnel for every signature the daemon emits.

```ts
async function signMandate<T>(args: {
  ledger: Ledger;
  mandateStore: MandateStore;
  body: Record<string, unknown>;
  mandate_jwt: string;
  module: ProtocolModule<T>;
  opts: SignPipelineOpts;
}): Promise<HandlerResult>
```

Pipeline owns (in order):
1. JWT verification + expiry  (`verifyMandate`)
2. Use-counter check (count of `mandate.redeem` ledger entries)
3. Mandate-row lookup
4. **Revocation gate** — `mandate.revoked_at_ms != null` → 403 MANDATE_REVOKED
5. Per-protocol payload validation (delegated)
6. Spend-cap check (in base units against `spend_cap_remaining_base`)
7. Per-protocol signing (delegated)
8. Atomic `ledger.append({type:'mandate.redeem', ...}) + decrementAndAudit`
9. Build response: `{ signature, from?, kid?, ...extra }`

Per-protocol modules in `quaestor-core/src/lib/proto-modules.ts` —
`x402Module`, `ap2Module`, `mppxModule` — each implement
`ProtocolModule<T>`: `validatePayload`, `payloadHash`, `sign`,
`buildCredential`. New protocols slot in here.

### Why 2.0 must care

Phase 2.0 plans to add a **local policy LLM** as the
`quaestor-policy` plugin. The intended insertion point is
**between (4) and (5)** of `signMandate`: after revocation/JWT/
counter checks pass, but before per-protocol signing fires. A new
hook — call it `policyEvaluate(claims, rawBody, mandate)` — would let
the policy plugin inspect the parsed body, the mandate metadata, and
free-form context (purpose_tag, recipient identity, recent ledger
history) and return either:
- `{ allow: true }` — pipeline proceeds
- `{ allow: false, reason: string, code: string }` — pipeline returns
  403 with the policy-supplied error code.

This is the cleanest place to add it because:
- Every protocol shares the pipeline, so one hook covers all three.
- The pipeline already owns the atomic ledger commit, so adding a
  `policy.deny` ledger entry on a denial is a one-liner inside the
  same transaction.
- The hook fires *before* any cryptographic operation, so denials
  cost nothing in vault key access.

The hook signature should be deliberately narrow:
- Pass `claims` (already parsed), `rawBody` (the protocol body as
  given), and the `MandateRow` (so the policy can read remaining cap,
  age, network).
- Do NOT pass the JWT, the auth token, or anything from the vault.
- Return synchronous-or-async; the pipeline already awaits across
  protocol modules.

### What you must NOT do in 2.0

1. Do not collapse the per-protocol modules. The fan-out at
   `validatePayload` / `sign` / `buildCredential` is what makes
   adding a fourth or fifth protocol a single file. Keep
   `proto-modules.ts` flat.
2. Do not move `signMandate` out of core. The trust boundary is the
   thing that makes Quaestor ship-able; the policy plugin attaches
   to it, doesn't replace it.
3. Do not let the policy plugin have a path to vault keys. The
   plugin's interface is `(claims, body, mandateRow) → allow | deny`.
   That's the entire surface.
4. Do not add an "advisory" / "warn" mode where the pipeline
   continues past a denial. Either the policy denies (pipeline
   returns 403) or it allows (pipeline proceeds). Anything else is
   a footgun.

## Key family map (for 2.0 plugin authors)

The core daemon now derives three distinct key families from the same
BIP-39 vault root, all in-process, never persisted:

| Family | Path | Use | Defined in |
|---|---|---|---|
| Mandate-issuance | `m/44'/0'/0'/0'/0'` | EdDSA over JWT (issued mandates) | `vault.ts` |
| Settlement (secp256k1) | `m/44'/60'/0'/0/<mandate_index>` | EIP-3009 (x402) + EIP-191 (mppx) | `eth-key.ts` |
| AP2 signing (Ed25519) | `m/44'/9402'/<mandate_index>'` | EdDSA over canonicalJSON(vc) | `ap2-key.ts` |

`9402` = "quaestor"; deliberately distinct slot to keep
compromise/rotation surfaces separated.

## Deprecated paths scheduled for removal in 1.6

- `Ap2Adapter.pay()` — composition-over-x402 path. Marked
  `@deprecated`, logs once-per-process to console.warn. Used by
  legacy callers. Test-only reset hook: `_resetAp2DeprecationLatch`.
- `MppAdapter.pay()` — fixture-builder fallback path. Same
  treatment. Test-only reset: `_resetMppDeprecationLatch`.
- Both surfaces remain functional through 1.6. Plan: a
  bridge `0.2.0` major bump removes them and renames `payOnCore`
  to `pay`.

If 2.0 touches the bridge adapters, do NOT delete `pay()` — the
removal happens at the 0.2.0 cut, not in a feature phase.

## Schema additions in 1.5b — survive across restarts

`mandates` table:
```sql
ALTER TABLE mandates ADD COLUMN revoked_at_ms INTEGER;
```

`audit_signatures` table:
```sql
ALTER TABLE audit_signatures ADD COLUMN kind TEXT NOT NULL DEFAULT 'sign';
```

Migrations are idempotent and run from `Ledger.runMigrations()` on
constructor. The funded 1.5a mandate row survives intact (now with
`revoked_at_ms = NULL`).

For 2.0: if you need a `policy_decisions` audit table, add it in the
same `Ledger` constructor block with a `CREATE TABLE IF NOT EXISTS`
plus a matching idempotent ALTER for any later columns. Don't break
the migration pattern.

## What 2.0 needs to know about CI / the build

- `quaestor-bridge` is consumed by `quaestor-mcp` via
  `file:../quaestor-bridge` (pnpm-9 directory-mode → symlink). Bridge
  ships a `prepare` hook + `.npmignore` so future consumers see a
  built `dist/`, but pnpm-9 directory mode does NOT fire `prepare`,
  so mcp's CI explicitly runs `pnpm build` in the bridge dir before
  its own `pnpm install`. This goes away when bridge publishes to
  npm in 1.6.
- Bridge has `jose` as a devDep so the `test/live/ap2-onchain.test.ts`
  imports resolve at vitest module-load time even when `RUN_LIVE` is
  unset. If 2.0 adds a new live-test file that pulls in a
  non-runtime dep, register it in `devDependencies` — vitest
  evaluates test-file imports unconditionally.
- All three repos expect Node 22 LTS, pnpm 9, `shamefully-hoist=true`.
  Conventional commits, lowercase, push after each commit.

## Files that matter for 2.0

**Core (the integration surface):**
- `quaestor-core/src/lib/sign-pipeline.ts` — the function 2.0 hooks
- `quaestor-core/src/lib/proto-modules.ts` — per-protocol fan-out
- `quaestor-core/src/lib/server.ts` — route → signMandate dispatcher
- `quaestor-core/src/lib/ledger.ts` — schema + EntryType union (add
  `'policy.deny'` here when 2.0 starts logging denials)
- `quaestor-core/src/lib/mandate-store.ts` — `revoke`, `decrementAndAudit`

**Tests that demonstrate the trust-boundary invariant:**
- `quaestor-core/test/no-key-leak.test.ts` — grep regression
- `quaestor-core/test/sign-{x402,ap2,mppx}.test.ts` — protocol parity
- `quaestor-core/test/revoke.test.ts` — gate semantics
- `quaestor-bridge/test/ap2.test.ts` — `no key material leaks` grep
- `quaestor-bridge/test/{ap2,mppx}-core.test.ts` — bridge-side mocks
- `quaestor-bridge/test/live/{x402,ap2,mppx}-onchain.test.ts` — RUN_LIVE
- `quaestor-mcp/test/no-key-material.test.ts` — mcp-side grep

**Phase 1.5a artifacts to NOT touch:**
- `quaestor-bridge/examples/demo-seller/` — stable, do not modify
- The 1.5a x402 tx hash above — historical record, immutable

## Things 2.0 might want but don't exist yet

- A `policy.evaluate` ledger EntryType. Add to `EntryType` union
  in `ledger.ts` when the plugin lands.
- A `/mandate/policies` endpoint to list active policy IDs per
  mandate. The shared pipeline could surface them in the 200 body.
- A way for the policy plugin to request the **last N redemptions
  for this mandate** to detect velocity. Right now `MandateStore`
  has `auditFor(jti)` which exposes `audit_signatures` rows; that's
  the cheapest source. Wire `auditFor` through the hook args if
  needed; do not give the plugin direct DB access.
- A `quaestor-policy` package layout. Suggested: TypeScript ESM,
  same Node-22+pnpm-9 toolchain, exports a `createPolicy(opts)`
  factory that returns the hook. Core depends on it via the same
  `file:../quaestor-policy` pattern bridge uses today.

## Read me twice

- The signMandate pipeline is the integration point. Do not bypass it.
- The bridge holds zero keys. The grep regression is sacred. Do not
  introduce a path that imports `viem/accounts.privateKeyToAccount`,
  `@noble/curves`, or any private-key-shaped string into bridge
  src/, even temporarily for "spike" code. The grep tests will fail
  and CI will block the merge.
- Phase 2.0 is policy, not crypto. If you find yourself signing
  something inside the policy plugin, stop — that work belongs in
  core, behind a new ProtocolModule.
