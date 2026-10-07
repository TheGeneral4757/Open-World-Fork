# 04 — Accounts, Auth, Roles & Admin

**Invariant: the server decides who you are and what you may do on every request. The browser
never holds a password hash, never decides who is admin, and never sees a secret.**

Status: account fields and super-admin rank decided (D27–D31, amended 2026-10-07). How it ties into Gamble Limited's
existing system will be **decided in a separate integration session** (D32). Everything here is
written so the identity layer can be swapped for a shared GLL identity service later.

---

## 1. Account model (D27, matching Gamble Limited)

| Field | Required | Notes |
|---|---|---|
| Username (handle) | ✅ | Login + display. Case-insensitive unique. `[A-Za-z0-9_]{3,20}`, reserved list |
| First name, last name | ✅ | **Private by default**: visible to staff only, never in chat/leaderboards (Q140) |
| Email | ✅ | Must **end in `@<DOMAIN>`** (same rule as GLL), stored lowercase. **No verification** (D27) |
| Password | ✅ | argon2id, min 8 chars, HIBP k-anonymity check |

⚠️ **Collecting real names + emails changes the privacy picture** (this supersedes D8 "no PII"):
- **Email is a domain check only, not proof** (D27, Boss's call, same as GLL). Anyone who types an
  address ending in `@<DOMAIN>` can sign up, including with someone else's address. Accepted for a
  friends game. Consequences: emails can't be used for password resets (resets are staff-issued),
  and the super-admin decides who's real. If abuse shows up, add a code-by-email step later
  behind a feature flag.
- Privacy policy + data deletion: reuse/extend GLL's (D33).
- Encrypt backups; limit who can see names/emails in the admin panel; log every admin view of PII.

## 2. Signup / login flow

```
signup:  POST /api/auth/signup {username, firstName, lastName, email, password}
         → validate fields → email.toLowerCase().endsWith('@' + ALLOWED_DOMAIN)?  (env/settings value)
         → argon2id(password) → INSERT user (status: 'active') → session cookie
login:   POST /api/auth/login {usernameOrEmail, password}
         → rate-limit (per IP via CF-Connecting-IP + per account)
         → argon2.verify (dummy verify when the user doesn't exist, for flat timing)
         → check status/bans → new session (rotate) → Set-Cookie
reset:   staff-issued one-time reset code (admin panel) → player sets new password → revoke all sessions
         (no email sending needed; optional recovery codes shown at signup)
```
- Sessions: random 32-byte token in an `HttpOnly; Secure; SameSite=Lax` cookie; stored **hashed**
  (sha256) in `sessions`; 30-day rolling; "log out everywhere".
- Serve the client and API from the same site (`game.example.com` + `/api`) so SameSite cookies
  just work; check `Origin` on state-changing routes.
- Invite codes are optional on top of the email domain rule (Q142).

## 3. Roles (D30, D31)

```ts
type Rank = 'THE_STRONGEST' | 'admin' | 'player';   // THE_STRONGEST = super-admin rank (matches GLL)
// granular permissions under the hood, so a moderator role can be added later without
// touching route code
```

| Role | Who | Powers |
|---|---|---|
| **`THE_STRONGEST`** (super-admin rank) | **Boss only.** Username **`thestrongest`**; the rank `THE_STRONGEST` is internal (matches GLL) | Everything, including granting/revoking admin, settings, viewing the full audit log and PII |
| admin | Nobody at launch (optional later) | Player tools, chat moderation, tickets, world edits. No role changes, no PII export |
| player | Everyone else | — |

Rules:
- The super-admin is defined by **rank in the DB** (`users.rank = 'THE_STRONGEST'`), never by a
  hard-coded username check in code. Exactly one account may hold it (partial unique index).
- The username `thestrongest` goes on the reserved list so nobody else can register it or anything
  confusable with it (`the_str0ngest`, `thestr0ngest`, Unicode look-alikes).
- Super-admin login requires **TOTP 2FA** (passkey later).
- Display: players see the username `thestrongest` with a staff badge; the rank name is shown
  only inside the admin panel.

## 4. Admin panel

Separate `/admin` route in the same client, lazy-loaded, **every action enforced server-side**.

| Area | Actions |
|---|---|
| Players | search, view profile/state/history, set money/items (reason required), reset, ban/temp-ban, mute, force logout, trigger password reset, rename |
| World | release/transfer territory, feature flags, maintenance mode, broadcast notice |
| Economy | price charts, money supply, top earners, large transactions, manual price nudge |
| War | active attacks, cancel/refund |
| Alliances | view, rename, disband, remove member |
| Chat | live view, delete message, mute from message, slow mode |
| Tickets | inbox, reply (player sees it in-game), status |
| Audit | append-only log of every staff action: who, what, before/after, reason. DB role can't UPDATE/DELETE it |
| Analytics (D35) | server-side stats only: signups, DAU, actions, economy health. No third-party trackers |

Destructive bulk actions need you to type the word `CONFIRM` + TOTP re-prompt.

## 5. Chat moderation (D29: minimal)

Channels: **global** + **alliance** (D28). Minimal moderation means:
- Server-side rate limit (e.g. 1 msg / 1.5s, burst 5) and max length.
- Text rendered with `textContent` only; links shown as plain text.
- Staff can delete messages and mute users; everything is logged.
- No automatic profanity filter at launch (can be a feature flag later).
