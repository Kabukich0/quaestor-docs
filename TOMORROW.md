# TOMORROW — picking up after 2026-04-29 (Phase 1.3 ship + fundraising prep)

## Done

The Phase 1.3 batch — `quaestor-mcp` shipped end-to-end, repos polished for fundraising, deck + investor pipeline + outreach drafts committed. Each item listed below was committed and pushed in its own atomic commit.

### Task 1 — Green CI on `quaestor-mcp`

Recovered from yesterday's iCloud disk-space crash, fixed the un-awaited rejects test, added the GitHub Actions workflow, wrote 32 tool-level vitest tests + the architectural-invariant grep for private-key strings, wrote the in-process programmatic-mcp-client demo, and shipped Phase 1.3 STATUS.md.

Then: chased and fixed three CI failures in sequence — bridge sibling repo missing in CI runner (added clone step gated on `BRIDGE_TOKEN` secret), bridge `dist/` not shipping in pnpm 9's `file:` virtual-store entry (replaced with rsync materialization step), and the actual root cause: bridge's published `package.json` `exports."."` points at `./dist/index.js` while its `tsconfig.rootDir: "."` build emits to `./dist/src/index.js`. CI now patches the cloned bridge's package.json before build. Both bridge fixes recorded in Outstanding below as Phase 1.5 cleanup.

CI run: https://github.com/Kabukich0/quaestor-mcp/actions/runs/25125854511 — green in 55 s, 43/43 vitest passing.

`quaestor-mcp` tip of `main`: `90e857e  docs: add CI/status/Node/license badges` (after Task 6).

### Task 2 — `quaestor-core` engines pin aligned

`package.json` `engines.node` was `">=20"`, drifted from the `">=20 <23"` invariant the other two repos already had. Fixed in a single one-line commit.

`quaestor-core` commit: `98a7e6d  chore: align engines pin to >=20 <23 (matches bridge and mcp)`.

### Task 3 — Seed deck v0 outline

12-slide outline at `quaestor-docs/deck-outline.md`. Each slide names its purpose, lists the 3–5 bullets it must land, and points to the supporting artifact in the repos (or flags where production data is still needed). Includes a "Visual artifacts needed" section for the designer and a "Risks honestly disclosed" section so diligence reveals nothing the deck didn't.

`quaestor-docs` commit: `3c537c4  docs(deck-outline): seed deck v0 — 12 slides, supporting artifacts, risks disclosed`.

### Task 4 — Investor-pipeline tracker template

`quaestor-docs/investor-pipeline.md`. Three sections: Tier-1 lead candidates (Paradigm, a16z crypto, Coinbase Ventures, Conviction, Boldstart), strategic angels (Matt Huang, Cuy Sheffield, Brian Armstrong angels list, Anthropic MCP engineers, Stripe agent-team alums, Browserbase founders, Crossmint/Skyfire intel-only, Chris Hay, A100/Startup Edmonton angels), Canadian funds (A100, BCF, Garage, Inovia, Real, Luge, Panache). Every entry starts at `not contacted` — this is a tracker template, not falsified status.

`quaestor-docs` commit: `fbfa738  docs(investor-pipeline): seed-round angel and VC tracker template`.

### Task 5 — Three cold-outreach drafts

`quaestor-docs/outreach-drafts.md`. Draft A (Tier-1 VC, Conviction-style — leads with the trust-model wedge), Draft B (strategic angel — leads with the technical demo), Draft C (Canadian pre-seed — leads with founder velocity + 36-hour Phase 1 cycle). Each under 150 words, one link, concrete ask. No fabricated metrics.

`quaestor-docs` commit: `b41dc45  docs(outreach): three cold-email drafts (Tier 1 VC, strategic angel, Canadian pre-seed)`.

### Task 6 — Badges + LICENSE for all three repos

All three repos got `[![CI]][..]` + status + Node + License badges right after the title. None of the three had a LICENSE file committed; added MIT (Kabukich0, 2026) to each. `quaestor-mcp` had no README at all — wrote one with quick-start, status pointer, and link to STATUS.md.

Commits (all pushed):
- `quaestor-core`: `962aebc  docs: add CI/status/Node/license badges`
- `quaestor-bridge`: `7bd0370  docs: add CI/status/Node/license badges`
- `quaestor-mcp`: `90e857e  docs: add CI/status/Node/license badges`

## Outstanding

Carried forward — not started this batch, deliberately deferred.

- **Real-network testnet integration (Base Sepolia, Coinbase Developer Platform API key needed)** — Phase 1.5. Posts a real x402 envelope at a live facilitator, watches the accept/reject. Closes the "we're using built-in builders" gap noted in `bridge/STATUS.md`.
- **POST `/mandate/revoke` endpoint in core** — Phase 1.4. `quaestor-mcp/src/tools/revoke_mandate.ts` returns the documented 404 verbatim until this lands.
- **Iroh P2P sync** — roadmap, not committed. Multi-machine mandate sync is the headline of the Cloud-sync paid tier; until Iroh ships, single-device only.
- **Demo video for MCP (Claude Code paying for Vercel)** — recording task, not coding. Highest-leverage missing fundraising asset; called out in `deck-outline.md → Visual artifacts needed`.
- **Logo / wordmark design** — design task. "Federal Reserve meets Stripe" theme per game plan; off-white background, oxblood accent, serif headlines, mono code.
- **Push bridge `package.json` exports fix (`./dist/src/` paths, version bump to 0.1.2) to Kabukich0/quaestor-bridge** — removes the "Patch quaestor-bridge package.json exports" step from `quaestor-mcp/.github/workflows/test.yml`. Phase 1.5 cleanup.
- **Fix bridge's `.npmignore` (or remove `dist/` from `.gitignore`) so pnpm 9 packs `file:` deps with `dist` included** — removes the rsync materialize step from `quaestor-mcp` CI. Phase 1.5 cleanup.

## Next session priorities

1. **Record MCP demo video (highest leverage for fundraising).** The deck outline references it; the outreach drafts reference the demo. Until it exists, the Tier-1 VC email is weaker by half.
2. **Begin angel outreach using `outreach-drafts.md`.** Personalize the opener for each recipient; one link per email; honest claims only. Update `investor-pipeline.md` status fields as conversations move.
3. **Start real-network x402 integration.** Get a Coinbase Developer Platform API key, target Base Sepolia, exercise the bridge's x402 adapter against a live facilitator, write the regression test.

## Repo state at session end

| Repo | Latest commit | Pushed? |
| --- | --- | --- |
| `Kabukich0/quaestor-core` | `962aebc  docs: add CI/status/Node/license badges` | yes |
| `Kabukich0/quaestor-bridge` | `7bd0370  docs: add CI/status/Node/license badges` | yes |
| `Kabukich0/quaestor-mcp` | `90e857e  docs: add CI/status/Node/license badges` | yes |
| `Kabukich0/quaestor-docs` | (this commit, see below) | yes |

CI on `quaestor-mcp`: green at https://github.com/Kabukich0/quaestor-mcp/actions/runs/25125854511 (Task 1 baseline). Subsequent commits to `main` since then trigger fresh runs — verify those are also green before claiming "all green" on the README badge.

All four working trees clean, on `main`, up to date with `origin/main`.
