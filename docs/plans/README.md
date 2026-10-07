# Open World — Planning Docs (personal dev fork)

Status: **Planning** · Fork point: upstream v1.15 (`f50ea29`) · Last updated: 2026-10-07

| Doc | What's in it |
|---|---|
| [00-audit.md](00-audit.md) | What the current code does well, and every security / integrity / scale problem found |
| [01-questions.md](01-questions.md) | **105 open questions** with defaults — answer these to unblock decisions |
| [02-roadmap.md](02-roadmap.md) | Phases 0–5, exit criteria, risk register |
| [03-architecture.md](03-architecture.md) | Target system: TS server + Postgres, data model, realtime protocol |
| [04-auth-and-admin.md](04-auth-and-admin.md) | Passwords, sessions, roles, admin panel v2, legacy account migration |
| [05-typescript-migration.md](05-typescript-migration.md) | Step-by-step move from single `index.html` to Vite + TS without breaking the game |
| [06-self-hosting.md](06-self-hosting.md) | Home-hardware topology, Compose sketch, hardening, backups, monitoring |
| [07-owner-workflow.md](07-owner-workflow.md) | How the owner keeps pushing to GitHub to update the live game, before and after the transition |
| [friend-security-summary.md](friend-security-summary.md) | Short, friendly write-up of the security issues to send the game's owner |

## The one-paragraph version

The game is a single 7,500-line HTML file that trusts every browser completely: the Supabase
table is public read/write/delete, password hashes are public 32-bit DJB2, admin is a
username check in JS, and money/combat/RNG all run client-side. The plan: (0) brief the friend on the
security issues, (1) wrap the client in Vite + TS and extract shared rules, (2) build a
Fastify + Postgres server that owns all rules and real auth, (3) reach feature parity,
(4) cut over to your hardware behind a Cloudflare Tunnel, (5) build admin v2 and new features.

## Blocking questions (answer these first)

Still open: **Q1, Q13, Q14** (+ Q21 specs, Q24 domain, Q71 wager cap). Everything else on the old blocking list is answered below.

## Decision log

| # | Date | Decision | Source |
|---|---|---|---|
| D1 | 2026-10-07 | Work happens in personal fork `TheGeneral4757/Open-World-Fork`; fork must never write to upstream's Supabase | Boss |
| D2 | 2026-10-07 | DB will eventually be hosted on Boss's own hardware | Boss |
| D3 | 2026-10-07 | Work flows back upstream as reviewed PRs; friend's repo stays canonical | Q2 |
| D4 | 2026-10-07 | Backend = own Node/TypeScript server + plain Postgres; whole backend on Boss's hardware | Q22, Q28, Q87 |
| D5 | 2026-10-07 | Host = Proxmox VM/LXC; ingress = Cloudflare Tunnel | Q21, Q23 |
| D6 | 2026-10-07 | Scale target: ≤30 concurrent, ≤200 accounts | Q7 |
| D7 | 2026-10-07 | Existing progress carries over; passwords re-set via admin-issued claim codes | Q16, Q48 |
| D8 | 2026-10-07 | Sign-in = username + password only, no email/PII | Q39 |
| D9 | 2026-10-07 | Roles = owner / admin / player (no moderator tier) | Q56 |
| D10 | 2026-10-07 | TypeScript on server + client, strict, shared rules package | Q86 |
| D11 | 2026-10-07 | Economy: server-authoritative, ONE global market with supply/demand pricing, NO offline production, casino stays (server RNG) | Q67–Q69, Q71 |
| D12 | 2026-10-07 | Phase 0 = Boss sends friend the security summary; no hotfix PR from us | Q3 |
| D13 | 2026-10-07 | Owner keeps "push to main = live" workflow; client via Pages, server auto-pulls images from GHCR on Boss's box | Boss |
