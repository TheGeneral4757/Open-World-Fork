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
- **Backup buttons**: Export all accounts to a JSON file / Import them back on any device
- **Optional cloud saving (Supabase)**: paste your project URL + anon key at the top of the game script (see `supabase-setup.sql`) and every account + save is mirrored to the cloud automatically — accounts survive cleared browsers and are shared across all devices. Old local accounts migrate to the cloud on first visit. The login screen shows whether cloud saving is ON or OFF.

### 🌐 Playing online together (Supabase)
The game can run as **one shared world across devices** using a free [Supabase](https://supabase.com) project as its backend — no server to run.

**One-time setup (~5 minutes):**
1. Create a free project at [supabase.com](https://supabase.com)
2. Dashboard → **SQL Editor** → paste the contents of [`supabase-setup.sql`](supabase-setup.sql) → **Run**
3. Dashboard → **Settings → API**: copy the **Project URL** and the **anon public key**
4. Paste both into `SUPABASE_URL` / `SUPABASE_ANON_KEY` at the top of the game script in `index.html` and commit — GitHub Pages redeploys automatically

The login screen shows **🌐 Online multiplayer: ON** when configured. What you get:
- **One shared world** — every account, save, and plot claim lives in the cloud; players on any device see each other's plots
- **Live sync** — while playing, the world refreshes every 30 seconds: new claims appear, enemy army sizes stay current, and plots conquered while you were online are stripped from your game with a notification (your next autosave agrees with the cloud instead of reverting it)
- **Cross-device progression** — log in on any device and continue where you left off; old local accounts migrate to the cloud on first visit
- **The leaderboard and wars span devices** — attack players who are playing on a different device than you

### 👑 World-owner admin panel (admin account only)
The shared world has an **admin panel for the world owner** — the account `kingkanye26`. Nobody else can see it, open it, or call its actions.

One door, right after logging in:
- Log in as the owner → the **home (menu) screen** shows a **👑 Admin** button (hidden for everyone else) → clicking it opens a **verification lock**: the owner's password is required **every single time** the panel is opened (closing and reopening always re-asks). Wrong passwords are rejected with a shake — the control center never renders until the password matches. No admin UI exists on the login screen or inside the world — entering the world always starts clean.
- **Standalone**: [`admin.html`](admin.html) — asks for the owner account's password before unlocking

Owner powers:
- **Set any account's money**, reset accounts (back to $100,000, no plots/businesses), zero inventories (materials + troops/ships/planes)
- **Delete accounts** completely — a deletion tombstone in the cloud makes sure no device can resurrect them; freed plots become claimable again
- **Reset ALL accounts** / **clear every plot claim** in one click

How it stays reliable with online multiplayer:
- Every owner edit carries an **owner stamp** — game devices treat it as authoritative, so the 10-second autosave or 30-second world sync can never overwrite an admin decision (this was the bug that made the first panel version silently revert)
- Edits are also mirrored into the owner's own browser, so changes apply the moment they re-enter the world
- A player whose account is edited while online gets a notification and keeps a consistent view until they re-enter

Notes: passwords are lightly hashed client-side and the anon key is public by design — this is a game for friends, not a bank. Supabase's free tier comfortably handles a group of friends playing casually.

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

### ⚔️ Military & Conquest
- **Barracks** (land) train ⚔️ troops, **Naval Yards** (coasts & oceans) train ⚓ ships, and **Airfields** (land) train ✈️ planes — military resources that can't be sold on the market
- **Affordable military**: Barracks $120k, Naval Yard $180k, Airfield $280k — priced with the mid-game economy tier so building an army is always within reach
- **Army housing cap**: each Barracks houses **500 troops** — your army cap is 500 × Barracks (captured ones count). Training pauses when the barracks are full; the HUD shows `troops / cap`
- Your forces defend your plots automatically; enemies see your strength before attacking
- **Conquer other commanders' plots**: click a claimed plot, deploy forces, and attack
  - Land plots are fought with ⚔️ troops (⚔️ Conquest); sea plots are fought with ⚓ ships (⚓ Naval Invasion)
  - ✈️ planes from your Airfields give air support to every battle: +0.05% strength per plane, up to +50%
  - Defenders get terrain bonuses: mountains ×1.5, highlands ×1.35, ocean ×1.3, other land ×1.2
  - Battles roll strength with ±15% luck — the modal shows your estimated win chance before you deploy
  - **Win**: the plot becomes yours — captured businesses keep producing for you — and both sides take casualties
  - **Lose**: the entire deployed force is lost; the defender takes light losses
- The 📊 **Army** chip in the HUD shows your troop strength and housing cap (e.g. `348 / 500`)
- **Coastal land plots show a bluer outline** — that's where Naval Yards can be built (ocean plots work too)
- **Military buildings are clearly distinct** from businesses: a separate red ⚔️ Military section in the build menu, MILITARY badges on army buildings, and a red base ring on the globe

### 🎰 Casino Gambling
- Casinos don't produce steadily — each tick rolls a slot machine:
  - 🎰 **JACKPOT** (10×) — 2% chance, triggers gold toast notification
  - ✨ **Big Win** (3×) — 10% chance
  - 🎲 **Normal** (1×) — 45% chance
  - 💀 **Bust** (0×) — 43% chance, no production
- Gambling stats panel shows roll history, jackpots, big wins, busts
- **Interactive slots minigame** — with a casino you can gamble your own money:
  - Open it from the HUD **🎰 Casino** button or the **Play Slots Here** button on any casino plot
  - Place a bet (from $100 up to your whole budget), spin the reels, watch them stop one by one
  - Same paytable as passive rolls: 💎💎💎 pays 10× your bet, a triple 7️⃣/🔔/⭐ pays 3×, a 🍒 pair pushes, mixed reels lose
  - **Every casino beyond your first reduces the bust chance** (up to −18%) — casino empires gamble with better odds
  - Session stats (spins, wagered, won, net, best) and a recent-spins history strip

### 📈 Market System
- **8 materials** trade on a global market with fluctuating prices
- Prices update every **15 minutes** via a gentle random walk (±8%, bounded 0.5×–2× base) — position your stock, don't watch the clock
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
- 📱 **On a Chromebook or older laptop?** Set Graphics to Balanced or Low in Settings (Auto usually picks the right one for you)

### ⚡ Performance (Chromebook-friendly)
- **Device-aware graphics quality** — Auto / High / Balanced / Low. Auto detects your hardware on first run (CPU cores, memory, GPU, ChromeOS); override it in **Settings** or with the in-game quality button. The choice is remembered per browser.
- **All 2,200 plots render in 4 draw calls** — cells and outlines are merged into pooled meshes with per-vertex colors, instead of one mesh per plot (~4,400 draw calls before) — the single biggest win for weak GPUs
- **Adaptive resolution** — in Auto mode, the render scale steps down (and back up) automatically to hold a smooth frame rate
- **Quality-scaled assets** — Balanced/Low load downscaled Earth textures and terrain masks, a lower-poly globe, fewer stars, and can disable antialiasing — faster load and far less GPU memory
- 3D rendering pauses while on the login/menu screens

## Tech

- [Three.js r160](https://threejs.org/) via importmap — no build step
- Single-file `index.html` — no bundler, no npm install
- Earth textures from [three-globe](https://github.com/vasturiano/three-globe) CDN
- **Seeded PRNG** (mulberry32) for deterministic plot generation (shared ownership)
- **Spherical Voronoi** via tangent-plane half-space intersection + Sutherland-Hodgman clipping
- **Merged plot render pools** — per-vertex-colored meshes/lines for all cells in 4 draw calls, with triangle→plot lookup for raycast picking
- **Quality levels + device detection** with adaptive resolution scaling
- **Water mask + topology** textures sampled for terrain/elevation detection
- **localStorage** for persistence (saves, claims, profiles, quality setting)
- Auto-deploys to GitHub Pages on every push to `main` via `.github/workflows/deploy.yml`

## Running locally

This is a static site — no server needed. Just open `index.html` in a browser.

```bash
npx serve .
# or
python3 -m http.server 8000
```

## Changelog

### v1.8 (2026-10-05) — Affordable army, troop housing cap, deploy language
- **Military buildings are now mid-game affordable**: Barracks $500k → **$120k**, Naval Yard $650k → **$180k**, Airfield $900k → **$280k**
- **Troop cap**: each Barracks houses 500 troops — cap = 500 × Barracks owned (captured barracks count). Training pauses when full; excess troops are never removed
- HUD Army chip now shows `troops / cap` (e.g. `0 / 500`), and Barracks cards show live housing numbers
- War panel wording changed from "Commit" to **"Deploy"** — deploy troops/ships to other commanders' plots and take them
- War hints show current build costs so new commanders know the path to their first army

### v1.6.5 (2026-10-05) — Owned plots hug the surface
- **Fixed**: owned plots used to hover visibly above the planet (they were radially inflated ~5% as a "proud of the surface" effect). They now sit flush on the earth exactly like every other cell — still shining gold, just no longer floating islands

### v1.6.4 (2026-10-05) — Even world lighting
- **Fixed**: half the globe used to sit in darkness — the sun was fixed in space while the world spun under it. The sun is now locked to the camera and the ambient light is much stronger, so the entire world is evenly lit from every angle

### v1.6.3 (2026-10-05) — Owner verification lock
- **Security**: opening the admin panel now requires the owner's password **every time** — the 👑 Admin button opens a gold "Verification required" lock screen inside the panel, and only the correct password swaps it for the control center (Enter key works, wrong attempts shake and clear)

### v1.6.1 (2026-10-05) — Working admin panel
- **Fixed**: admin-panel edits were silently reverted within seconds — the panel only wrote to the cloud, while each device's 10s autosave kept pushing stale local state back over it. Owner edits are now **owner-stamped** (`adminStamp`) and win over any autosave; deletes leave **tombstones** so deleted accounts stay deleted; claim resets carry a **claims stamp** so cleared claims stay cleared
- Admin edits also write through to the owner's own browser — enter the world and the change is already live
- Players whose account is adjusted while online see a notification instead of silently diverging
- Every admin action is double-guarded to the `kingkanye26` account (button visibility + per-action check)

### v1.6 (2026-10-04) — Online multiplayer
- Live world sync: while playing online, the world refreshes every 30 seconds — new claims from other devices appear on your globe, and enemy army sizes stay current for war
- Conquests propagate live: if one of your plots is conquered while you're online, it's stripped from your game with a notification and your next autosave agrees with the cloud (previously your device could silently revert another player's conquest)
- Conquest rewrites of a victim's save now carry a fresh timestamp so the ownership change wins cloud merges
- Login screen now reads "🌐 Online multiplayer: ON/OFF" and the code carries step-by-step setup instructions
- See the new "Playing online together" section for the 5-minute Supabase setup

### v1.5 (2026-10-04) — Navy & air force
- **⚓ Naval Yard** ($650k): builds warships on ocean plots and coastal land plots; ships fight and defend sea plots
- **✈️ Airfield** ($900k): operates a runway on land; planes provide air support to every battle (+0.05% strength each, up to +50% for both attacker and defender)
- Sea plots are now invaded with ships (⚓ Naval Invasion) instead of troops — build a navy before attacking islands and ocean rigs
- Coastal land plots show a bluer outline so Naval Yard sites are easy to spot
- New meshes: an octagonal dock platform for the Naval Yard, a runway strip for the Airfield
- War panel shows your full forces (⚔️ troops · ⚓ ships · ✈️ planes) and adapts per plot type

### v1.4.1 (2026-10-04) — Military/business distinction
- Build menu split into 🏗️ Businesses and ⚔️ Military sections, with red military styling
- Owned military buildings show a MILITARY badge and a red-tinted card
- Barracks (and future military buildings) render with a red base ring on the globe

### v1.4 (2026-10-04) — Slow-paced rebalance
- Production rates cut to ~40% across all businesses (a Farm now pays for itself in ~8 minutes instead of ~90 seconds)
- Business build costs doubled (Farm $16k → Tech Hub $1.2M)
- Plot prices doubled across all latitude bands — a $100k start now begins on the cheap high-latitude frontier, with the valuable equator as a long-term goal
- Market prices update every 15 minutes (was 5) with gentler ±8% swings — sell on strategy, not on a stopwatch
- Troops train slower (Barracks rate 0.4/s), making war preparation a deliberate investment
- Existing saves are fully compatible; balances and businesses carry over unchanged

### v1.3.1 (2026-10-03) — Bugfix
- Fixed mirrored longitude in plot labels and terrain detection: plots now show their true coordinates and terrain (a plot over Canada was labeled "Asian Steppes 93°E" and classified Deep Ocean; plot positions were always correct — only the labels and map sampling were mirrored)
- All plot names and terrain re-derive on reload; existing ownership, businesses, and prices are unaffected

### v1.3 (2026-10-03) — Military & conquest
- **Barracks** business (land, $250k): trains ⚔️ troops, a military resource excluded from the market
- **Army HUD chip** shows your troop strength at a glance
- **Conquest**: attack any plot claimed by another commander — commit troops, see the estimated win chance, and fight
- Terrain defense bonuses (mountains ×1.5, highlands ×1.35, ocean ×1.3, land ×1.2) and ±15% battle luck
- Victory transfers the plot and its businesses to the conqueror; the defender's save is updated live
- Losses on both sides scale with how close the battle was; a lost attack kills the entire committed force

### v1.2 (2026-10-03) — Interactive casino
- **Lucky Diamond Slots**: interactive slot-machine minigame for casino owners — bet your own money and spin
- Real bets with instant payouts: jackpot 10×, big win 3×, cherry-pair push, mixed reels lose (same odds table as passive casinos)
- Every casino beyond the first reduces the bust chance (up to −18%), rewarding casino empires
- Bet chips (+1K / +5K / +25K / +100K / MAX) with a $100 minimum
- Animated reels with staggered stops, pop animations, result line and jackpot/big-win toasts
- Session stats (spins, wagered, won, net, best) and recent-spin history
- HUD **🎰 Casino** shortcut appears whenever you own a casino; **Play Slots Here** button on every casino plot

### v1.1 (2026-10-03) — Performance & device optimization
- Graphics quality system: Auto / High / Balanced / Low with hardware auto-detection, persisted choice, Settings UI and in-game quality button
- Merged all 2,200 plot meshes + outlines into 4 pooled draw calls (was ~4,400 draw calls when zoomed in)
- Adaptive resolution scaling in Auto mode (dynamic render-scale to hold smooth FPS)
- Downscaled Earth textures and terrain masks on Balanced/Low (big cut in GPU memory and load time)
- Quality-scaled star count, Earth/atmosphere geometry, and antialiasing
- Rendering pauses while on login/menu screens; zoom-meter DOM updates throttled
- Fixed: second business on a plot could get a NaN position (missing plot size field)

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
