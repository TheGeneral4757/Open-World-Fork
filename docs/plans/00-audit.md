# 00 — Upstream Audit (reference only)

> **Read this as a list of lessons, not a porting guide.** Since D18/D20 the new game is written
> from scratch. Nothing here should be copied into the new codebase. The "tell the friend"
> list in §6 is what Boss is passing on to the upstream owner (D25).

Fork point: upstream v1.15, commit `f50ea29`.

> **Bottom line:** the game is genuinely impressive for a no-build single file — the
> Voronoi tessellation, pooled draw calls, deterministic war seeds and the changelog
> discipline are all solid work. But the **trust model is "every client is the server."**
> The database is fully public, auth is decorative, and every rule (money, combat, RNG,
> admin) is enforced only by the code the player is running. For a game among friends
> that's survivable; the moment it's on your hardware with real passwords, it isn't.

Severity: 🔴 Critical · 🟠 High · 🟡 Medium · ⚪ Low / hygiene

---

## 1. Security

### 🔴 S1 — Database is world-readable *and* world-writable
`supabase-setup.sql` creates RLS policies `using (true)` / `with check (true)` for
**select, insert, update and delete**. The anon key ships in the page (fine on its own —
it's meant to be public), so anyone can, from the browser console:

```js
// conceptually — anyone can do this with the published key:
supabase.from('openworld_data').delete().neq('key', '')            // wipe the world
supabase.from('openworld_data').upsert({ key: 'openworld_save_X', data: {...money: 1e15} })
```

Impact: total data loss, arbitrary money/plots, account takeover, chat/ticket spam.
RLS is only as good as its policies; these policies say "yes" to everyone.

### 🔴 S2 — Password hashes are public, and the hash is DJB2 (32-bit)
- Every save row contains `passwordHash`; every client downloads every save on each 30s pull.
- `hashPassword()` (`index.html` ~3395, duplicated in `admin.html`) is DJB2 → 2³² outputs,
  no salt, microseconds per guess. A collision for any account can be found by brute force
  in seconds-to-minutes on a laptop. Short real passwords can be **recovered outright**.
- Login compares hashes **client-side** — so you don't even need a collision; you can edit
  localStorage or just write the save row directly (S1).
- ⚠️ If any player reused a real password (kids do), that password should be considered
  exposed. **Tell the friend.**

### 🔴 S3 — Admin is a username string checked in the browser
`ADMIN_ACCOUNT = 'kingkanye26'`; gates are `if (CURRENT_USER !== ADMIN_ACCOUNT)`.
The admin "verification lock" re-checks the same public DJB2 hash. The admin functions
(`adminUpsertRow`, `adminDeleteRow`) are plain REST calls with the public anon key —
anyone can perform any admin action without ever touching the panel. The admin username
is also printed in the README and `admin.html`.

### 🔴 S4 — Stored XSS via usernames
Usernames are interpolated **unescaped** into `innerHTML`:
- Leaderboard: `` `<div class="lb-name">${entry.name}...` `` (`renderLeaderboard`)
- Admin table: `${name}` and `onclick="adminResetOne('${name}')"` (attribute + JS-string injection)

`maxlength="20"` is client-side only; S1 lets anyone create a save with any name, e.g.
`<img src=x onerror=...>`. That script then runs in **every player's** browser that opens the
leaderboard — and in the admin's. Chat is safe (uses `textContent`); clan names use `clEsc()`.
There are 34 `innerHTML` sites total; each needs review.

### 🟠 S5 — Signup can overwrite an existing account
Signup checks `getSave(name)` against **localStorage only**. If the device hasn't pulled
(offline at boot, fresh browser before the first sync completes, or cloud read fails), a
new signup with an existing name is pushed via upsert and can replace the real player's
row (newest `savedAt` wins the merge).

### 🟠 S6 — `saveGame()` can silently reset a password to `"default"`
```js
passwordHash: existing ? existing.passwordHash : hashPassword('default'),
```
If the local save disappears mid-session (tombstone purge, storage cleared, quota error),
the next 10-second autosave writes a save whose password is `default` and pushes it.

### 🟡 S7 — Unpinned third-party scripts, no SRI
`@supabase/supabase-js@2` floats on jsdelivr; a compromised/broken minor release ships
straight into the game. No Content-Security-Policy.

### 🟡 S8 — Client-only rate limits
Chat cooldown (1.5s) and support cooldown (60s, in localStorage) are UI courtesy, not limits.

---

## 2. Game integrity (cheating)

### 🔴 G1 — Everything is client-authoritative
| System | Where it runs | Cheat |
|---|---|---|
| Money / inventory | `STATE` → localStorage → cloud | Edit save, reload |
| Production | `tickProduction()` in render loop | Speed up clock / edit |
| Market prices | `updateMarket()` with `Math.random()` **per player** | Set your own prices |
| Slots | `pickSlotOutcome()` with `Math.random()` | Always jackpot |
| Plot purchase / border rule | `canBuyPlot()` | Write `openworld_claims` directly |
| Battles | Any client resolves marches and writes *both* players' saves | Fabricate outcomes |
| Clan peace rule | `attackPlot` checks client-side | Skip the check |

There is no fix for this inside the current architecture. It requires a server that owns
the rules (see `03-architecture.md`).

### 🟡 G2 — "Global market" isn't global
`marketPrices` / `marketTrend` / `lastMarketUpdate` are saved **per player**, and each
client runs its own random walk. Two players see different prices. README says "global."
→ Design decision needed (Q in `01-questions.md`).

---

## 3. Data consistency & scale

### 🟠 D1 — Single shared JSON rows are last-write-wins
`openworld_claims`, `openworld_chat`, `openworld_wars`, `profiles` are each one row that
many clients read-modify-write. The "read-merge-write" pattern narrows but doesn't close
the race; there are no transactions or row versions. Consequences: two players can buy the
same plot; chat messages or marches can vanish; `profiles` can drop names.

### 🟠 D2 — Every client downloads the entire database every 30 seconds
`pullAndMerge()` does `select('key,data')` with no filter: every save (incl. hashes), every
clan, chat, wars, support tickets. Cost grows as players × rows. Fine at 10 players, painful
at 100, unworkable at 1,000. Plus chat polls every 3s and wars every 4s per client.

### 🟡 D3 — Conflict logic has grown a lot of special cases
`adminStamp`, `claims_stamp`, tombstones, `_freezeSaves`, `created` vs deletion time… each
patch fixed a real bug (the changelog shows admin edits "reverting" twice), but they're all
symptoms of D1 + G1. A server with a real DB makes almost all of it disappear.

### 🟡 D4 — No offline progression
Production only runs while the tab is open and `requestAnimationFrame` fires (hidden tabs
throttle it to ~0). A server-side "settle on read" model gives offline income for free —
if you want it (design question).

---

## 4. Code health

| # | Finding | Sev |
|---|---|---|
| C1 | 7,500-line single file; ~6,400 lines of JS in one module | 🟡 |
| C2 | 84 `window.*` globals for inline `onclick` handlers | 🟡 |
| C3 | Logic duplicated between `index.html` admin and `admin.html` | ⚪ |
| C4 | `GAME_VERSION = 'v1.11'` while README is at v1.15; two `v1.12` changelog entries | ⚪ |
| C5 | Monkey-patching `saveGame = function(){...}` to bolt on cloud sync | ⚪ |
| C6 | Four separate `createClient()` instances (CloudSync, Clans, Chat, Wars) | ⚪ |
| C7 | No tests, no lint, no types; regressions found by players | 🟡 |
| C8 | Comments in `updateMarket` contradict the code (±20% vs ±8%, 5 vs 15 min) | ⚪ |
| C9 | Two "Chromebook work clobbered" restore commits → workflow/branching problem upstream | 🟡 |

## 5. What's genuinely good (keep it)

- **Deterministic seeded plot generation** — plot ids are stable across clients; a server can
  regenerate the same grid (or we export it once to a static JSON/DB table).
- **Pooled render meshes** (4 draw calls for 2,200 plots) and adaptive quality — real perf work.
- **Deterministic war resolution via seeds** — ports cleanly to server-side.
- **Changelog discipline** — the README is an excellent spec of intended behavior.
- **Chat uses `textContent`** — the right instinct, just not applied everywhere.

## 6. Immediate "tell the friend" list (independent of the rebuild)

1. Players should not reuse real passwords; assume the hashes are public.
2. Escape usernames in the leaderboard and admin table (S4) — 10-minute fix, high value.
3. Fix the `hashPassword('default')` fallback (S6).
4. Pin `supabase-js` to an exact version.
5. At minimum, drop the public **delete** policy and make admin actions go through a
   Supabase RPC/Edge Function that checks a server-side secret — a stop-gap until the rebuild.

(Do these as a polite PR / message to the friend — they're his production users.)
