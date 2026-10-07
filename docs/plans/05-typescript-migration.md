# 05 — TypeScript Migration Plan (client)

**Invariant: the game stays playable after every step.** We move code out of `index.html`
into typed modules one slice at a time; nothing is rewritten "from scratch" unless it is being
replaced by a server call anyway.

---

## 1. Is TypeScript worth it here?

Yes — but for the *right* reason. Not "TS is modern," but:
- **Shared rules**: `packages/shared` types + pure functions used by client *and* server.
  That's the single biggest payoff — one definition of plot price, production, battle math.
- **The save/state shape** is currently implicit across ~40 functions; typing it catches the
  exact class of bug the changelog keeps fixing (local vs global plot ids, v1.15).
- **Refactor safety** for a 6,400-line module.

Cost: a build step (Vite). Your friend loses "edit index.html, push, done." We replace it with
`pnpm dev` (hot reload) and CI deploys — arguably *faster* once set up.

## 2. Step-by-step

| Step | Change | Shippable? |
|---|---|---|
| 0 | Add Vite with `index.html` as entry, **zero code changes**; `vite build` → `dist/`; Pages deploys `dist/` | ✅ identical game |
| 1 | Move `<style>` → `src/styles/*.css` (split by screen) | ✅ |
| 2 | Move the `<script type="module">` body → `src/legacy/main.js` verbatim; import three from npm (pinned) instead of unpkg; supabase-js pinned | ✅ |
| 3 | Turn on `allowJs` + `checkJs` with `// @ts-nocheck` on the legacy file; add `tsconfig` strict for new files | ✅ |
| 4 | Extract **pure** pieces first (no DOM, no globals): `catalog.ts` (MATERIALS, BUSINESS_TYPES, PLANETS, REGION_BONUSES), `rng.ts` (mulberry32), `pricing.ts`, `war-math.ts` (`estimateWinChance`, terrain/air bonus) → `packages/shared` | ✅ |
| 5 | Extract scene: `quality.ts`, `stars.ts`, `globe.ts`, `voronoi.ts` (`generatePlots`), `plot-pools.ts` | ✅ |
| 6 | Replace `window.fn` + inline `onclick` with `addEventListener` / event delegation per panel (kills the 84 globals and the attribute-injection XSS class) | ✅ |
| 7 | Introduce `net/api.ts` + `state/store.ts`; switch subsystems from CloudSync to the new server **one at a time**: auth → claims/buy → businesses/production → market → chat → clans → wars → admin | ✅ each |
| 8 | Delete `CloudSync`, `Clans`, `Chat`, `Wars` legacy modules and `admin.html` | ✅ |
| 9 | Remove `@ts-nocheck`; legacy folder empty | 🎉 |

## 3. Typing the core shapes (first files to write)

```ts
// packages/shared/src/types.ts
export type PlotId = number & { readonly __brand: 'PlotId' };   // global id (idBase + index)
export type PlanetId = 'earth' | 'luna' | 'mars' | 'oceanus';
export type MaterialId = 'food' | 'ore' | 'goods' | 'tourism' | 'oil' | 'fun' | 'capital' | 'tech'
                       | 'troops' | 'ships' | 'planes';
export type BusinessId = 'farm' | 'mine' | 'factory' | 'resort' | 'oilrig' | 'casino' | 'bank'
                       | 'techhub' | 'barracks' | 'navalyard' | 'airfield';
export type Region = 'tropical' | 'temperate' | 'polar' | 'mountain' | 'highland' | 'ocean-deep' | 'arctic-ocean';

export interface PlotMeta {
  id: PlotId; planet: PlanetId; lat: number; lon: number;
  areaMultiplier: number; sizeTier: 'cottage' | 'standard' | 'estate' | 'province' | 'territory';
  region: Region; isWater: boolean; coastal: boolean; neighbors: PlotId[];
}
```
The branded `PlotId` makes the v1.15 bug (local index used where a global id was expected) a
**compile error**.

(Verify exact business/region ids against `BUSINESS_TYPES` / `REGION_BONUSES` when extracting.)

## 4. Tooling

- `tsconfig.base.json`: `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `moduleResolution: bundler`.
- Biome for lint + format. Vitest for shared rules. Playwright smoke test that boots the
  client against a seeded dev server.
- Three.js types from `@types/three` matching r160 (or upgrade three deliberately later — not
  during the migration).

## 5. Things NOT to do during migration

- Don't upgrade Three.js, change plot count/seed, or retune the economy in the same PR as a
  move. Moves are moves; changes are changes.
- Don't introduce a UI framework mid-migration. Decide after step 6 (Q91).
