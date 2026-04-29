# HANDOFF — Phase 2.0 (local policy LLM)

You (fresh Claude Code session) are picking up after Phase 1.5b shipped
the generic `signMandate` pipeline + uniform core-signs across x402 /
ap2 / mppx, and after Phase 1.5a's narrated walkthrough was published as
a GitHub Release on quaestor-demos. This is the entry point for Phase
2.0: the local policy LLM. Read this carefully — the four design
decisions below are LOCKED.

## Final state (1.5a + 1.5b combined)

### Tip of main, all repos green

| Repo | Tip | Latest CI |
|---|---|---|
| `Kabukich0/quaestor-core` | `e754f49 docs: embed phase 1.5a demo video` | https://github.com/Kabukich0/quaestor-core/actions (green) |
| `Kabukich0/quaestor-bridge` | `e765db3 docs: embed phase 1.5a demo video` | https://github.com/Kabukich0/quaestor-bridge/actions (green) |
| `Kabukich0/quaestor-mcp` | `6256389 docs: embed phase 1.5a demo video` | https://github.com/Kabukich0/quaestor-mcp/actions (green) |
| `Kabukich0/quaestor-demos` | `d55b544 docs: embed phase 1.5a video + add MILESTONES.md` | https://github.com/Kabukich0/quaestor-demos/actions (green) |
| `Kabukich0/quaestor-docs` | `a886415 docs: cross-link phase 1.5a release` | n/a |

### Phase 1.5a — on-chain artifact

Settlement tx on Base Sepolia:
`0xbaeb93856c4a660e06d0733b53f67c60d1fa59667fd5810e0c82cbf86ba8ac56`
(https://sepolia.basescan.org/tx/0xbaeb93856c4a660e06d0733b53f67c60d1fa59667fd5810e0c82cbf86ba8ac56).
Bridge held zero keys; core's HD vault signed in-process; demo-seller +
public x402.org facilitator broadcast.

### Phase 1.5b — architecture commits

Shipped in this order on core, bridge, mcp:

- core `f3442c6` `feat(daemon): add POST /mandate/revoke with audit trail`
- core `2de9aa8` `feat(daemon): generic signMandate pipeline + sign-ap2 + sign-mppx + jwks`
- bridge `c968f01` `feat(adapters): migrate ap2 + mppx to core-signs via payOnCore`
- bridge `ffa098d` `chore(pkg): add prepare hook + .npmignore so consumers get a built dist`
- mcp `a2073aa` `ci(mcp): drop bridge package.json patch + rsync materialize hacks`
- mcp `ea3d5e0` `ci(mcp): build bridge before tsc — pnpm 9 file: dir-mode skips prepare`
- bridge `ef60f7d` `test(live): add ap2 + mppx RUN_LIVE round-trip tests`
- bridge `a8f8614` `chore(deps): add jose to devDeps for ap2 live test JWS verification`

RUN_LIVE artifacts (1.5b):
- AP2: kid `ap2-2-587620f73f5039cf`, JWS verifies via `jose.compactVerify`
  against `GET /mandate/<jti>/jwks`.
- MPP-X: signer `0x24c38c46BAdd0b92a168c0E3ce71bD483fF37f91`,
  `viem.recoverMessageAddress` recovers the same address as the
  settlement-address endpoint.

### Phase 1.5a video — the canonical walkthrough

Released as `v0.1.5a` on quaestor-demos:
- Tag: https://github.com/Kabukich0/quaestor-demos/releases/tag/v0.1.5a
- Video URL (canonical, embed-this): https://github.com/Kabukich0/quaestor-demos/releases/download/v0.1.5a/phase-1.5a.mp4
- Asset size: 6,381,700 bytes (6.4 MB), 90s composition, ~42.7s narration over the front half + silent caption-only back half.

**Voice is LOCKED for the project lifetime**: ElevenLabs
"Joseff Novak — Calm and Professional", Eleven v3 model,
stability 50, similarity boost 75, speed 1.01. Same voice across every
phase video — this is brand identity, not a per-phase choice.

`narration/<phase>.txt` is the source-of-truth script. Generated audio
goes to `public/<phase>-narration.mp3` (gitignored, not committed). The
optional fallback `scripts/narrate.ts` (msedge-tts) produces lower-
fidelity audio if you don't want to round-trip ElevenLabs; production
videos use ElevenLabs.

The four op repos (core, bridge, mcp) and the demos repo all embed the
v0.1.5a release URL in their READMEs. Updating the release re-cuts
every embed — keep that in mind before re-tagging.

---

## Phase 2.0 — locked design decisions

These FOUR decisions were debated and resolved before this handoff.
Phase 2.0 implements them; it does not relitigate.

### (a) Hybrid enforcement, 0.85 threshold, per-mandate `enforcement_mode`

The policy LLM emits a `policy_score` in `[0.0, 1.0]` for every signing
request. **Hybrid enforcement**:
- Score >= 0.85 → ALLOW (proceed to per-protocol sign).
- Score < 0.85 → DENY with `error_code: "POLICY_REJECTED"`.

Hard threshold, no soft-warn middle band — softer tiers were rejected
to keep the trust boundary auditable. The 0.85 cutoff is deliberately
slightly conservative; the eval set in (4) calibrates whether to ship
0.80 or 0.90 instead, but the SHAPE is locked.

**Per-mandate `enforcement_mode` field** added to the mandate request
body and persisted on the mandate row. Three values:
- `"strict"` — apply the hybrid threshold, deny on low score.
- `"advisory"` — log the policy decision but always allow. Used for
  observation periods + eval-set generation. **NEVER the default.**
- `"off"` — skip policy entirely. Used only by the bridge's own self-
  tests + the live RUN_LIVE round-trips. **Refused on mainnet.**

Default is `"strict"`. The mode is set at issuance time and immutable
for the mandate's lifetime — you cannot relax enforcement on an
already-issued mandate.

### (b) Qwen 2.5 3B Instruct Q4_K_M (~2GB), single model, no tiering

Model: `Qwen/Qwen2.5-3B-Instruct` quantized to `Q4_K_M` GGUF
(~2GB on disk). Loaded via `node-llama-cpp` in the plugin process. No
tiering, no fallback to a larger model on uncertain scores, no cloud
inference. **One model, every decision.** This decision was load-
bearing because it determines:
- Memory budget on the user's laptop (~3GB peak with KV cache).
- Latency per decision (~150–400ms on M-series Apple Silicon, ~600–
  1200ms on x86 without acceleration). Acceptable because this is a
  single hop in front of an HTTPS round-trip the agent was already
  going to make.
- The eval set in (4) calibrates against this model and only this
  model. Re-running on a larger model invalidates the eval.

Phase 2.0 ships hardcoded Qwen 2.5 3B. Phase 2.1+ may consider
swappable models behind an interface, but not until 2.0 has run in
production for at least a release cycle.

### (c) Intent stored in core's `mandate_intent` SQLite table — NEVER in JWT

The policy LLM evaluates a structured prompt that includes:
- Mandate claims (sub, aud, amount_max, currency, network, purpose_tag,
  ttl, use_counter_max).
- The current request's protocol-specific payload (parsed value from
  the per-protocol `validatePayload`).
- **`intent_text`** — a free-form natural-language description of what
  the agent is trying to do, supplied at mandate-issuance time.

`intent_text` is THE load-bearing privacy invariant of Phase 2.0. It
contains arbitrary user-or-agent-provided text that may include
personal info, internal merchant rationale, instructions, etc.

**Storage rules — break these and the trust boundary is gone:**
- Intent is sent at `POST /mandate/request` body, persisted to a new
  `mandate_intent` SQLite table keyed by `jti`.
- Intent is **NEVER** put into the mandate JWT. JWT claims are
  unchanged from 1.5b (sub, aud, amount_max, currency, network,
  purpose_tag, jti, iat, nbf, exp, use_counter_max).
- Intent is **NEVER** logged. The audit table keeps `payload_hash`
  only; do not extend it to store intent.
- Intent is read from the table only by the policy plugin's evaluate
  hook, only inside the daemon process.
- The bridge does not see intent. Ever.
- The grep regression suite gains a new clause: every src/ in core,
  bridge, and mcp must NOT match `/intent_text/` outside of the four
  expected files (db migration, mandate_intent.ts, sign-pipeline.ts
  evaluate hook, mandate-request route). Any other match is a leak.

### (d) Separate plugin repo `@quaestor/policy` via `node-llama-cpp`

The policy code lives in a NEW repo: `Kabukich0/quaestor-policy`. NOT
inside core. Layout:

```
quaestor-policy/
  src/
    index.ts          createPolicy({ modelPath, options }) → PolicyHandle
    evaluate.ts       PolicyHandle.evaluate(args) → { score, reason }
    prompt.ts         buildPrompt(claims, payload, intent) → string
    model.ts          node-llama-cpp wrapper, Qwen-specific tuning
  test/
    eval.test.ts      drives 60-case eval set, checks rate metrics
    eval-cases.json   the 60 hand-labeled cases
  package.json        node-llama-cpp, gguf-parser
  README.md
```

Core consumes it via `file:../quaestor-policy` (1.5b pattern, same as
bridge). Phase 2.1 publishes to npm.

**Plugin must gracefully degrade when not installed.** Core's
sign-pipeline calls the policy hook through a try/import wrapper:

```ts
let policy: PolicyHandle | null = null;
try { policy = await import("@quaestor/policy").then(m => m.createPolicy(...)); }
catch (e) { /* policy not installed — log once, proceed without */ }
```

If the import fails (plugin not installed, model file missing, init
crashes), core continues to operate exactly as 1.5b. The mandate's
`enforcement_mode` is then ignored — behavior reverts to "policy off"
regardless of what the mandate row says, and a single boot-time warning
is logged. **No existing 1.5b flow may break when the plugin is
absent.**

---

## Critical context

### Integration point — where the hook lands

In Phase 1.5b, `signMandate<T>(args)` lives at
`quaestor-core/src/lib/sign-pipeline.ts` and runs an ordered pipeline:

1. Verify JWT (claims via `verifyMandate`)
2. Use-counter check
3. Mandate row lookup
4. Revocation gate (`mandate.revoked_at_ms != null` → 403)
5. Per-protocol `validatePayload(args)` — returns `value` + `value_base`
6. Spend-cap check
7. Per-protocol `sign(args)` — returns `signature` + `from`/`kid`
8. Atomic `ledger.append + decrementAndAudit`

**Phase 2.0 inserts step 4.5: `policyEvaluate(claims, value, mandate)`
between revocation gate and per-protocol validation.** That is the
correct insertion point because:
- JWT + revocation already failed-closed; we don't waste an LLM call
  on a revoked mandate.
- The protocol-specific `value` isn't yet computed, so the policy
  evaluates the request body BEFORE protocol-specific normalization.
  This keeps the prompt protocol-agnostic.
- The mandate row is in scope, so `enforcement_mode` and `intent_text`
  are both available.

The hook signature should be deliberately narrow:
```ts
interface PolicyHandle {
  evaluate(args: {
    claims: MandateClaims;
    rawBody: Record<string, unknown>;
    mandate: MandateRow;
    intent: string | null;
  }): Promise<{ score: number; reason: string }>;
}
```

Do NOT pass: the JWT itself, the auth token, the vault handle, the
network. The plugin's surface is `(claims, body, mandate, intent) →
score+reason` and nothing else.

### File paths the next session needs

**Core** (the integration point):
- `quaestor-core/src/lib/sign-pipeline.ts` — the function 2.0 hooks.
- `quaestor-core/src/lib/proto-modules.ts` — per-protocol fan-out.
- `quaestor-core/src/lib/server.ts` — route → signMandate dispatcher;
  also where `POST /mandate/request` accepts the new
  `enforcement_mode` + `intent_text` fields.
- `quaestor-core/src/lib/ledger.ts` — `EntryType` union (add
  `'policy.evaluate'` and `'policy.deny'` here when 2.0 starts logging
  decisions). Schema migrations live in `Ledger.runMigrations()`.
- `quaestor-core/src/lib/mandate-store.ts` — extend with
  `MandateRow.enforcement_mode` and a new `MandateIntentStore`.
- `quaestor-core/src/lib/mandate.ts` — JWT `MandateRequest` /
  `MandateClaims` shapes. **Do not add intent_text here.** Add
  `enforcement_mode` to the request only (it's persisted, not signed).

The 1.5b "schema migrations" pattern: `Ledger.runMigrations()` runs
idempotent `ALTER TABLE ADD COLUMN` introspections via PRAGMA
table_info. Phase 2.0 needs:
- `ALTER TABLE mandates ADD COLUMN enforcement_mode TEXT NOT NULL DEFAULT 'strict'`
- `CREATE TABLE IF NOT EXISTS mandate_intent (jti TEXT PRIMARY KEY,
  intent_text TEXT NOT NULL, created_at_ms INTEGER NOT NULL)`

**The repo does not currently have a `db/migrations/` directory.**
Migrations live inline in `Ledger.runMigrations()`. If 2.0 wants
externalised migrations, add the directory; otherwise keep the inline
pattern.

**Plugin** (new repo):
- `quaestor-policy/src/{index,evaluate,prompt,model}.ts`
- `quaestor-policy/test/eval.test.ts` + `eval-cases.json`

### Eval set — required before merging Phase 2.0

60 hand-labeled cases. Each case:
```json
{
  "claims": { "sub": "...", "aud": "...", "amount_max": 5.0, ... },
  "rawBody": { /* protocol body */ },
  "intent": "buy a domain name for the agent's project",
  "expected_decision": "allow" | "deny",
  "label_rationale": "why a human says allow/deny"
}
```

Distribution target:
- ~30 "obvious allow" (intent matches purpose_tag, amount well below
  cap, recipient plausible).
- ~15 "obvious deny" (intent contradicts purpose_tag, amount near cap
  for unusual recipient, scam-like patterns).
- ~15 "edge" (gray-area cases — these calibrate the threshold).

Acceptance gates:
- **False-approve rate <= 5%** on the obvious-deny + edge subset.
  False-approve = case labeled deny, model scored >= 0.85.
- **False-reject rate <= 10%** on the obvious-allow subset.
  False-reject = case labeled allow, model scored < 0.85.

Both rates must be measured on Qwen 2.5 3B Instruct Q4_K_M
specifically. The eval test runs in CI on the plugin repo.

### Privacy invariants — the grep regression suite

Phase 2.0 adds a fourth grep regression to the existing three:

- core `test/no-key-leak.test.ts` — already there, scans for
  private-key-shaped strings in `src/`.
- bridge `test/ap2.test.ts` "no key material leaks" — already there.
- mcp `test/no-key-material.test.ts` — already there.
- **NEW** core `test/no-intent-leak.test.ts` — scans for
  `/intent_text/` in `src/` outside the allowlist.

Allowlist for the new regression:
- `src/lib/mandate-intent.ts` (the new store)
- `src/lib/sign-pipeline.ts` (where the policy hook reads it)
- `src/lib/server.ts` (where the issuance route accepts it)
- `src/lib/ledger.ts` (where the schema lives — should mention only
  the table name, not column reads in business code)

Any other file matching `intent_text` is a leak. Test fails CI.

---

## Explicit non-goals for Phase 2.0

These were considered and deliberately deferred:

- **Model tiering** — no "small for cheap, large for uncertain" routing.
  Single model, single decision.
- **Cloud fallback** — no calls to OpenAI / Anthropic / Together / etc.
  Local LLM only. The whole point of Quaestor is local-first.
- **Fine-tuning** — no LoRA, no fine-tune. Eval set + threshold
  calibration only.
- **Telemetry** — no per-decision metrics shipped off the device.
  The audit table records `policy.evaluate` entries with score and
  decision, but they stay local.
- **Mainnet** — testnet only until first design partner. Same posture
  as 1.5b. The policy plugin REFUSES to evaluate against a mainnet
  network field until the maintainer flips a flag.
- **Streaming inference** — single-shot generation for a structured
  score. Streaming UX comes in 2.1+ if we even need it.
- **Multi-language intent** — English-only for the eval set. Multi-
  lingual evaluation is 2.2+ territory.

---

## Read me twice

- The `signMandate` pipeline is the integration point. Do not bypass it.
- `intent_text` NEVER touches the JWT. The grep regression enforces
  this.
- The plugin must gracefully degrade — if the user doesn't install it,
  core behaves exactly like 1.5b.
- The four locked decisions above are decisions, not options. If 2.0
  needs to revisit any of them, it stops and gets explicit user
  approval. Drift kills phases.
