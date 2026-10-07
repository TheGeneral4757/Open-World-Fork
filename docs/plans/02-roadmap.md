# 02 — Roadmap (phased, each phase shippable)

**Rule: stop the bleeding first, then move the rules to the server, then move the server home.**
Polishing anything before the trust model is fixed is decorating a house with no doors.

```
Phase 0 ── Phase 1 ──── Phase 2 ─────────── Phase 3 ────────── Phase 4 ──────── Phase 5
triage     foundations  server + auth       feature parity      cutover          admin v2 &
(days)     (1–2 wks)    (core loop)         (social + war)      (go live home)   beyond
```

Estimates assume evenings/weekends with Claude doing the bulk of implementation. Adjust after Q8.

---

## Phase 0 — Triage (upstream, small, with the friend) · ~1–3 days

**Decision D12:** no hotfix PR from us. Boss sends the friend
[`friend-security-summary.md`](friend-security-summary.md); the friend decides what to patch.

- [ ] Boss sends the summary; friend warns players about password reuse.
- [ ] *(Friend's call)* the quick fixes listed in the summary.
- [ ] Ask the friend for (or take, read-only) a JSON export of the table — needed for import tests.

**Exit:** friend is informed; we have an export of current world data.

### Upstream model (D3)

Every phase lands as PRs from this fork into `kayneheffelfinger-cyber/Open-World`. Implications:
- Each PR must keep the live game working — the strangler approach in `05` is mandatory, not optional.
- The **build step** (Vite, Phase 1) and the Pages workflow change need the friend's explicit buy-in
  *before* we start — it changes how he edits the game.
- The server code lives in the upstream repo too (`apps/server`), but only Boss's box deploys it.
  The upstream Pages workflow deploys the client only.
- Keep PRs small and reviewable; the friend is the reviewer.

## Phase 1 — Foundations (this fork) · ~1–2 weeks

- [ ] Agree with the friend on the monorepo + Vite build step (D3).
- [ ] Point the fork at **offline mode** (blank Supabase constants) so nothing touches prod.
- [ ] Monorepo: `pnpm` workspaces `apps/client`, `apps/server`, `packages/shared`.
- [ ] Vite wraps the existing `index.html` unchanged (migration step 0–2 in `05`).
- [ ] CI: typecheck, Biome, Vitest, build on every PR. Protected `main`.
- [ ] `tools/export-plots.ts`: run the seeded generator for all 4 planets → `plots.json` +
      neighbors. **Golden test:** ids/neighbors match what the live client generates.
- [ ] Extract `catalog.ts` + pure rules into `packages/shared` with unit tests that pin current
      numbers (plot prices, production rates, battle odds) — these are the regression net.
- [ ] Local dev stack: `docker compose -f deploy/docker-compose.dev.yml up` → Postgres.

**Exit:** `pnpm dev` runs the same game as upstream (offline); shared rules are tested.

## Phase 2 — Server, auth, core loop · ~2–4 weeks

- [ ] Fastify server skeleton, config via env, health endpoints, Pino logging.
- [ ] Drizzle schema + migrations for identity, world, player state, economy (see `03`).
- [ ] Seed `planets`, `plots`, `plot_neighbors`, `materials` from Phase 1 exports.
- [ ] Auth: signup/login/logout/sessions/rate limits/Turnstile (see `04`).
- [ ] Endpoints: world snapshot, buy plot (+multi-buy), sell plot, build/demolish business,
      settle production (online-only, D11), sell materials, global supply/demand market, planet unlock/travel, plot color.
- [ ] WebSocket: `plot.claimed/released`, `market.tick`, `self.updated`.
- [ ] Client switches auth + core loop from CloudSync → API. Legacy cloud code disabled behind a flag.
- [ ] Server-side slots (or casino disabled until Phase 3 — Q15/Q71).
- [ ] Playwright e2e: signup → buy → build → wait → sell.

**Exit:** two browsers on the dev stack can play the core loop; editing localStorage or calling
the API directly cannot create money or plots.

## Phase 3 — Social & war parity · ~2–3 weeks

- [ ] Chat (rows, channels global + clan, server rate-limit, profanity mask, reports).
- [ ] Clans + alliances with server-enforced peace rule.
- [ ] Wars: marches, scheduler, deterministic resolution with server seeds, truces, shields,
      recall, war room events.
- [ ] Leaderboards (net worth / plots / military; all-time + weekly) as SQL views.
- [ ] Support tickets with admin reply.
- [ ] Minimal admin (players list, set money w/ reason, ban/mute, audit log) — enough to run launch.

**Exit:** feature parity with upstream v1.15 (minus anything Q15 deferred).

## Phase 4 — Cutover to your hardware · ~1 week + a launch evening

- [ ] Provision VM, Compose stack, Cloudflare Tunnel, backups — run the `06` checklist.
- [ ] Restore drill passes.
- [ ] `tools/import-legacy.ts` dry run against the latest export; review conflict log.
- [ ] Load test: 50 simulated clients (k6 / autocannon + WS) for 30 min, no errors.
- [ ] Launch plan: announce → freeze upstream (maintenance banner on old site) → final export →
      import → hand out claim codes → point Pages at new client → watch logs.
- [ ] Rollback plan: old Supabase site stays read-only for 2 weeks.

**Exit:** players are on your hardware; old backend frozen.

## Phase 5 — Admin v2, ops, and fun again · ongoing

- [ ] Full admin panel (`04` §4), TOTP for staff, economy dashboards.
- [ ] Feature flags / maintenance mode.
- [ ] Error tracking (GlitchTip), Uptime Kuma, monthly update routine.
- [ ] Finish TS migration step 8–9; delete legacy code and `admin.html`.
- [ ] Then: new gameplay (trading, clan bank, seasons, Discord notifications…) driven by Q18/Q70–Q84.

---

## Risk register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Live DB gets vandalized before cutover | Medium | High | Phase 0 fixes; regular exports |
| Friend keeps shipping features upstream during rebuild → parity target moves | High | Medium | Agree on a feature freeze date, or port weekly |
| Plot ids differ between generator export and client | Low | Critical | Golden test in Phase 1 |
| School network blocks your domain / WebSockets | Medium | High | Cloudflare (443), polling fallback, test from a school Chromebook early |
| Home outage on launch day | Low | Medium | Launch when you're home; status message on Pages |
| Scope creep (new features before parity) | High | Medium | Features go to a "Later" list until Phase 5 |
| Bus factor = you | Medium | High | `06` runbook, friend gets documented access to backups |
