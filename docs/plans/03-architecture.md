# 03 — Target Architecture (proposal, pending answers in `01-questions.md`)

**Core invariant: the client asks, the server decides.** Money, inventory, ownership,
prices, RNG and battles live in Postgres and change only inside server transactions.
The browser renders the world and sends *intents* ("buy plot 1234"), never *state*
("my money is now 9,000,000").

---

## 1. Today vs target

```
TODAY
  Browser (all rules, all state) ──anon key──▶ Supabase table openworld_data
       ▲                                        (one jsonb row per thing,
       └──── pulls EVERY row every 30s ─────────  public read/write/delete)

TARGET
  Browser (render + UI)                     Your hardware
  ┌──────────────────┐   HTTPS / WSS    ┌──────────────────────────────────────┐
  │ Vite + TS client │ ───────────────▶ │ cloudflared ─▶ Caddy ─▶ API (Fastify)│
  │ Three.js globe   │ ◀── events ───── │                         │  game loop │
  │ no secrets       │                  │                         ▼            │
  └──────────────────┘                  │                   Postgres 16        │
     served by Pages/CDN                │                   backups ─▶ NAS/B2  │
                                        └──────────────────────────────────────┘
```

## 2. Why our own server instead of "Supabase but with better RLS"

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| **A. Keep Supabase Cloud, fix RLS + Supabase Auth + RPC functions** | Smallest change; no hardware | Game rules in PL/pgSQL; doesn't satisfy "my hardware"; Realtime limits on free tier | Good **stop-gap** (Phase 1) |
| **B. Self-hosted Supabase** | Same APIs as today | ~10 containers, heavy RAM, painful upgrades, still PL/pgSQL for rules | Not worth it |
| **C. Postgres + own TypeScript server** | Rules in TS (shared with client), full control, light footprint, easy to test | You run & secure it | **Recommended** |

## 3. Components

| Component | Choice (default) | Job |
|---|---|---|
| Client | Vite + TypeScript, Three.js r160, vanilla TS modules | Render, input, panels; holds a *read model* of state |
| API | Node 22 + Fastify + Zod | REST for actions, auth, admin |
| Realtime | `@fastify/websocket` | Push events: chat, claims, marches, price ticks, admin notices |
| Game loop | In-process scheduler (same Node process at first) | Market ticks (global supply/demand, D11), march arrivals, cleanup |
| DB | Postgres 16 + Drizzle migrations | Source of truth |
| Auth | Own implementation: argon2id + opaque session cookies | See `04-auth-and-admin.md` |
| Edge | Cloudflare Tunnel → Caddy | TLS, no open ports, hides home IP, works on school wifi |
| Backups | `pg_dump` nightly + WAL-G/pgBackRest | See `06-self-hosting.md` |

> Single process is fine for ≤ a few hundred players. Don't build microservices for a
> friends' game. The schema and module boundaries are what make splitting possible later.

## 4. Data model (first cut)

```sql
-- identity
users            (id uuid pk, username citext unique, password_hash text, role text,
                  created_at, last_login_at, banned_until timestamptz null, muted_until null,
                  plot_color text null, legacy_import boolean default false)
sessions         (id uuid pk, user_id fk, token_hash bytea unique, created_at, expires_at,
                  ip inet, user_agent text)
recovery_codes   (user_id fk, code_hash text, used_at null)
totp_secrets     (user_id pk fk, secret_enc bytea, enabled_at)

-- world (static, imported once from the seeded generator)
planets          (id text pk, name, cost bigint, price_mult numeric, id_base int, water text)
plots            (id int pk, planet_id fk, lat, lon, area_mult numeric, size_tier text,
                  region text, is_water bool, coastal bool, base_price bigint)
plot_neighbors   (plot_id fk, neighbor_id fk, primary key (plot_id, neighbor_id))

-- player state (mutable, server-owned)
players          (user_id pk fk, money bigint, current_planet fk, net_worth bigint,
                  shield_until timestamptz, updated_at)
player_planets   (user_id fk, planet_id fk, unlocked_at)               -- unlocked worlds
plot_ownership   (plot_id pk fk, owner_id fk, acquired_at, truce_until null)  -- PK = no double-buy
businesses       (id bigserial pk, plot_id fk, type text, built_at, last_settled_at)
inventory        (user_id fk, material text, qty numeric(20,4), primary key(user_id, material))

-- economy
materials        (id text pk, base_price bigint, military bool)
market_prices    (material fk pk, price numeric, trend text, updated_at)
market_history   (material fk, price, at)                              -- charts + admin dashboards
transactions     (id bigserial, user_id, kind text, amount bigint, ref jsonb, at)  -- ledger

-- war
marches          (id uuid pk, attacker_id, defender_id, plot_id, kind, committed int,
                  seed bigint, terrain_bonus numeric, launched_at, arrive_at,
                  status text, outcome jsonb null)

-- social
clans            (id uuid pk, name citext unique, tag citext unique, motto, owner_id, created_at)
clan_members     (clan_id fk, user_id fk unique, joined_at, role)
alliances        (clan_a, clan_b, status text, proposed_by, created_at)
chat_messages    (id bigserial pk, channel text, user_id fk, body text, created_at, deleted_at null)

-- ops
support_tickets  (id, user_id, type, body, status, admin_reply, created_at, updated_at)
reports          (id, reporter_id, target_user_id, chat_message_id null, reason, status)
admin_audit_log  (id bigserial, actor_id, action text, target text, before jsonb, after jsonb,
                  reason text, at)                                     -- append-only (REVOKE UPDATE/DELETE)
settings         (key text pk, value jsonb)                            -- feature flags, maintenance mode
```

Key wins over today:
- `plot_ownership.plot_id` **primary key** → two players can't own one plot; buy is
  `INSERT ... ON CONFLICT DO NOTHING` inside a transaction that also debits money.
- Money is `bigint` cents/dollars — no float drift. Inventory `numeric`.
- Chat is rows, not one blob → no lost messages, real moderation.
- Admin log is append-only and stored server-side.

## 5. Economy: "settle on read" (no per-second DB writes)

Don't tick production every second for every player. Store `last_settled_at` and compute:

```
produced = rate × area_mult × region_bonus × min(now − last_settled_at, CAP)
```

**D11: no offline production.** Production only accrues while the owner has a live session.
Implementation: track `players.online_since` (set on WS connect / first API call) and clamp the
window to `[max(last_settled_at, online_since), min(now, last_seen_at + 60s)]`. On WS disconnect
(or 60s without a heartbeat) settle once and clear `online_since`. Multiple tabs/devices = still
one window, not double production.

Settle (write inventory + bump `last_settled_at`) whenever the player does anything that
reads or spends inventory (sell, attack, view market) or every N minutes while connected.
The client runs the *same formula* locally (shared package) purely for a smooth ticking
counter — the server number wins on every response.

Troop housing cap and military resources follow the same pattern.

## 5b. Global supply/demand market (D11)

One price per material, shared by everyone, stored in `market_prices`.

```
on sell(material, qty):
  price_paid  = integrate price over the sale (large sales get a worse average price)
  pressure   += qty / liquidity[material]          # liquidity = tuning constant per material
  price       = base × clamp(1 − pressure_term, 0.5, 2.0)

every tick (e.g. 60s):
  pressure   *= decay                               # price recovers toward base
  small random drift (±1–2%) so the market isn't static
  broadcast market.tick
```
- Bounds stay 0.5×–2× base, as today.
- Selling in one big chunk is penalised vs selling over time, so whales can't crash a price and
  dump in one click.
- `market_history` stores every tick for charts and admin dashboards.
- Tuning constants live in `settings`, so they can be changed without a deploy.

## 6. Action flow example — buying a plot

```
Client                          Server (one DB transaction)
  │ POST /api/plots/1234/buy      │
  │ ─────────────────────────────▶│ 1. auth: session → user
  │                               │ 2. rate-limit
  │                               │ 3. SELECT player FOR UPDATE
  │                               │ 4. rule: first plot on planet OR borders owned (plot_neighbors)
  │                               │ 5. rule: planet unlocked, money ≥ price
  │                               │ 6. INSERT plot_ownership ON CONFLICT DO NOTHING → 0 rows? 409
  │                               │ 7. UPDATE players money -= price; INSERT transactions
  │                               │ COMMIT
  │ ◀──── 200 {money, plot} ──────│ 8. WS broadcast {type:'plot.claimed', plotId, owner, color}
```

Every rule that exists in `canBuyPlot`, `buildTerrainRule`, `attackPlot`, etc. moves
into `packages/shared/rules/*.ts` so client (for UI hints) and server (for enforcement) use
identical code.

## 7. Wars on the server

- `POST /api/marches` validates troops, peace rule (clan/alliance), truce, shield, max 3 marches;
  debits troops; inserts march with a **server-generated** seed.
- A scheduler (`setInterval` 1s + on-boot catch-up) picks `status='marching' AND arrive_at <= now()`
  with `FOR UPDATE SKIP LOCKED`, resolves with the existing formula, writes both players,
  ownership, businesses, truce, outcome; broadcasts reports.
- Reboots are safe: arrivals that happened while down resolve on boot in `arrive_at` order.

## 8. Realtime protocol (typed, tiny)

```ts
// packages/shared/protocol.ts
type ServerEvent =
  | { t: 'plot.claimed';   plotId: number; owner: string; color: string | null }
  | { t: 'plot.released';  plotId: number }
  | { t: 'chat.msg';       id: number; channel: string; user: string; body: string; at: number }
  | { t: 'chat.deleted';   id: number }
  | { t: 'march.launched'; march: MarchView }
  | { t: 'march.resolved'; march: MarchView }
  | { t: 'market.tick';    prices: Record<MaterialId, number> }
  | { t: 'self.updated';   money: number; inventory: Inventory }   // admin edit, conquest, etc.
  | { t: 'notice';         level: 'info' | 'warn'; text: string };
```
Client actions stay REST (easier to rate-limit, validate, and log). WS is server→client
push + chat send. On reconnect the client refetches `/api/world/snapshot`.

## 9. Bandwidth comparison

| | Today | Target |
|---|---|---|
| Boot | Every row in the DB | `/api/world/snapshot`: ownership map (≈ plotId→ownerId, few KB gzipped) + own player state |
| Steady state | Full DB every 30s + chat row every 3s + wars row every 4s **per client** | Only deltas pushed over one WebSocket |
| Password hashes sent to clients | All of them | None. Ever. |

## 10. Client structure (after migration)

```
apps/client/src/
  main.ts                 boot, router (menu / world / admin)
  net/api.ts              typed fetch wrapper (cookies, errors)
  net/socket.ts           WS with reconnect + backoff
  state/store.ts          read model (player, ownership, prices) + event reducer
  scene/                  globe.ts, plots.ts (pools/picking), planets.ts, quality.ts, stars.ts
  ui/                     hud.ts, plot-modal.ts, market.ts, wars.ts, clans.ts, chat.ts, casino.ts, settings.ts
  admin/                  admin views (lazy-loaded, server-gated)
packages/shared/src/
  catalog.ts              MATERIALS, BUSINESS_TYPES, PLANETS, REGION_BONUSES
  rules/                  pricing.ts, production.ts, border.ts, war.ts (pure functions)
  protocol.ts, schemas.ts (Zod)
apps/server/src/
  app.ts, auth/, routes/, game/ (economy, market, wars, scheduler), admin/, db/ (drizzle schema + migrations)
tools/
  export-plots.ts         runs the seeded Voronoi generator → plots.json / SQL seed
  import-legacy.ts        Supabase rows → new schema
```

## 11. Explicitly out of scope (for now)

Horizontal scaling, Redis, Kubernetes, microservices, GraphQL, event sourcing.
Each is a fine answer to a problem this game doesn't have yet.
