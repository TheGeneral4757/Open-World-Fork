# 03 — Target Architecture (greenfield)

**Core invariant: the client asks, the server decides.** Money, inventory, ownership, prices, RNG
and combat live in Postgres and change only inside server transactions. The browser renders the
world and sends *intents* ("buy FR-IDF"), never *state* ("my money is 9,000,000").

Everything here is written from scratch (D20). The upstream game is a lesson in what *not* to do
architecturally (see `00-audit.md`), not a code source.

---

## 1. System

```
Browser (render + UI)                      Boss's Proxmox box
┌──────────────────────┐  HTTPS / WSS  ┌───────────────────────────────────────────┐
│ Vite + TS client     │ ────────────▶ │ cloudflared ─▶ Caddy ─▶ API (Fastify, TS) │
│ Three.js globe (LOD) │ ◀── events ── │                          │  game loop      │
│ no secrets, no rules │               │                          ▼                 │
└──────────────────────┘               │                 Postgres 16 + PostGIS     │
                                       │   backups: pg_dump/WAL + Proxmox Backup Srv│
                                       └───────────────────────────────────────────┘
```
Static client assets can be served by Caddy on the box or by a CDN (Cloudflare caches them
either way).

## 2. Components

| Component | Choice | Job |
|---|---|---|
| Client | Vite + TypeScript strict, Three.js | Globe, panels, admin UI; holds a read model of state |
| API | Node 22 LTS + Fastify + Zod | REST actions, auth, admin |
| Realtime | `@fastify/websocket` | Push: chat, ownership changes, attacks, market ticks, notices |
| Game loop | In-process scheduler | Market ticks, attack arrivals, cleanup |
| DB | Postgres 16 + PostGIS, Drizzle migrations | Source of truth |
| Shared | `packages/shared` | Types, Zod schemas, catalogs, pure rule functions (client hints + server enforcement) |
| Edge | Cloudflare Tunnel → Caddy | TLS, no open ports, hides home IP |

One process is plenty for a friends-scale game (D6). Module boundaries keep splitting possible later.

## 3. Repo layout (new private repo, D24)

```
apps/client/      Vite app (game + /admin)
apps/server/      Fastify app (auth/, routes/, game/, admin/, db/)
packages/shared/  types, schemas, catalogs, rules/
tools/build-world/ map pipeline (see 05)
deploy/           compose files, Caddyfile, backup scripts
docs/             plans (moved from this fork)
```

## 4. Data model (first cut; game tables firm up once Boss's game ideas land)

```sql
-- identity (see 04)
users          (id uuid pk, username citext unique, display_name text, first_name text,
                last_name text, email citext unique, password_hash,
                rank text, status text, created_at, last_login_at, banned_until, muted_until)
sessions       (id, user_id, token_hash bytea unique, created_at, expires_at, ip inet, user_agent)
totp_secrets   (user_id pk, secret_enc bytea, enabled_at)

-- world (seeded by tools/build-world, see 05)
territories, territory_neighbors

-- player state
players        (user_id pk, money bigint, last_settled_at timestamptz, last_seen_at, ...)
ownership      (territory_id pk, owner_id, acquired_at, protected_until)   -- PK = no double-buy
buildings      (id, territory_id, type, level, built_at)
inventory      (user_id, item, qty numeric, pk(user_id, item))
transactions   (id, user_id, kind, amount, ref jsonb, at)                   -- money ledger

-- economy
market_prices  (item pk, price numeric, pressure numeric, updated_at)
market_history (item, price, at)

-- social / ops
alliances, alliance_members, chat_messages (channel: 'global' | 'alliance:<id>')
support_tickets, admin_audit_log (append-only), settings (feature flags, tuning)
```

## 5. Offline progression (D23): settle on read

No per-second writes. Each player stores `last_settled_at`. Whenever the player acts, or a
scheduled sweep runs, the server computes:

```
elapsed  = min(now − last_settled_at, OFFLINE_CAP)        -- cap: Q118 (default 24h)
produced = Σ buildings: rate × territory_factor × elapsed × efficiency(elapsed)
```
- `efficiency` can taper offline gains (e.g. 100% for the first 2h, then 50%) so being online
  is still worth something (Q119).
- The client runs the same function from `packages/shared` for a smooth ticking counter. The
  server's number always wins.
- "Harder, more realistic but still easy" (D23) lives in the tuning constants (`settings`
  table), not in code. Balance is changed without a deploy.

## 6. Market: one global price, supply/demand (D11)

Selling adds pressure that pushes the price down, and pressure decays over time so prices
recover. Big dumps get a worse *average* price than gradual selling. Prices are bounded (e.g.
0.5×–2× base) and get small random drift on each tick. Constants live in `settings`, and every
tick goes to `market_history`.

## 7. Action flow (buying a territory)

```
POST /api/territories/FR-IDF/buy
 → session → user → rate limit
 → BEGIN
     settle player (offline production)
     SELECT player FOR UPDATE
     rules (shared package): adjacency/first-claim, funds, restrictions
     INSERT ownership ON CONFLICT DO NOTHING   → 0 rows = 409 already owned
     UPDATE money; INSERT transactions
   COMMIT
 → 200 {player}   +  WS broadcast {t:'territory.claimed', id, owner, color}
```

## 8. Realtime protocol

```ts
type ServerEvent =
  | { t: 'territory.claimed'; id: string; owner: string; color: string | null }
  | { t: 'territory.released'; id: string }
  | { t: 'chat.msg'; id: number; channel: string; user: string; body: string; at: number; staff?: true }
  | { t: 'chat.deleted'; id: number }
  | { t: 'attack.launched' | 'attack.resolved'; attack: AttackView }
  | { t: 'market.tick'; prices: Record<string, number> }
  | { t: 'self.updated'; money: number; inventory: Record<string, number> }
  | { t: 'notice'; level: 'info' | 'warn'; text: string };
```
Actions stay REST (easy to validate, rate-limit and audit). WS is for server push + chat. On
reconnect, the client refetches `/api/world/snapshot`.

## 9. Lessons taken from upstream (behavior, not code)

| Upstream problem | Our rule |
|---|---|
| Public read/write DB | DB never reachable from the internet; only the API talks to it |
| Client-side auth/admin | Server sessions + DB roles + audit log |
| Client-side money/RNG | Server-authoritative, server RNG |
| One JSON blob per feature, last-write-wins | Normalized tables + transactions + PK constraints |
| Every client downloads the whole DB every 30s | Snapshot once, then deltas over WS |
| Unescaped usernames in innerHTML | `textContent` / escaping by default; strict CSP |
| CDN deps unpinned | Bundled, lockfile-pinned deps |

## 10. Out of scope (for now)

Redis, Kubernetes, microservices, GraphQL, event sourcing, horizontal scaling.
