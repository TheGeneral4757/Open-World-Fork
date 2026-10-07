# New Game: Planning Docs (Gamble Limited Ltd.)

Status: **Planning** · Working title: *TBD* · Last updated: 2026-10-07

> **Pivot (2026-10-07):** this is no longer a rebuild of the friend's game. Boss is building a
> **new, original, proprietary game** under Gamble Limited, inspired by the *mechanics* of
> `kayneheffelfinger-cyber/Open-World` but sharing **no code, text or assets** with it (clean-room,
> see [08](08-ownership-and-licensing.md)). Earlier decisions that assumed a rebuild are marked
> **superseded** below.

| Doc | What's in it |
|---|---|
| [00-audit.md](00-audit.md) | Analysis of the upstream game: what to learn from, what mistakes not to repeat (reference only) |
| [01-questions.md](01-questions.md) | Open questions with defaults. **Section I is the new-game questionnaire** |
| [02-roadmap.md](02-roadmap.md) | Greenfield phases from empty repo to launch |
| [03-architecture.md](03-architecture.md) | TS server + Postgres/PostGIS, data model, realtime, offline progression |
| [04-auth-and-admin.md](04-auth-and-admin.md) | Passwords, sessions, roles, admin panel |
| [05-world-map.md](05-world-map.md) | Real countries/regions/waters: data sources, granularity, rendering, politics |
| [06-self-hosting.md](06-self-hosting.md) | Proxmox + Cloudflare Tunnel, Compose, hardening, backups |
| [08-ownership-and-licensing.md](08-ownership-and-licensing.md) | Clean-room rules, what you can and can't take, Gamble Limited IP, asset provenance |
| [friend-security-summary.md](friend-security-summary.md) | Security notes Boss is sending to the upstream owner |

## The one-paragraph version

A persistent browser strategy/economy game on a real-world 3D globe: countries, subdivided
regions, and maritime sectors at mixed detail. Buy and develop territory, run businesses, trade on
one global supply/demand market, and fight. Harder and more realistic than upstream but still easy
to pick up, with offline progression. TypeScript everywhere (Vite + Three.js client, Fastify
server, Postgres + PostGIS) on Boss's Proxmox box behind Cloudflare Tunnel. Proprietary, © Gamble
Limited Ltd. Written from scratch.

## Blocking questions

Q106 (codename), Q112 (map granularity), Q118 (offline cap), Q121 (core loop: Boss's ideas doc),
Q110 (which GLL assets get reused). See `01-questions.md` §I.

## Decision log

| # | Date | Decision | Status |
|---|---|---|---|
| D1 | 2026-10-07 | This fork is reference + planning only; never touch upstream's live Supabase | ✅ active |
| D2 | 2026-10-07 | Backend hosted on Boss's own hardware | ✅ active |
| D3 | 2026-10-07 | ~~Work flows back upstream as PRs~~ | ❌ superseded by D18 |
| D4 | 2026-10-07 | Backend = own Node/TypeScript server + Postgres | ✅ active |
| D5 | 2026-10-07 | Proxmox VM/LXC + Cloudflare Tunnel | ✅ active |
| D6 | 2026-10-07 | Scale target ≤30 concurrent / ≤200 accounts at launch (friends) | ✅ active |
| D7 | 2026-10-07 | ~~Import upstream accounts via claim codes~~ | ❌ superseded by D21 |
| D8 | 2026-10-07 | ~~Username + password, no email/PII~~ | ❌ superseded by D27 |
| D9 | 2026-10-07 | ~~Roles owner / admin / player~~ | ❌ superseded by D30 |
| D10 | 2026-10-07 | TypeScript strict, server + client + shared package | ✅ active |
| D11 | 2026-10-07 | Server-authoritative economy, global supply/demand market, ~~no offline production~~, ~~casino stays~~ | ⚠️ amended by D23; casino → Q134 |
| D12 | 2026-10-07 | Boss sends the upstream owner the security summary | ✅ active (D25) |
| D13 | 2026-10-07 | ~~Upstream owner's push-to-main workflow~~ | ❌ superseded by D18 |
| D14–D16 | 2026-10-07 | ~~Ownership split with upstream owner, AGPL~~ | ❌ superseded by D19–D20 |
| D17 | — | ~~Exit plan with upstream owner~~ | ❌ moot |
| **D18** | 2026-10-07 | **New, original, competing game**, not a rebuild of upstream | ✅ |
| **D19** | 2026-10-07 | Branded **Gamble Limited Ltd.**, a fictional studio name that is also Boss's GitHub org (and runs his other game); **proprietary**, all rights reserved (legally Boss's) | ✅ |
| **D20** | 2026-10-07 | **Clean-room**: upstream ideas/mechanics only; zero copied code, text, or assets | ✅ |
| **D21** | 2026-10-07 | No upstream players, accounts or data are imported | ✅ |
| **D22** | 2026-10-07 | World = **real countries / sub-regions / water sectors**, **mixed granularity** (exact rules TBD) | ✅ (details Q112–Q117) |
| **D23** | 2026-10-07 | **Offline progression ON**; progression harder/more realistic but still easy to start | ✅ (cap Q118) |
| **D24** | 2026-10-07 | Code lives in a **new private repo in the Gamble Limited GitHub org**; Boss does the repo/git setup in a separate session | ✅ |
| **D25** | 2026-10-07 | Boss tells the upstream owner about his security risks | ✅ |
| **D26** | 2026-10-07 | Gamble Limited assets that Boss owns may be reused | ✅ (provenance list Q110) |
| **D27** | 2026-10-07 | Accounts: username + **first/last name + email**, email must **end in `@<DOMAIN>`** (like GLL), **no verification**; staff-issued password resets | ✅ (amended) |
| **D28** | 2026-10-07 | Chat channels: **global + alliance** | ✅ |
| **D29** | 2026-10-07 | Chat moderation: **minimal** (rate limits, staff delete/mute, logged) | ✅ |
| **D30** | 2026-10-07 | Ranks: `THE_STRONGEST` (super admin) / admin / player | ✅ (amended) |
| **D31** | 2026-10-07 | Super admin = Boss: username **`thestrongest`**, internal rank **`THE_STRONGEST`** (matches GLL); rank stored in DB, TOTP required | ✅ (amended) |
| **D32** | 2026-10-07 | Gamble Limited integration designed in a **separate session** | ✅ |
| **D33** | 2026-10-07 | Legal pages: reuse/extend GLL's ToS + privacy policy | ✅ |
| **D34** | 2026-10-07 | Audience = friends; **no real money ever, no ads**; team = Boss + Claude; no deadline | ✅ |
| **D35** | 2026-10-07 | Analytics: server-side only, no third-party trackers | ✅ |
| **D36** | 2026-10-07 | Availability: best effort for now; DB backups + **Proxmox Backup Server** | ✅ (details TBD) |
| **D37** | 2026-10-07 | Release flow same as/similar to GLL's | ✅ |
| **D38** | 2026-10-07 | Platform: browser, desktop-first. Visual style: decide later | ✅ |
| **D39** | 2026-10-07 | Codename for now; branding pattern "Gamble Limited's <Name>" | ✅ |
