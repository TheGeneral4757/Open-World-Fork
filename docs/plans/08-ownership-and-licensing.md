# 08 — Ownership, Licensing & Clean-Room Rules

> ⚠️ **Not legal advice.** If this ever involves money, investors, or a dispute, talk to a lawyer.

**Invariant: ideas are free, expression is owned.** You can build a game with the same *kind*
of mechanics as anyone else's. You can't take their code, text, art, or other creative
expression without a license.

---

## 1. Decisions

| # | Decision |
|---|---|
| D18 | The new game is an original, competing game, not a rebuild of upstream |
| D19 | Owned by **Gamble Limited**; proprietary, all rights reserved |
| D20 | Clean-room: upstream ideas/mechanics only, no copied code, text, or assets |
| D21 | No upstream player data or accounts imported |
| D26 | Gamble Limited assets Boss owns may be reused |

## 2. Why upstream's code can't be reused (the correction)

| Claim | Reality |
|---|---|
| "His repo is public, so it's open source" | ❌ Public ≠ open source. **No license = all rights reserved.** GitHub's Terms let others *view and fork on GitHub*; they grant no right to copy, modify, host or relicense elsewhere |
| "It's made by AI, so it's open/public domain" | ⚠️ Unsettled. Purely AI-generated output may lack copyright, but human selection, arrangement, editing and direction can be protected, and you can't tell from outside which parts are which. Not a safe basis for a commercial product |
| "Its general principle can be used" | ✅ **Correct.** Mechanics, rules, genre, and concepts (buy land, build, trade, war, alliances, chat) are not copyrightable |

If he ever adds a license to his repo, re-check this. Even then, the clean-room approach stays
simpler for a proprietary product.

## 3. Clean-room rules for this project

1. New code is written in the **new private repo**, from specs in `docs/plans/`.
2. Specs describe **behavior in our own words** ("territories must touch your land unless it's
   your first"), never upstream's code, constants tables, or prose.
3. Don't have `index.html` open while writing new-game code. Claude follows the same rule
   (CLAUDE.md).
4. No upstream text: no release notes, UI copy, item descriptions, names like "Oceanus"/"Luna"
   planets, or emoji/stat catalogs lifted wholesale.
5. No upstream assets: images, CSS, textures, sounds.
6. Fresh numbers: balance values are designed for the new game (the real-world map changes them
   anyway).
7. Keep this repo's history as a record that the new code was written separately (a dated trail
   helps if anyone ever asks).

## 4. What Gamble Limited owns

| Item | Owner | Notes |
|---|---|---|
| All new code, docs, designs | Gamble Limited | Header: `// © <year> Gamble Limited. All rights reserved.` + root `LICENSE` stating proprietary |
| Code Claude writes in Boss's sessions | Gamble Limited (via Boss) | Anthropic's terms assign outputs to the user |
| Reused GLL assets (D26) | Gamble Limited | List each one with origin in `ASSETS.md` (Q110) |
| Third-party libraries | Their authors | Only permissive licenses (MIT/BSD/Apache/ISC) in the client bundle; **no GPL/AGPL** in proprietary code; track in `NOTICE` |
| Map data | Natural Earth (PD), others CC BY | Attribution screen in-game; **avoid GADM (non-commercial) and OSM (ODbL share-alike)** for the territory DB |
| Player data (names, emails, gameplay) | Players; Gamble Limited is custodian | Covered by the GLL privacy policy (D33) |
| Brand/name | Gamble Limited | Working pattern: "Gamble Limited's <Name>" (D39). Check the final name isn't trademarked |

## 5. Things to check

- **Is Gamble Limited a registered company or a brand name?** (Q143) This affects whether "©
  Gamble Limited" or "© <your name>, trading as Gamble Limited" is right, and whether "Limited"
  is OK to use (in many countries "Limited"/"Ltd" is reserved for registered companies).
- Contributor terms if anyone else ever contributes (default: they assign to Gamble Limited).
- Asset provenance for everything reused from GLL (Q110).
- Final name trademark search before going public (Q106).
