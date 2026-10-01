# Open World 🌍

An interactive 3D Earth you can explore from space down to the surface — and buy plots of land on.

**Play it live:** https://kayneheffelfinger-cyber.github.io/Open-World/

## What it does

- 🌍 Realistic 3D Earth with NASA Blue Marble textures, bump map, and atmospheric glow
- ✨ 6,000-star space background
- 🖱️ Drag to rotate, scroll to zoom from space all the way down to the surface
- 🟢 480 buyable plots distributed across the globe — invisible from space, fade in as you zoom below ~14,000 km altitude
- 🪙 Click any visible plot → buy modal with name, lat/lon, price
- 💰 Start with $1,000,000 — owned plots turn gold and stay visible from any altitude
- 📊 HUD with budget counter, owned counter, altitude meter, and control hints
- 🎮 Cinematic menu screen with Play / Settings (preserves the original landing aesthetic)

## How to play

1. Open the live site above (or open `index.html` in any modern browser)
2. Click **PLAY**
3. **Scroll** to zoom in — keep zooming until green plot squares appear
4. **Drag** to rotate the globe and find a plot you like
5. **Click** a green plot → review the details → click **BUY PLOT**
6. Watch it turn gold — your land is yours forever (until refresh, since progress is in-memory for now)

## Tech

- [Three.js r160](https://threejs.org/) loaded via [importmap](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/importmap) — no build step
- Single-file `index.html` — no bundler, no npm install, no dependencies to maintain
- Earth textures from [three-globe](https://github.com/vasturiano/three-globe) CDN
- Auto-deploys to GitHub Pages on every push to `main` via `.github/workflows/deploy.yml`

## Running locally

This is a static site — no server needed. Just open `index.html` in a browser.

If you want a local dev server with hot reload (optional):

```bash
npx serve .
# or
python3 -m http.server 8000
```

## Roadmap

Ideas for future work:

- [ ] Persist owned plots to `localStorage` so progress survives reloads
- [ ] Plot income — owned plots generate passive revenue over time
- [ ] Dynamic plot subdivision — smaller, pricier plots appear as you zoom closer
- [ ] Lat/lon or city-name search bar
- [ ] Country / region border overlay for orientation
- [ ] Ambient space drone + click sound effects
- [ ] Plot resale market with fluctuating prices
