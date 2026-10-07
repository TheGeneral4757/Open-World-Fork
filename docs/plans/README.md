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

## The one-paragraph version

The game is a single 7,500-line HTML file that trusts every browser completely: the Supabase
table is public read/write/delete, password hashes are public 32-bit DJB2, admin is a
username check in JS, and money/combat/RNG all run client-side. The plan: (0) ship tiny
upstream hotfixes, (1) wrap the client in Vite + TS and extract shared rules, (2) build a
Fastify + Postgres server that owns all rules and real auth, (3) reach feature parity,
(4) cut over to your hardware behind a Cloudflare Tunnel, (5) build admin v2 and new features.

## Blocking questions (answer these first)

Q1, Q2, Q13, Q14, Q16, Q21, Q22, Q23, Q28, Q39, Q48, Q56, Q67, Q68, Q69, Q86, Q87

## Decision log

| # | Date | Decision | Source |
|---|---|---|---|
| D1 | 2026-10-07 | Work happens in personal fork `TheGeneral4757/Open-World-Fork`; fork must never write to upstream's Supabase | Boss |
| D2 | 2026-10-07 | DB will eventually be hosted on Boss's own hardware | Boss |
| D3 | — | *(pending)* Backend: own TS server + Postgres vs Supabase | Q28, Q87 |
| D4 | — | *(pending)* TypeScript scope | Q86 |
| D5 | — | *(pending)* Legacy account migration method | Q48 |
