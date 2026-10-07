# 02 — Roadmap (greenfield)

**Rule: foundations → identity → world → economy → social → war → polish.** Each phase ends in
something you can run and click. No deadline (D34); phases are ordered, not dated.

```
P0 Repo & infra ─▶ P1 Accounts & admin core ─▶ P2 World map ─▶ P3 Economy loop
      ─▶ P4 Chat & alliances ─▶ P5 Conflict ─▶ P6 Alpha with friends ─▶ P7 Polish & GLL integration
```

---

## P0 — Repo & infrastructure
- [ ] Create the private repo under a Boss GitHub org (D24); move `docs/plans` there; archive/keep this fork as reference only.
- [ ] pnpm monorepo (`apps/client`, `apps/server`, `packages/shared`, `tools/`, `deploy/`), TS strict, Biome, Vitest.
- [ ] CI: typecheck, lint, test, build; Docker image to GHCR (private).
- [ ] Proxmox VM/LXC: Docker Compose (Postgres+PostGIS, API, Caddy, cloudflared), staging + prod stacks.
- [ ] Backups: nightly `pg_dump` + WAL + **Proxmox Backup Server** snapshots (D36); first restore drill.
- [ ] Release flow mirrors GLL's (D37): auto-deploy to staging, promote to prod.

**Exit:** a hello-world API + page served through the tunnel, deployed by CI, backed up, and restored once.

## P1 — Accounts & admin core
- [ ] Signup (username, first/last name, email on the allow-list) → email verification → login/logout/sessions (see `04`).
- [ ] Password reset by email; rate limits; reserved names.
- [ ] Roles; super-admin `THE_STRONGEST` / `thestrongest` with TOTP.
- [ ] Admin shell: players list, ban/mute, audit log, feature flags, maintenance mode.
- [ ] Privacy policy/ToS pages (reused from GLL, D33).

**Exit:** friends can register and log in; you can see and manage them; every staff action is logged.

## P2 — World map
- [ ] `tools/build-world`: Natural Earth (+ EEZ / H3 water) → granularity rules → ids, neighbors, terrain, base prices (see `05`).
- [ ] Seed PostGIS; build LOD meshes for the client.
- [ ] Client globe: pooled meshes, picking, LOD switching, ownership colors, performance on a Chromebook.
- [ ] Visual style prototype(s) (Q-style, D38 = decide later).

**Exit:** a smooth real-world globe where you can click any territory and see its info.

## P3 — Economy loop
- [ ] Buy/sell territory with adjacency rules; buildings; settle-on-read production with offline cap.
- [ ] Global supply/demand market; transactions ledger; leaderboard (net worth).
- [ ] Server-side analytics tables for the admin dashboard (D35).
- [ ] Your game ideas land here: Boss's design doc drives catalogs and rules.

**Exit:** two accounts can play the core loop; nothing can be cheated from the browser.

## P4 — Chat & alliances
- [ ] Global + alliance chat over WS (D28), minimal moderation (D29), staff delete/mute.
- [ ] Alliances: create/join/leave/kick, roles, peace between members.
- [ ] Support tickets with in-game replies.

## P5 — Conflict
- [ ] Attacks with travel time, server seeds, defender warnings, protection windows; resolved by the scheduler.
- [ ] Rules designed fresh (no copied formulas, D20).

## P6 — Friends alpha
- [ ] Invite-only (allow-listed emails); load test with ~50 simulated clients; balance pass with server analytics.

## P7 — Polish & ecosystem
- [ ] GLL integration (shared accounts / branding / staff), decided in its own session (D32).
- [ ] Visual polish, notifications, quality-of-life, new features from the ideas backlog.

## Risks

| Risk | Mitigation |
|---|---|
| Accidentally copying upstream code | Clean-room rule in CLAUDE.md; code written from `docs/plans` specs only |
| Map pipeline harder than expected | Start with countries + admin-1 only; add admin-2/water later |
| PII leak (names/emails) | Email verification, encrypted backups, minimal admin exposure, audit log |
| Home outage | Best effort + PBS backups (D36); status note |
| Scope creep | Ideas backlog; only the current phase gets built |
