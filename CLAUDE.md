# CLAUDE.md — Open World (personal dev fork)

Guidance for Claude (and humans) working in this repository.

## What this repo is

**Open World** is a browser-based 3D multiplayer strategy game: claim Voronoi plots on a
3D Earth (plus Luna / Mars / Oceanus), build businesses, produce and sell materials,
form clans, chat, and conquer other players' land with marching armies.

- **Upstream (the friend's game, live):** `kayneheffelfinger-cyber/Open-World` → GitHub Pages
- **This repo:** `TheGeneral4757/Open-World-Fork` — a **personal development fork** used to
  plan and prototype a rebuild (security, self-hosted DB, possible TypeScript, real auth,
  real admin). Nothing here deploys to the friend's production site.
- **Planning docs live in [`docs/plans/`](docs/plans/README.md).** Read the index first
  before proposing architecture changes — decisions and open questions are tracked there.

## ⚠️ Hard rules (read before running anything)

1. **`index.html` and `admin.html` are hard-wired to the friend's LIVE Supabase project**
   (`SUPABASE_URL` / `SUPABASE_ANON_KEY` near `index.html:1086`, and again in `admin.html`).
   The database is world-writable (see `supabase-setup.sql`). Opening the game from this
   fork **reads and writes real players' data**.
   - Never run, test, or automate the game against that project.
   - Before any local testing, blank both constants (offline mode) or point them at a
     dev project / local stack you control. Do not commit a change that points the fork
     at someone else's database.
2. **Never exploit the live backend**, even "to prove a point". Findings go in
   `docs/plans/00-audit.md`, then to the friend privately.
3. **Never commit secrets.** The current anon/publishable key is public by design; a
   `service_role` key, DB password, session secret, or `.env` file is not. Use `.env`
   (gitignored) once a server exists.
4. **Don't push to `main` of upstream.** Work on branches in this fork; upstreaming is a
   deliberate, reviewed step coordinated with the friend.
5. **Don't touch `.github/workflows/deploy.yml` casually** — on the upstream repo it
   deploys every push to `main` straight to GitHub Pages.

## Current codebase (as of the fork point, game v1.15)

No build step, no package.json, no tests. Static files served as-is.

| File | Lines | What it is |
|---|---|---|
| `index.html` | ~7,500 | **The entire game.** CSS (≈7–804), HTML markup (≈805–1057), one ES module script (≈1069–7485) |
| `admin.html` | ~420 | Standalone owner admin panel (duplicates hashing + REST logic) |
| `supabase-setup.sql` | 40 | One table `openworld_data(key text pk, data jsonb, updated_at)` + **fully public RLS** |
| `README.md` | — | Feature list + full changelog (source of truth for what shipped) |
| `assets/`, `*.webp` | — | Menu images. Earth textures load from the three-globe CDN |
| `.github/workflows/deploy.yml` | — | GitHub Pages deploy on push to `main` |

External runtime deps (CDN, **unpinned / no SRI**): `@supabase/supabase-js@2` (jsdelivr),
`three@0.160.0` + `OrbitControls` (unpkg, via importmap).

### Module map inside `index.html` (approximate line numbers — grep to confirm)

| Area | Anchor | Notes |
|---|---|---|
| Supabase config | `const SUPABASE_URL` (~1086) | Empty strings = offline/localStorage-only mode |
| Graphics quality | `QUALITY_LEVELS` (~1095) | Auto/High/Balanced/Low + adaptive resolution |
| Global state | `const STATE` (~1235) | Money, owned set, inventory, market — **all client-authoritative** |
| Planets | `const PLANETS` (~1278) | idBase: Earth 0, Luna 10000, Mars 20000, Oceanus 30000; 2,200 plots each |
| CloudSync | `const CloudSync` (~1339) | Pull-all-rows + merge; admin stamps; tombstones |
| Save hooks | `saveGame = function` (~1600) | Monkey-patches saveGame/addProfile/saveClaims to mirror to cloud |
| Clans & pacts | `const Clans` (~1645) | Rows `openworld_clan_<id>`, `openworld_alliance_<a>__<b>` |
| Chat | `const Chat` (~2057) | Single row `openworld_chat`, 3s poll, read-merge-write |
| Wars (marches) | `const Wars` (~2304) | Single row `openworld_wars`; seeded mulberry32 deterministic battles |
| Claims | `getClaims` (~2932) | Single row `openworld_claims` = `{plotId: username}` |
| Plot color | `PLOT_COLOR_SWATCHES` (~2988) | Stored in each save |
| Save / load | `saveGame` / `loadGame` (~3159) | Save JSON shape is defined here |
| Release notes popup | `GAME_VERSION` (~3340) | **Out of sync** with README (code says v1.11) |
| Login | `hashPassword` (~3395), `loginSubmit` | DJB2 32-bit hash, compared client-side |
| Timers | `setInterval` (~3697–3750) | 10s autosave, 30s cloud pull, 60s support badge |
| Catalogs | `MATERIALS`, `BUSINESS_TYPES` (~3752–3860) | Economy tuning lives here |
| Scene/terrain | `initScene`, `buildEarth`, `classifyRegion` (~3939–4290) | Water mask + topology sampling |
| Plot generation | `generatePlots` (~4604) | Seeded spherical Voronoi; neighbors use **global** ids |
| Plot UI / buying | `openPlotModal`, `canBuyPlot` (~5155–5440) | Border-buy rule |
| My Land / sell | `sellPlotCore` (~5596) | 50% refund |
| Multi-buy | `togglePlotSelection` (~5742) | |
| Market | `updateMarket` (~6000) | **Per-player** random walk stored in each save (not truly global) |
| Casino slots | `pickSlotOutcome` (~6372) | `Math.random()` client-side |
| War UI | `attackPlot` (~6544) | Launches a march |
| Admin | `ADMIN_ACCOUNT = 'kingkanye26'` (~6624) | Client-side gate only; raw REST with anon key |
| Support tickets | `submitSupportTicket` (~7055) | Rows `openworld_support_<id>` |
| Production | `tickProduction` (~7205) | Runs from the render loop — no offline income |

### Cloud data model (one table, key → jsonb)

```
profiles                       ["name", ...]                 all usernames
openworld_save_<username>      {passwordHash, money, ownedPlotIds, businesses, inventory, marketPrices, ...}
openworld_claims               {"<plotId>": "<username>"}    global ownership map
openworld_claims_stamp         <ms>                          admin "claims rewrite" stamp
openworld_deleted              {"<username>": <ms>}          deletion tombstones
openworld_chat                 {msgs:[{ts,n,t}], cleared}
openworld_wars                 {marches:[...], truces:{plotId: until}}
openworld_clan_<id>            clan roster + log
openworld_alliance_<a>__<b>    pact state
openworld_support_<id>         support ticket
```

### Conventions the existing code follows

- Single file, vanilla JS, `window.fn = function` for anything called from inline `onclick`.
- Modules are IIFEs returning an API object (`CloudSync`, `Clans`, `Chat`, `Wars`).
- Heavy explanatory comments above every subsystem — keep that style when editing.
- Every shipped change gets a README changelog entry (`### vX.Y (YYYY-MM-DD) — Title`).
  `GAME_VERSION` + `RELEASE_NOTES` drive the in-game "What's New" popup.
- "No schema change" is a recurring design constraint in upstream — new features have
  been shoe-horned into extra jsonb rows. The rebuild intentionally drops that constraint.

## Running locally

```bash
# 1) FIRST: blank SUPABASE_URL / SUPABASE_ANON_KEY in index.html (offline mode),
#    or point them at a Supabase project / local stack you own.
# 2) Serve statically:
python3 -m http.server 8000      # or: npx serve .
# open http://localhost:8000
```

Debug knobs: `?warmarch=<seconds>` shortens war marches (min 3s) for QA.
There is no automated test suite. Verify in a browser (Playwright + Chromium are available
in cloud sessions).

## Known critical problems (summary — details in `docs/plans/00-audit.md`)

- World-readable/writable DB; anyone can edit or delete any row from the browser console.
- Password hashes are public and use a 32-bit non-cryptographic hash.
- Admin is a hard-coded username checked only in the client.
- All economy/combat/RNG runs on the client → trivially cheatable.
- Stored XSS: usernames are interpolated unescaped into `innerHTML` (leaderboard, admin table, inline `onclick`).
- Shared single-row JSON blobs (claims, chat, wars) are last-write-wins → lost updates / double-buys.

## How Claude should work here

- **Planning phase right now.** Prefer writing/updating docs in `docs/plans/` over code
  changes unless Boss asks for code. Record decisions in `docs/plans/README.md`
  (Decision Log) and strike answered items in `01-questions.md`.
- When editing `index.html`, make surgical changes; grep for the anchor names above
  rather than trusting line numbers. Keep inline comment density similar to surroundings.
- Escape anything user-controlled before `innerHTML` (there is a `clEsc()` helper at the
  Clans module — reuse it until the rebuild).
- Any new server code: TypeScript, strict mode, server-authoritative for money/inventory/
  combat/RNG, parameterized SQL only, secrets via env.
- Commit messages: imperative summary line, then body explaining *why*.
- Don't edit upstream-facing files (README changelog, deploy workflow) as part of planning work.
