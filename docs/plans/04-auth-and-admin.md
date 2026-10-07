# 04 — Auth, Sessions, Roles & Admin Plan

**Invariant: identity and permissions are decided by the server on every request. The
browser never holds a password hash, never decides who is admin, and never sees a secret.**

---

## 1. Bad → Better → Best

| | Password storage | Login check | Admin check |
|---|---|---|---|
| **Today** | DJB2 32-bit, public in every save | Client compares hashes | `CURRENT_USER === 'kingkanye26'` in JS |
| Stop-gap (Phase 1, Supabase) | Supabase Auth (bcrypt) | Supabase Auth | RLS + `role` claim, admin via RPC `security definer` |
| **Target** | argon2id, server-only table | Server, constant-time, rate-limited | `role`/permissions in DB, checked per route + audited |

## 2. Signup / login flow (target)

```
signup:  POST /api/auth/signup {username, password, turnstileToken}
         → validate username rules, HIBP k-anon check, Turnstile verify
         → argon2id(password) → INSERT users + players (starting money)
         → create session → Set-Cookie: ow_session=<random 32B>; HttpOnly; Secure; SameSite=Lax; Path=/
         → return { user, recoveryCodes[8] }   (shown once)

login:   POST /api/auth/login {username, password}
         → rate-limit (per IP + per username) → lookup → argon2.verify (dummy verify if no user,
           to keep timing flat) → check banned_until → new session (rotate) → cookie

request: cookie → sha256(token) → sessions row (not expired) → user → role
logout:  DELETE session row, clear cookie.   "Log out everywhere": delete all user sessions.
```

Details:
- Session tokens stored **hashed** (sha256) so a DB leak doesn't hand out live sessions.
- Rolling expiry: 30 days, refreshed on use at most once per hour.
- CSRF: SameSite=Lax + require `Content-Type: application/json` + check `Origin` on state-changing
  routes. If the client is on a different domain than the API (Pages vs your box), use
  `SameSite=None; Secure` + strict CORS allow-list + Origin check, or (simpler) proxy the API
  under the same site (`play.example.com/api`).
- Error messages: "Invalid username or password" (don't reveal which).

## 3. Roles & permissions

```ts
type Role = 'owner' | 'admin' | 'moderator' | 'player';
const PERMS = {
  owner:     ['*'],
  admin:     ['players.view', 'players.edit', 'players.ban', 'world.edit', 'clans.edit',
              'chat.moderate', 'tickets.manage', 'settings.edit', 'audit.view'],
  moderator: ['players.view', 'players.mute', 'chat.moderate', 'tickets.manage'],
  player:    [],
} as const;
```
- Only `owner` can grant/revoke `admin`. Owner can't be demoted by admins.
- Every admin route: `requirePerm('players.edit')` + **reason required** + audit log write in
  the same transaction as the change.
- Admin/mod accounts **must** have TOTP enabled before admin routes respond (Q46).
- Optional hardening: admin routes only reachable via Tailscale / Cloudflare Access (Q61).

## 4. Admin panel v2 — feature list

| Area | Actions |
|---|---|
| Players | search, view (money, inventory, plots, businesses, sessions, history), set money (with reason), grant/remove items, reset, ban/temp-ban, mute, force logout, issue password-reset code, rename |
| World | release plot, transfer plot, clear planet, feature flags, maintenance mode, broadcast notice |
| Economy | price chart per material, money supply over time, top earners, recent large transactions, manual price nudge |
| War | active marches, cancel/refund a march, set truce/shield |
| Clans | view, rename, disband, remove member |
| Chat | live view, delete message, slow mode, mute user from message |
| Tickets & reports | inbox, assign, reply, status |
| Audit | filterable log (actor, action, target, before/after diff) — owner-only for full view |

Every destructive action = confirm dialog showing the diff; bulk actions ("reset all") require
typing the word `RESET` and owner role.

## 5. Migrating existing accounts (Q48 default = claim codes)

1. Export all `openworld_save_*` rows from Supabase (read-only, using the public endpoint —
   no writes to production).
2. `tools/import-legacy.ts`: create `users` with `password_hash = NULL`, `legacy_import = true`,
   import money/plots/businesses/inventory; resolve claim conflicts with the same
   "first alphabetical wins" rule upstream uses, log every conflict.
3. Generate one **claim code** per account (random 10 chars, stored hashed, 30-day expiry).
4. Friend distributes codes privately (Discord DM / in person).
5. `POST /api/auth/claim {username, code, newPassword}` → sets argon2 hash, clears flag.
6. Unclaimed accounts after 30 days: plots released (configurable).

Never accept the old DJB2 hash as proof of identity — collisions are trivially computable.

## 6. Stop-gap if Phase 1 stays on Supabase

- Enable **Supabase Auth** (username → synthetic email `name@players.invalid`, email confirmations off).
- New table `profiles(id uuid = auth.uid(), username, role)`.
- RLS: players may `select` public fields of everyone, `update` nothing directly.
- All game mutations via `security definer` Postgres functions (`buy_plot(plot_id)`, `sell(material, qty)`)
  that check `auth.uid()`.
- Admin functions check `(select role from profiles where id = auth.uid()) in ('owner','admin')`.
- Remove the anonymous insert/update/delete policies on `openworld_data`.

This is real work for a temporary state — only do it if the self-hosted server is > ~2 months away.
