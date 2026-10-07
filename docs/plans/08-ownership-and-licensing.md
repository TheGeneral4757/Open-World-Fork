# 08 — Ownership & Licensing

> ⚠️ **Not legal advice.** This is a planning document between friends. If money, a company, or
> serious disagreement ever shows up, get a real lawyer. Both of you are likely minors, which
> means a written agreement is a *record of intent*, not an iron-clad contract. That's still
> worth having.

**Invariant: write it down while you're still friends.** Ownership questions are free to answer
now and expensive to answer after a falling-out.

---

## 1. Decisions so far (2026-10-07)

| # | Decision |
|---|---|
| D14 | **The game belongs to the owner** (kayneheffelfinger-cyber): name, concept, design, the existing code, and final say on the game. |
| D15 | **Boss owns what Boss builds** (server/backend, infra code, tooling, his own docs) and keeps the copyright. The game gets a license to use it. |
| D16 | Boss's code is licensed under **AGPL-3.0**. |
| D17 | Exit plan (what happens to hosting if you part ways): **undecided**, tracked as an open risk. |

## 2. Who owns what (proposed map)

| Thing | Owner | Notes |
|---|---|---|
| Game name "Open World", concept, design, art direction | Friend | D14 |
| Existing code (everything up to fork point `f50ea29`) | Friend | He wrote it (with whatever AI tools he used) |
| Client code changes the friend writes | Friend | |
| Client code changes Boss writes | **Q-L4** | See §4. Default: Boss keeps copyright, license to friend (same as D15) |
| `apps/server/**`, `deploy/**`, `tools/**` written by Boss | Boss, AGPL-3.0 | D15, D16 |
| `packages/shared/**` (game rules used by BOTH client and server) | **Conflict, see §3** | |
| Planning docs (`docs/plans/**`) | Boss | |
| Player data in the database | Players' data, with the game as custodian | Boss *hosts* it but doesn't own it (Q-L10) |
| Hardware, Proxmox box, home network | Boss | |
| Domain, Cloudflare account, GHCR images | **Q-L7** | Whoever's name is on the account controls it |
| GitHub repo `Open-World` | Friend | Boss's fork is Boss's |
| Supabase project (current backend) | Friend | |
| Third-party: Three.js (MIT), supabase-js (MIT), NASA Blue Marble textures (public domain), three-globe textures (check license) | Their authors | Keep attributions; `home-screen.webp` / menu art origin unknown (Q-L12) |

## 3. The AGPL snag (and the fix)

```
packages/shared  (rules: prices, production, combat)
      │                        │
      ▼                        ▼
apps/client  ──bundled──▶  browser          apps/server ──▶ your box
(friend's, closed)                          (Boss's, AGPL)
```

If `packages/shared` is AGPL, the friend's client bundles AGPL code. The *whole client* then has
to be AGPL-compatible, which contradicts "the game license is his."

**Options**

| | Option | Effect |
|---|---|---|
| A ✅ | **`packages/shared` under MIT** (or dual MIT/AGPL), server stays AGPL | The client can use it freely, the server stays protected. Cleanest. |
| B | Shared stays AGPL, and Boss gives the friend a **written extra permission** to use it in the closed client | Works, but it's a custom license exception to keep track of |
| C | Whole repo AGPL | Simplest legally, but the friend's game becomes open source. Only if he wants that |

**Recommendation: A.** Also note that the server and client talking over HTTP/WebSocket is fine.
They're separate programs, and AGPL doesn't spread across a network API.

**What AGPL means in practice for Boss's server:** anyone may use, modify and run it, but if they
run a modified version for users over a network, they must offer those users the source. That
includes the friend if he ever runs his own modified copy. It stops someone from taking your
backend and running a closed rival.

## 4. Open questions (answer in a later session)

**Q-L1 — Does the friend agree to D14–D16?** None of this counts until he says yes in writing (a
signed `OWNERSHIP.md` in his repo is enough). Default: Boss shows him this doc.

**Q-L2 — Shared rules package license:** Option A (MIT), B (permission), or C (all AGPL)?
Default: A.

**Q-L3 — License for the friend's own code/client:** all rights reserved, or does he want it
open too? Default: all rights reserved (his call).

**Q-L4 — Client code Boss writes** (TypeScript migration, auth screens, fixes): Boss keeps
copyright + license to friend, or assigned to friend since it's "the game"? Default: assigned to
friend for client code, so the game stays his in one piece. Boss keeps the server.

**Q-L5 — Code the friend writes inside `apps/server`:** who owns it? Default: contributions to a
directory follow that directory's license (AGPL), with copyright kept by the author.

**Q-L6 — Claude-written code:** who counts as the author? (Under Anthropic's terms the output
belongs to the user who prompted it, so code Claude writes in Boss's sessions = Boss's.) Whether
AI-written code can be copyrighted at all is legally unsettled. Default: treat it as the
prompting person's contribution.

**Q-L7 — Accounts:** whose name is on the domain, the Cloudflare account, the GHCR images, and the
Discord (if any)? Default: domain + Cloudflare = Boss (it's his infra), but the friend gets
documented access and a transfer promise in the exit plan.

**Q-L8 — Can the friend relicense or sell the game?** If he does, Boss's AGPL server goes with it
under AGPL terms, unless he negotiates a separate license with Boss. Is that OK?

**Q-L9 — Can Boss reuse his server code in other projects?** Under D15, yes. Confirm the friend
is fine with that.

**Q-L10 — Player data:** who is responsible for it (privacy, deletion requests, breaches)? Boss
hosts it, the game owns the relationship with players. Default: joint, with Boss handling
technical security and the friend handling player-facing decisions.

**Q-L11 — Exit plan (D17 open):** if you part ways: (a) Boss hands over a DB export + X days'
notice, (b) infra stays Boss's and the friend rebuilds hosting, (c) Boss keeps running it for
some time. Also: does the friend get a license to keep running Boss's server code? (Under AGPL he
automatically can.)

**Q-L12 — Asset provenance:** where did `assets/home-screen.webp` and `open-world-menu.webp` come
from (AI-generated, drawn, downloaded)? Downloaded images may not be usable.

**Q-L13 — Credits:** how is each person credited (README, in-game credits screen, About page)?
Default: in-game credits, "Created by <friend> · Backend & infrastructure by <Boss>".

**Q-L14 — Money:** if the game ever makes money (donations, ads), how is it split? And who pays
hosting costs (domain ~$10/yr, power, backups)? Default: no money; Boss covers his own hardware
and the friend covers the domain if he wants it in his name.

**Q-L15 — Third contributors:** if someone else contributes, do they sign anything? Default: a
one-paragraph contributor note in `CONTRIBUTING.md` (inbound = outbound licensing).

**Q-L16 — Parents:** should a parent/guardian see the agreement? Default: yes, a good idea if either
of you is under 18. It costs nothing and makes it more meaningful.

## 5. Next steps

1. Boss reviews this doc and answers Q-L2…Q-L16 in a later session.
2. Draft `OWNERSHIP.md`: a plain-English, one-page version both of you sign/approve by commit.
3. Add license files **only after the friend agrees**:
   - `apps/server/LICENSE`, `deploy/LICENSE`, `tools/LICENSE` → AGPL-3.0 (copyright Boss)
   - `packages/shared/LICENSE` → per Q-L2
   - root `LICENSE` → friend's choice (Q-L3)
   - SPDX headers (`// SPDX-License-Identifier: AGPL-3.0-only`) at the top of Boss's server files.
4. Add a `NOTICE`/credits file listing third-party assets and their licenses.
