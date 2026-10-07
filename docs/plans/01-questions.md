# 01 — Open Questions (answer these, Boss)

**How to use this:** every question has my **default** — the thing I'll assume if you don't
answer. Reply with `Q12: default`, `Q12: B`, or your own words. Questions marked **🚧 blocking**
change the architecture; answer those first. Answered questions move to the Decision Log in
[`README.md`](README.md).

Legend: 🚧 blocking · 🔐 security · 🎮 game design · 🛠 tech · 🏠 hosting · 👥 people/process

---

## A. People, ownership & process (Q1–Q12)

**Q1 👥🚧 — Who owns the final call on design?** Is this *your friend's* game where you're the
infra/backend person, or a co-owned project? Who breaks ties?
Default: friend owns game design; you own infra/security/backend.

**Q2 👥🚧 — Is this fork meant to merge back upstream, or become the new official version?**
(a) PRs back to `kayneheffelfinger-cyber/Open-World`, (b) fork becomes canonical and upstream
is archived, (c) permanent separate game.
Default: (b) — rebuild here, then cut over the live game to it.
✅ **Answered 2026-10-07: (a) PRs back upstream** — friend's repo stays canonical.

**Q3 👥 — Does your friend know about the security issues in `00-audit.md`?** Should I prepare a
short, non-scary summary + small hotfix PR for upstream (escape usernames, pin deps, drop the
delete policy)?
Default: yes, prepare it; you send it.
✅ **Answered: Boss tells the friend (summary in `friend-security-summary.md`); no hotfix PR from us.**

**Q4 👥 — How many people will write code?** Just you + friend + Claude? Anyone else?
Default: 2 humans + Claude.

**Q5 👥 — What's your friend's coding level / comfort with build tools?** The current code is
deliberately no-build. If we add TypeScript + Vite + a server, can he still contribute?
Default: comfortable enough; we document `npm run dev` very clearly.

**Q6 👥 — Who are the players?** Friends/classmates only, or public? Rough age range? (Affects
moderation, privacy law, and whether email collection is even appropriate.)
Default: friends/classmates, mostly teens, invite-only-ish.

**Q7 👥 — Expected player count?** Concurrent at peak, and total accounts in 6 months.
Default: ≤30 concurrent, ≤200 accounts.
✅ **Answered: small — ≤30 concurrent, ≤200 accounts.**

**Q8 👥 — Timeline / pressure?** Is there a date you want the self-hosted version live?
Default: no hard date; phased, each phase shippable.

**Q9 👥 — Where do players currently play — school Chromebooks?** (Impacts: school networks
blocking your home IP / non-443 ports / WebSockets, and performance budget.)
Default: yes, lots of Chromebooks → must work on port 443, plain HTTPS.

**Q10 👥 — Do you want a public repo or private?** The fork and/or the upstream.
Default: fork private while security holes exist upstream.

**Q11 👥 — Branching model?** Upstream had "Chromebook work clobbered" twice. Do you want PRs +
review, protected `main`, and a `dev` branch?
Default: protected `main`, feature branches, PR required, CI must pass.

**Q12 👥 — How should Claude be used going forward?** Planning only, or also implementing
phases, writing tests, reviewing friend's PRs?
Default: Claude implements phases on branches; you review + merge.

## B. Scope & priorities (Q13–Q20)

**Q13 🚧 — Rank these 1–8:** security fix · self-hosted DB · real auth · admin panel v2 ·
TypeScript · anti-cheat / server-authority · new gameplay features · performance.
Default: security → server-authority+auth → self-host DB → admin v2 → TS (alongside) → features.

**Q14 🚧 — Rebuild or incremental refactor?** (a) Keep `index.html` running and peel pieces out
gradually ("strangler"), (b) build v2 alongside and cut over once at parity.
Default: (a) for the client (keep Three.js code), (b)-style *new server* — i.e. new backend,
progressively refactored client.

**Q15 — Must v2 keep feature parity with v1.15 at cutover?** Or can some features (casino,
planets, clans) return later?
Default: parity for core loop (plots, businesses, market, war, chat, clans); casino can lag.

**Q16 🚧 — Do existing accounts and progress carry over?** Or fresh "season 1" wipe at cutover?
Default: carry over money/plots/businesses; force password reset (see Q48).
✅ **Answered: carry over progress + claim codes.**

**Q17 — Is a full world reset ever acceptable (seasons)?**
Default: yes — seasons are a feature, not a failure (see Q75).

**Q18 — Any features you already know you want that aren't in the game?** (List freely.)
Default: none assumed.

**Q19 — Anything in the current game you or your friend *dislike* and want gone?**
Default: none assumed.

**Q20 — Mobile support target?** Phones, tablets, or desktop/Chromebook only?
Default: Chromebook/desktop first; tablets usable; phones "works but not great."

## C. Hosting & infrastructure (Q21–Q38)

**Q21 🏠🚧 — What hardware?** CPU, RAM, disk (SSD/HDD), OS. Is it a dedicated box, a Proxmox
VM/LXC, a NAS, a Pi?
Default: Proxmox VM, 2 vCPU / 4 GB RAM / SSD, Debian 12 or Ubuntu 24.04.
✅ **Answered: Proxmox VM/LXC** (specs still TBD).

**Q22 🏠🚧 — Host only the DB, or the whole backend (API + game server + DB)?** "Host the DB on my
hardware" with the game server elsewhere means DB traffic over the internet — not recommended.
Default: whole backend on your hardware; static client can stay on GitHub Pages or move too.
✅ **Answered: whole backend on Boss's hardware.**

**Q23 🏠🚧 — How will players reach it?** Port-forward + dynamic DNS, Cloudflare Tunnel, Tailscale
Funnel, or a cheap VPS reverse-proxying to home?
Default: **Cloudflare Tunnel** — no open ports, hides home IP, free TLS, works on school networks.
✅ **Answered: Cloudflare Tunnel.**

**Q24 🏠 — Do you own a domain?** Which one? Subdomain plan (e.g. `play.`, `api.`, `admin.`)?
Default: you'll buy/use one; `play.example.com` + `api.example.com`.

**Q25 🏠 — Home internet upload speed and reliability?** Any CGNAT? Data caps?
Default: ≥20 Mbps up, no CGNAT issues (Tunnel sidesteps CGNAT anyway).

**Q26 🏠 — Acceptable downtime?** If your house loses power or ISP, the game is down. OK?
Default: yes for a friends' game; status page optional.

**Q27 🏠 — UPS on the server?**
Default: unknown → recommend one; Postgres hates sudden power loss less than you'd think, but still.

**Q28 🏠🚧 — Postgres self-managed, or self-hosted Supabase?** Self-hosted Supabase is ~10
containers (Kong, GoTrue, PostgREST, Realtime, Storage, Studio…) — heavy for one box. Plain
Postgres + our own TypeScript server is lighter and more controllable.
Default: **plain Postgres 16 + our own TS server**. (Option: keep Supabase Cloud for now, move later.)
✅ **Answered: plain Postgres + own TS server.**

**Q29 🏠 — Docker / Docker Compose OK?** Or do you prefer bare systemd services?
Default: Docker Compose (postgres, api, caddy/cloudflared, backups).

**Q30 🏠 — Backups:** where to? (Second disk, NAS, Backblaze B2, another friend's machine.) How
much history?
Default: nightly `pg_dump` + WAL archiving to a NAS + weekly off-site to B2; keep 14 daily / 8 weekly.

**Q31 🏠 — Restore drills:** willing to test restoring once per phase?
Default: yes — an untested backup is a rumour.

**Q32 🏠 — Do you already run monitoring (Uptime Kuma, Grafana, Prometheus)?**
Default: Uptime Kuma for uptime; structured JSON logs; Grafana later.

**Q33 🏠 — Separate staging/dev environment?** A second DB + server on the same box for testing?
Default: yes — `dev` compose stack on different ports/DB.

**Q34 🏠 — Where does the static client live?** GitHub Pages (free, CDN) vs served from your box.
Default: keep GitHub Pages / Cloudflare Pages for the client; only API on your hardware.

**Q35 🏠 — Is a cheap VPS ($5/mo) acceptable as a fallback or relay?**
Default: not needed if using Cloudflare Tunnel.

**Q36 🏠 — Email sending?** Needed only for password resets / verification. Have an SMTP
provider (Resend, Postmark, Gmail app password)?
Default: no email at first (see Q47); admin-issued reset codes.

**Q37 🏠 — Any other services already on that box** that could conflict (ports 80/443/5432)?
Default: assume a reverse proxy may already exist; we'll integrate.

**Q38 🏠 — Who has SSH/root on the server?** Just you?
Default: just you; friend gets admin panel access, not shell.

## D. Authentication & accounts (Q39–Q55)

**Q39 🔐🚧 — Login identifier:** username only, email + username, or "Sign in with Google /
Discord"?
Default: **username + password**, optional email later; Discord OAuth as phase-2 nice-to-have.
✅ **Answered: username + password, no email/PII.**

**Q40 🔐 — OAuth providers you'd want, if any?** Google (school accounts may block it),
Discord, GitHub, Microsoft?
Default: Discord only, optional.

**Q41 🔐 — Password policy:** minimum length? Breached-password check (HIBP k-anonymity)?
Default: min 8, max 128, no composition rules, HIBP check on signup.

**Q42 🔐 — Hashing:** argon2id OK? (Requires a native module or WASM in Node.)
Default: argon2id (m=19 MiB, t=2, p=1 per OWASP).

**Q43 🔐 — Sessions:** httpOnly cookie sessions (server-stored) vs JWT?
Default: **opaque session token in httpOnly, Secure, SameSite=Lax cookie**, stored hashed in DB.
No JWTs needed for a single backend.

**Q44 🔐 — Session length / "remember me"?**
Default: 30-day rolling session; "log out everywhere" button.

**Q45 🔐 — Multiple simultaneous logins** (Chromebook at school + PC at home)?
Default: allowed; same account, server is source of truth so no conflicts.

**Q46 🔐 — 2FA?** TOTP for admins only, or everyone optional?
Default: **TOTP required for admin/mod roles**, optional for players.

**Q47 🔐 — Password reset without email:** admin generates a one-time reset code you hand to the
player? Or recovery codes at signup?
Default: both — 8 recovery codes shown at signup + admin-issued one-time reset link.

**Q48 🔐🚧 — Migrating existing accounts:** the old hashes are public and weak. Options:
(a) force everyone to set a new password via an admin-issued claim code,
(b) accept old password once then rehash (risky: anyone can find a DJB2 collision),
(c) wipe accounts.
Default: **(a)** — import saves, lock accounts, friend/you hand out claim codes.
✅ **Answered: (a) claim codes.**

**Q49 🔐 — Username rules:** allowed characters, length, case-insensitive uniqueness, reserved
names (admin, mod, system), profanity filter?
Default: `[A-Za-z0-9_]{3,20}`, case-insensitive unique, reserved list, basic profanity filter.

**Q50 🔐 — Can users rename themselves?** How often?
Default: once per 30 days, old name held for 30 days.

**Q51 🔐 — Account deletion by the user themself?** (GDPR-style "delete my data".)
Default: yes, with 7-day grace period; plots released.

**Q52 🔐 — Rate limiting on login/signup:** per IP and per account. Behind Cloudflare, use
`CF-Connecting-IP`. OK?
Default: 5 failed logins / 15 min per account, 20 / 15 min per IP; captcha (Turnstile) after that.

**Q53 🔐 — CAPTCHA:** Cloudflare Turnstile on signup?
Default: yes on signup, conditional on login.

**Q54 🔐 — Multiple accounts per person (alts):** allowed, tolerated, or banned?
Default: tolerated but no alt-feeding (transfers between accounts disallowed / flagged).

**Q55 🔐 — Age / privacy:** if players are under 13 (US) / under 16 (EU), collecting emails gets
legally interesting. Collect nothing personal?
Default: collect no PII — username + password only; privacy note on signup.

## E. Admin, moderation & roles (Q56–Q66)

**Q56 🔐🚧 — Roles:** just `admin` + `player`, or `owner / admin / moderator / player`?
Default: `owner`, `admin`, `moderator`, `player` (permission-based under the hood).
✅ **Answered: owner / admin / player (no moderator tier).**

**Q57 — Who are the admins at launch?** Friend (owner) + you (admin)?
Default: friend = owner, you = admin.

**Q58 — Admin powers needed?** (current: set money, reset, zero inventory, delete, reset all,
clear claims, disband clan, clear chat, support tickets). Add: ban/mute, edit plots, give items,
rollback a player, impersonate (view-as), broadcast announcement, toggle maintenance mode?
Default: all of the above except impersonate (view-only "inspect" instead).

**Q59 — Ban types:** temp ban, permanent ban, chat mute, shadow-mute? IP bans?
Default: temp/perm ban, chat mute; no IP bans (school NAT = everyone shares an IP).

**Q60 🔐 — Audit log:** every admin action recorded with who/when/before/after, viewable by
owner, not deletable by admins?
Default: yes, append-only.

**Q61 — Admin panel location:** inside the game UI, separate `/admin` app, or both?
Default: separate `/admin` route of the same client app, role-gated by the **server**.

**Q62 — Player reports** (report a chat message/player) in addition to support tickets?
Default: yes, right-click/long-press a chat message → report.

**Q63 — Chat moderation:** profanity filter (mask vs block), link blocking, slow mode?
Default: mask profanity, block links, per-user rate limit server-side, admin slow-mode toggle.

**Q64 — Support tickets:** reply-to-player capability (in-game inbox)? Statuses?
Default: yes — open / in-progress / resolved / won't-fix, with an admin reply the player sees.

**Q65 — Should admin actions be visible to players?** (e.g. "An admin adjusted your account: reason …")
Default: yes, with a required reason field.

**Q66 — Economy dashboards for admins?** (money supply over time, top earners, inflation.)
Default: yes, phase 4 — simple charts from DB aggregates.

## F. Game design & rules (Q67–Q85)

**Q67 🎮🚧 — Server-authoritative economy:** OK that the server decides money/production/prices
and the client only displays + requests actions? (Required to stop cheating.)
Default: yes, absolutely.
✅ **Answered: yes (implied by server backend).**

**Q68 🎮🚧 — Offline production:** should businesses keep producing while players are offline?
Capped (e.g. max 8 hours of storage)?
Default: yes, capped at 12h of production per business ("storage full").
✅ **Answered: NO offline production** — businesses only produce while the owner is online (see `03` §5).

**Q69 🎮🚧 — Market model:** (a) one truly global market with shared prices, (b) global prices
driven by supply/demand (selling lowers price), (c) keep per-player random walk.
Default: **(b)** — global prices, server tick, selling pressure moves price, slow recovery to base.
✅ **Answered: (b) one global market with supply/demand pricing.**

**Q70 🎮 — Player-to-player trading** (direct trades, auction house, money transfers)?
Default: not at launch; design for it later (and anti-alt-feeding rules from Q54).

**Q71 🎮 — Casino:** keep it? Server RNG with published odds? Daily wager limits? (Kids +
gambling mechanics = something your friend should decide consciously.)
Default: keep, server-side RNG, daily wager cap, clearly shows odds.
✅ **Answered: keep casino** (server RNG; wager cap still TBD).

**Q72 🎮 — War timing:** keep 5/8-minute marches and 10-minute truces? Offline defense —
should plots be attackable while the owner is offline?
Default: keep timings; attackable offline (that's the point of armies), with "new player shield" for 48h.

**Q73 🎮 — New-player protection:** shield duration, can't be attacked until X plots / Y hours?
Default: 48h or until they attack someone, whichever first.

**Q74 🎮 — Battle resolution:** keep the current formula (±15% luck, terrain, air bonus)?
Default: keep, move to server verbatim; tune later with data.

**Q75 🎮 — Seasons / resets:** periodic world resets with a hall of fame?
Default: no seasons at launch; design schema so a season id can be added.

**Q76 🎮 — Plot count:** 2,200 per planet — keep? (Grid is deterministic from the seed; changing
it reshuffles every plot id.)
Default: keep exactly — existing ownership depends on it.

**Q77 🎮 — Leaderboard:** what counts — money, net worth (money + land + businesses + stock),
plots, military? Weekly boards?
Default: net worth (primary) + plots + military; all-time + weekly.

**Q78 🎮 — Clans:** keep 20 members / 3 pacts / $50k? Clan bank / shared treasury? Clan chat?
Default: keep limits; add clan chat; clan bank later.

**Q79 🎮 — Chat channels:** global only, or global + clan + DMs?
Default: global + clan at launch; DMs later (moderation burden).

**Q80 🎮 — Notifications:** in-game only, or browser push / Discord webhook for "you're under attack"?
Default: in-game + optional Discord webhook per clan later.

**Q81 🎮 — Real-time:** is 1–3s latency fine, or do you want instant (WebSockets)?
Default: WebSockets for chat/war/claims events; polling fallback.

**Q82 🎮 — Planets:** keep Luna/Mars/Oceanus as-is, more planets planned?
Default: keep; data-driven so new planets are a config row.

**Q83 🎮 — Selling plots:** keep 50% refund? Can you sell a plot with businesses on it?
Default: keep current behavior.

**Q84 🎮 — Anti-snowball:** upkeep costs, taxes on large empires, diminishing returns?
Default: not at launch; collect data first (Q66).

**Q85 🎮 — In-game currency purchases / monetization ever?**
Default: never (and if ever, that's a very different legal conversation).

## G. Tech stack & code (Q86–Q100)

**Q86 🛠🚧 — TypeScript:** client, server, or both? Strict mode?
Default: both, `strict: true`, shared types package.
✅ **Answered: server + client, strict.**

**Q87 🛠🚧 — Server framework:** Fastify, Hono, Express, NestJS? Runtime Node or Bun?
Default: **Node 22 LTS + Fastify** (mature, fast, great TS + schema validation). Hono is the runner-up.
✅ **Answered: Node/TS server** (framework default Fastify unless objected).

**Q88 🛠 — DB access:** Drizzle ORM, Kysely, Prisma, or raw SQL?
Default: **Drizzle** (typed, SQL-shaped, real migrations, no heavy engine).

**Q89 🛠 — Validation:** Zod / TypeBox for request schemas shared with the client?
Default: Zod, schemas in the shared package.

**Q90 🛠 — Realtime transport:** raw `ws`, Socket.IO, or Server-Sent Events?
Default: `ws` via `@fastify/websocket` with a tiny typed message protocol.

**Q91 🛠 — Client build:** Vite + TS, keep Three.js? Any UI framework (React/Svelte/Preact/Lit),
or vanilla TS with modules?
Default: **Vite + vanilla TS modules** first (lowest-churn port of existing DOM code); revisit
a UI lib (Preact or Svelte) for panels later.

**Q92 🛠 — Monorepo layout** with npm/pnpm workspaces: `apps/client`, `apps/server`, `packages/shared`?
Default: yes, pnpm workspaces.

**Q93 🛠 — Testing:** Vitest (unit), Playwright (e2e), DB tests against a throwaway Postgres?
Default: all three; e2e covers login → buy → build → sell → attack.

**Q94 🛠 — Lint/format:** ESLint + Prettier, or Biome?
Default: Biome (one tool, fast).

**Q95 🛠 — CI:** GitHub Actions running typecheck, lint, tests, build on every PR?
Default: yes.

**Q96 🛠 — Deploy:** how does new server code reach your box? Manual `git pull && docker compose
up -d`, GitHub Actions → SSH, or Watchtower pulling images from GHCR?
Default: Actions builds image → GHCR → you run `docker compose pull && up -d` (later: automated).

**Q97 🛠 — Plot geometry on the server:** regenerate the Voronoi grid in Node (same seeded code),
or export it once to a `plots` table?
Default: export once to a `plots` table (id, planet, centroid, neighbors, area, region, isWater, coastal).
The server never needs the meshes.

**Q98 🛠 — Textures/CDN:** keep loading Earth textures from three-globe's CDN, or self-host them?
Default: self-host (version-pinned, no third-party outage risk).

**Q99 🛠 — Error tracking:** Sentry (self-hosted GlitchTip?) for client + server errors?
Default: GlitchTip self-hosted later; structured logs now.

**Q100 🛠 — Feature flags / maintenance mode:** server-driven flags so you can disable casino or
war without a deploy?
Default: yes — simple `settings` table read by the server and pushed to clients.

## H. Wildcards (Q101–Q105)

**Q101 — Analytics:** any (privacy-friendly, self-hosted Plausible/Umami) or none?
Default: none at launch.

**Q102 — Localization:** English only?
Default: English only, but no hard-coded strings in new server error messages.

**Q103 — Accessibility:** keyboard nav, colorblind-friendly plot colors?
Default: colorblind-safe default palette, keyboard shortcuts for main panels.

**Q104 — Licensing:** the repo has no LICENSE. Who owns the code? Open-source it?
Default: add "All rights reserved" until your friend decides.

**Q105 — Name/branding:** keep "Open World"? (Very generic; also clashes with the genre name.)
Default: keep for now.
