# Open World 🌍

An interactive 3D Earth strategy game — claim land, build businesses, produce materials, and trade on a fluctuating global market. Compete with other players on the leaderboard.

**Play it live:** https://kayneheffelfinger-cyber.github.io/Open-World/

## Features

### 🌍 3D Earth
- Realistic globe with NASA Blue Marble textures, bump map, atmospheric glow
- 6,000-star animated space background
- Drag to rotate, scroll to zoom from space down to the surface
- 2,200 plots covering the entire globe via **spherical Voronoi tessellation** — non-overlapping, irregular polygonal cells that fit together perfectly

### 🔐 Accounts & Persistence
- **Login system** with username + password (separate Login and Create Account tabs)
- Passwords hashed (DJB2-style) and stored in localStorage
- **Full save/load**: money, owned plots, businesses, inventory, market prices — all persisted to localStorage
- Auto-save every 10 seconds + on buy/build/sell/demolish/logout
- Multiple users can have separate saves on the same browser

### 🏞️ Plots
- **Non-overlapping Voronoi cells** — every plot is a unique irregular polygon (4-12 sides)
- **5 size tiers**: cottage, standard, estate, province, territory
- **Terrain detection** via water mask + topology (elevation) textures:
  - 7 regions: 🌴 Tropical, 🌳 Temperate, ❄️ Polar, ⛰️ Mountain, 🏔️ Highland, 🌊 Deep Ocean, 🧊 Arctic Ocean
- **Shared ownership**: plots bought by one player are locked for everyone else (global claims registry)
- Other players' plots show in **muted purple** on your globe

### 🏗️ Businesses
- **8 business types**, each producing a different material:
  - 🌾 Farm → Food | ⛏️ Mine → Ore | 🏭 Factory → Goods | 🏖️ Resort → Tourism
  - 🛢️ Oil Rig → Oil | 🎰 Casino → Entertainment | 🏦 Bank → Capital | 💻 Tech Hub → Electronics
- **Terrain restrictions**: Oil Rigs only in ocean, everything else only on land
- **Region bonuses**: each business gets production multipliers based on plot region
  - e.g. Resorts get +100% in tropics, Mines get +100% in mountains, Oil Rigs +80% in deep ocean
- **Multiple businesses per plot**: bigger plots hold more (cottage 1, estate 2, province 3, territory 4)
- Each business has a unique 3D shape (cone, cylinder, box, dome, pyramid, sphere, octahedron)
- **Demolish** any business individually

### 🎰 Casino Gambling
- Casinos don't produce steadily — each tick rolls a slot machine:
  - 🎰 **JACKPOT** (10×) — 2% chance, triggers gold toast notification
  - ✨ **Big Win** (3×) — 10% chance
  - 🎲 **Normal** (1×) — 45% chance
  - 💀 **Bust** (0×) — 43% chance, no production
- Gambling stats panel shows roll history, jackpots, big wins, busts

### 📈 Market System
- **8 materials** trade on a global market with fluctuating prices
- Prices update every **5 minutes** via random walk (±20%, bounded 0.5×–2× base)
- Trend indicators: ▲ up / ▼ down / ◆ stable
- Live countdown timer to next price update
- **Sell individual materials** or **Sell All** at once
- HUD shows total stock value at current market prices

### 🏆 Leaderboard
- Ranks all commanders on the device by **Money**, **Plots**, or **Businesses**
- Top 3 get medals: 🥇 🥈 🥉
- Your row highlighted in gold with "(You)" label
- Accessible from the home page

### 🎨 Home Page
- Modern split-screen design with animated CSS Earth (continents, clouds, atmosphere, orbiting plot markers)
- Welcome card shows your save stats (budget, plots, businesses)
- Footer: "Claim · Build · Trade · Conquer"
- Responsive: 2-column on desktop, single-column on mobile

### 💰 Economy
- Start with **$100,000**
- Plot prices scale with area × latitude base × shape premium
- Business build costs scale with plot area
- Production rates scale with plot area × region bonus
- Income from selling materials at market prices

## How to play

1. Open the live site → **Create Account** (enter name + password)
2. Click **Enter World** → zoom in until plots appear
3. **Buy a plot** — check the terrain chip (🌿 land / 🌊 ocean) and capacity
4. **Build a business** — check the region bonus for each type
5. Materials accumulate automatically — open the **Market** to sell them
6. Wait for market prices to rise, then sell for maximum profit
7. Buy more plots, build more businesses, climb the **Leaderboard**

### Tips
- 🌴 **Tropical land** → build Resorts (+100% bonus)
- ⛰️ **Mountains** → build Mines (+100% bonus)
- 🌊 **Deep ocean** → build Oil Rigs (+80% bonus)
- 🌳 **Temperate land** → build Farms (+50%) or Tech Hubs (+40%)
- 🎰 **Casinos** are volatile but can hit 10× jackpots
- 📈 **Watch the market** — hold materials when prices are low, sell when they spike
- 🏆 **Bigger plots** (estate/province/territory) hold multiple businesses

## Tech

- [Three.js r160](https://threejs.org/) via importmap — no build step
- Single-file `index.html` — no bundler, no npm install
- Earth textures from [three-globe](https://github.com/vasturiano/three-globe) CDN
- **Seeded PRNG** (mulberry32) for deterministic plot generation (shared ownership)
- **Spherical Voronoi** via tangent-plane half-space intersection + Sutherland-Hodgman clipping
- **Water mask + topology** textures sampled for terrain/elevation detection
- **localStorage** for persistence (saves, claims, profiles)
- Auto-deploys to GitHub Pages on every push to `main` via `.github/workflows/deploy.yml`

## Running locally

This is a static site — no server needed. Just open `index.html` in a browser.

```bash
npx serve .
# or
python3 -m http.server 8000
```

## Changelog

### v1.0 (2026-10-02)
- 3D Earth with Voronoi plot tessellation (2,200 non-overlapping cells)
- Login system with password authentication
- Full persistence (save/load to localStorage)
- 8 business types with material production
- Market system with 5-minute price fluctuations
- Region bonuses (7 regions, per-business multipliers)
- Casino gambling (jackpots, big wins, busts)
- Terrain restrictions (oil rigs in ocean, others on land)
- Shared plot ownership (global claims registry)
- Multiple businesses per plot (capacity by tier)
- Leaderboard (money / plots / businesses)
- Redesigned home page with animated CSS Earth
- Starting budget: $100,000

### Earlier
- Initial 3D Earth with buyable plots
- GitHub Pages auto-deploy workflow
- Basic menu screen with hero image
