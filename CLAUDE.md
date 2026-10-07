# CLAUDE.md — Planning workspace for a new game (Gamble Limited)

## What this repo is (read this first)

This repository is a **public GitHub fork** of `kayneheffelfinger-cyber/Open-World`, a friend's
browser game. It is used for **two things only**:

1. **Reference.** The upstream game (`index.html`, `admin.html`, `supabase-setup.sql`) is kept
   as-is so its *ideas and mechanics* can be studied.
2. **Planning.** The docs in [`docs/plans/`](docs/plans/README.md) describe a **new, original,
   proprietary game** that Boss is building under his company **Gamble Limited**. It will be hosted
   on Boss's own hardware and set on real-world countries, regions and waters.

The new game's code will live in a **new private repo under one of Boss's GitHub organizations**
(not created yet). Nothing in this fork is the new game's code.

> Note: forks of public repos stay public. Sensitive plans should move to the private repo as soon
> as it exists.

## ⚠️ Hard rules

1. **Clean-room rule: never copy upstream code, text, CSS, or assets into the new game.**
   - Upstream has **no license**, so it is *all rights reserved* by default. Being public on GitHub
     does not make it open source, and being AI-assisted does not make it public domain.
   - Allowed: ideas, mechanics, genre conventions, and general principles (not copyrightable),
     described **in our own words** in `docs/plans/`.
   - Not allowed: pasting or "translating" functions (e.g. porting `generatePlots`, `hashPassword`,
     the war formula code, catalogs, CSS, release notes, images) into the new codebase.
   - When writing new-game code, work from the specs in `docs/plans/`, not from `index.html`.
2. **Upstream's live backend is off-limits.** `index.html` and `admin.html` point at the friend's
   LIVE Supabase project, and its database is world-writable. Never run, test, automate, read player
   data from, or exploit it. Don't serve this fork's `index.html` unless the Supabase constants are
   blanked.
3. **No upstream player data or accounts** go into the new game. No imports, no scraping, no
   "migration".
4. **Never commit secrets** (DB passwords, session secrets, API keys, `.env`).
5. Don't modify upstream files in this fork (`index.html`, `admin.html`, `README.md`,
   `.github/workflows/deploy.yml`). Planning lives in `docs/plans/` and this file.

## The new game in one paragraph (details in docs/plans)

A persistent, browser-based 3D strategy/economy game on a real-world globe. Territory is real
countries, subdivided regions, and maritime sectors, at mixed granularity. Players buy and develop
territory, run businesses, trade on a server-side global market, and fight. Progression is harder
and more realistic than upstream but still approachable, **with offline progression**. The stack is
TypeScript end to end: a Vite + Three.js client, a Fastify server, and Postgres (+ PostGIS) on
Boss's Proxmox box behind a Cloudflare Tunnel. Proprietary, © Gamble Limited.

## Plans index

See [`docs/plans/README.md`](docs/plans/README.md) for the doc map, decision log, and open questions.
Record every new decision in that log, and mark answered questions in `01-questions.md`.

## How Claude should work here

- Planning phase: prefer updating `docs/plans/` over writing code unless Boss asks.
- When referencing upstream for inspiration, describe *what it does* (behavior), not *how its code
  does it*, and never paste its code into plans meant for implementation.
- New-game code standards (once the private repo exists): TypeScript `strict`, server-authoritative
  for money, inventory, combat and RNG, parameterized SQL only, secrets via env, and a
  `// © Gamble Limited. All rights reserved.` style header per the licensing doc.
- Commit messages: imperative summary line, then a body explaining *why*.
