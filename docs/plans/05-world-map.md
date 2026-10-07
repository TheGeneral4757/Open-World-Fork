# 05 — World Map: Real Countries, Regions & Waters

**Invariant: a territory is a polygon with a stable id from a licensed dataset, never something
generated at runtime.** The server owns the territory table; the client only draws it.

Status: direction decided (D22: real geography, mixed granularity); exact rules open (Q112–Q117).

---

## 1. Granularity model (mixed by density)

```
Country (admin-0) ──split?──▶ Regions (admin-1: states/provinces) ──split?──▶ admin-2 (counties)
   small countries stay whole            most countries stop here            only the biggest/most played
```

A build-time script decides the level per country using rules like:

| Rule (example, tune later) | Effect |
|---|---|
| Country area < X km² **or** population < Y → keep whole | Liechtenstein = 1 territory |
| Medium countries → admin-1 | France → regions, Brazil → states |
| Very large admin-1 units (> Z km²) → admin-2 or split | Siberian oblasts, Australian outback |
| Target: territory sizes within ~1 order of magnitude of each other | Fair-ish land grabs |

Waters, three options (Q114):
- **EEZ sectors**: each coastal country's maritime zone, split into chunks.
- **High-seas grid**: open ocean in hex or lat/lon cells (H3 resolution 2–3).
- **Named seas/basins** (IHO sea areas): fewer, bigger water territories.
Default: EEZ chunks near coasts + an H3 grid for high seas.

## 2. Data sources (check licenses before shipping; record them in `NOTICE`)

| Data | Source | License | Notes |
|---|---|---|---|
| Countries, admin-1 | **Natural Earth** 1:10m / 1:50m | Public domain | Cleanest for a proprietary game |
| admin-2 | geoBoundaries / GADM | geoBoundaries: CC BY 4.0 (most); **GADM: non-commercial only** | Avoid GADM; prefer geoBoundaries + attribution |
| Maritime EEZ, IHO seas | Marine Regions (VLIZ) | CC BY 4.0 (verify version) | Attribution required |
| Hex grid | Uber H3 | Apache-2.0 (library) | Generated, no data license |
| Elevation / land cover (terrain types) | NASA SRTM / ESA WorldCover | Public domain / CC BY 4.0 | Drives terrain bonuses |
| Population / GDP (realism stats) | World Bank, UN | CC BY 4.0 | Optional "realistic" base values |
| OpenStreetMap | — | **ODbL share-alike** | Avoid for the territory database in a proprietary game |

## 3. Pipeline (build time, not runtime)

```
raw shapefiles ──▶ tools/build-world (TS or Python + GDAL/mapshaper)
                     1. pick granularity per country (rules in §1)
                     2. simplify geometry per zoom level (mapshaper -simplify)
                     3. fix topology, compute neighbors (shared borders + sea links)
                     4. compute area, centroid, terrain mix, coastal flag, base price
                     5. assign stable ids (ISO 3166 + admin codes, e.g. FR-IDF, US-CA)
                 ──▶ seed.sql into PostGIS  (server: rules, adjacency, ownership)
                 ──▶ world-lod{0,1,2}.bin  (client: compact triangulated meshes per LOD)
```

Stable ids matter: once players own `US-CA`, the id can't change when the dataset updates.
Keep a `territory_aliases` table for future splits and merges.

## 4. Rendering on the globe

- Pre-triangulate polygons at build time, project them onto the sphere, and merge everything into
  a few pooled meshes with per-vertex color (ownership color). Picking goes through a
  triangle→territory lookup table.
- **LOD:** countries-only mesh when zoomed out, region mesh at mid zoom, admin-2 when close. Load
  higher LODs lazily per visible area.
- Borders: separate line meshes per LOD; coastlines thicker.
- Expected size: ~5k–15k territories at mixed granularity. Fine for WebGL with pooling.
  Chromebooks need the low-LOD path to be the default.

## 5. The politics problem (decide on purpose)

Real borders are disputed (Kashmir, Taiwan, Crimea, Western Sahara, the South China Sea…).
Natural Earth ships **"point of view" variants** for this. Options (Q116):
- (a) Use the de facto boundaries as Natural Earth draws them, with a neutral in-game disclaimer.
- (b) Make disputed areas their own neutral territories.
- (c) Pick one country's point of view (not recommended).
Default: (b) + a disclaimer: "Territories are gameplay regions, not political statements."

Also decide whether real country names are shown (`France`) or only region names (Q117).

## 6. Server model (PostGIS)

```sql
territories (id text pk,            -- 'FR-IDF', 'SEA-H3-82f8...'
             parent_id text null,   -- country for a region
             level smallint,        -- 0 country, 1 region, 2 sub-region, 9 water
             name text, country_iso char(3) null,
             kind text,             -- land | coastal | sea | high_seas
             area_km2 numeric, terrain jsonb, base_price bigint,
             geom geography(MultiPolygon) null)   -- server-side checks/analytics only
territory_neighbors (a text, b text, kind text)    -- land | sea | strait
```
Gameplay never needs heavy geometry at runtime. Adjacency comes from the precomputed neighbors
table; PostGIS is for tooling and analytics.
