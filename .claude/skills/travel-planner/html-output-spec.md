# HTML Output Specification

Full layout and content rules for the trip itinerary HTML generated in Phase 4.

## Design Philosophy — 不完美的秩序 (Imperfect Order)

Clear information hierarchy, but every element feels touched by a human hand — a well-kept notebook, not a corporate dashboard.

- **Materials, not glass.** Background reads as paper (dot grid, subtle texture). Cards are sticky notes with hard-edge shadows. Pastel palettes (淡黄/淡粉/浅蓝/浅绿) with deep blue-gray ink.
- **Lines breathe.** Asymmetric `border-radius` and slight rotation (±0.5deg) simulate hand-drawn edges. Keep jitter subtle — too much reads as noise.
- **Typography anchors the tone.** Handwriting fonts for headings (Ma Shan Zheng for Chinese, Caveat for Latin), readable system fonts for body. Size, weight, and whitespace maintain scannability.
- **Interaction feels like paper.** Hover lifts a card 2–4px with shadow deepen. Animations use elastic easing at 300–500ms.
- **Know the boundary.** If you printed this on A4 and tucked it in a notebook, would it read as a carefully kept draft? If yes, the style is working.

## Route Map — Interactive Leaflet

Leaflet on real map tiles so travelers see actual geography — roads, lakes, passes, cities — without manual drawing.

### Dependencies (load in `<head>`)

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
  integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
  integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
```

### Tile Layers

Two base layers via `L.control.layers` (radio-style):

1. **高德地图** (default for China trips):
   - URL: `https://webrd0{s}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}`
   - `subdomains: '1234'`, `attribution: '© 高德'`
   - Better Chinese road/POI labels than OSM. No API key needed.

2. **OpenStreetMap** (fallback):
   - URL: `https://tile.openstreetmap.org/{z}/{x}/{y}.png`
   - `attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'`

### Map Container

```html
<div class="diagram-wrap" id="map-wrap">
  <div id="map" style="height:520px;width:100%;border-radius:12px 2px 12px 2px/2px 12px 2px 12px;"></div>
</div>
```

Height: 480–560px depending on north-south span (taller for 川西, shorter for 甘青).

### Node Data

Collected during Phase 2. Every node: `{name, lat, lng, type, emoji, section}`.

| type | meaning | marker radius | fill |
|------|---------|:------------:|------|
| `base` | multi-night stay (≥2 nights) | 10–11px | section pastel color |
| `overnight` | single-night stop | 8px | section pastel color |
| `pass-through` | scenic stop / pass / charging stop | 5–6px | `#fff` with colored border |
| `hub` | start/end city (flight/train connection) | 10px | `#fff` with dashed border |

All markers: `L.circleMarker`, `stroke: '#2c3e50'`, `weight: 2`. Color-code by section using hardcoded hex:
- 🏔️ 山区/草原: `#f0faf4` fill, `#4a7c5e` border
- 🏜️ 戈壁/沙漠: `#fff7ed` fill, `#c7854a` border
- 💧 高原/盐湖: `#f7f2ff` fill, `#7c5aa8` border

### Route Polylines

Each segment includes 2–5 intermediate waypoints so lines roughly follow actual roads (not straight lines).

| transport | color | weight | dashArray | opacity |
|-----------|-------|:------:|-----------|:-------:|
| 🚗 自驾 (drive) | `#2c3e50` | 3 | — | 0.85 |
| 🚄 高铁 (train) | `#6366f1` | 2 | `8 6` | 0.6 |
| ✈️ 飞行 (flight) | `#22c55e` | 2 | `4 8` | 0.5 |
| ↳ 支线往返 (detour) | `#e8734a` | 2.5 | `6 4` | 0.7 |

**Detour identification:** Day trips returning to base (e.g., 大柴旦→水上雅丹→大柴旦) use the detour line style with `↳` prefix in tooltips.

**Direction arrows:** Small `▶` markers (`divIcon`, rotated to heading) at 2–3 evenly spaced points along each drive segment.

### Marker Interactivity

- **Tooltip** (`L.tooltip` on hover): stop name + brief info (e.g., "祁连 · D2-3 · 2晚 · 2800m")
- **Click**: zoom in then `scrollIntoView({behavior:'smooth'})` to the corresponding day card
- **Permanent labels**: for major stops (base + hub), show emoji as `permanent: true, direction: 'center'` tooltip

### Legend

Custom `L.control` at `'bottomleft'`:
```
● Solid line  = 自驾
- - Dashed    = 高铁
· · Dotted    = 飞行
- · Dash-dot  = 支线往返
○ Circle size = 连住多晚 > 住1晚 > 途经
```

### Initial View

```js
const bounds = L.latLngBounds(allCoords);
map.fitBounds(bounds.pad(0.15));  // 15% padding
```

### Map Verification Checklist

- [ ] All markers at correct lat/lng: spot-check against known geography (西宁 east of 青海湖, 敦煌 far west of 张掖, 格尔木 south of 大柴旦)
- [ ] Polyline waypoints follow visible roads, not straight lines across mountains/lakes
- [ ] Zoom/pan works on mobile (touch events enabled by default)
- [ ] Layer switcher toggles between 高德 and OSM; both render without errors
- [ ] No marker clusters overlap at default zoom; offset tooltip directions if stops <5km apart
- [ ] `fitBounds` shows all nodes at initial load with comfortable padding

## Day Cards

- CSS Grid `minmax(340px, 1fr)`: 2–3 columns desktop, single column mobile
- Each card: day number + date, route badge (start→end city only), quick-info tags (weather, transport mode), sections for activities / food / accommodation, note callout
- Section labels in traveler's language (e.g., 玩什么 / 吃什么 / 住哪里 for Chinese trips)
- Subtle random rotation (±0.5deg max) on cards; functional elements (tags, badges) stay unrotated
- Pastel color variants rotated across cards
- **Note-callout rules:**
  - ✅ Practical warnings for THIS day: weather hazards, road conditions, altitude
  - ✅ Insider tips that enhance THIS day's experience (e.g., "丹霞下午4点后进，正午光线太硬")
  - ✅ Historical/cultural context for THIS day's attractions
  - ❌ "明天去XXX…" itinerary previews — next-day info is in the next day card + segment table
  - ❌ Repeated transport details (in the segment table)
  - ❌ Hotel check-in/checkout logistics (in accommodation summary)
  - ❌ General regional advice applying to ≥3 days (in Reminders)

## Additional Sections

**Route flow overview** — inline pills, ≤10 items, major stops only. A 5-second scan index; detailed sequence lives on the map and segment table.

**Stats bar** — total days, driving distance, train/flight segments, regions covered, buffer days.

**Segment summary table** — placed between map and day cards. **Single source of truth** for segment data. One row per leg.

| 日期 | 路段 | 距离(km) | 驾车 | 道路 | 备注 |
|------|------|:------:|:----:|------|------|
| D1 | 广州✈️兰州🚄西宁 | — | — | — | 飞行3h+高铁1h |
| D2 | 西宁→门源→祁连 | 280 | 5h | G227 | 达坂山▲3792m |
| D8 | 敦煌→冷湖→大柴旦 | 500 | 6h | G215 | 翻当金山▲3648m |
| ↳ D9 | 大柴旦→水上雅丹(往返) | 440 | 5h | G315 | 支线往返 |
| **合计** | | **~3000** | **~50h** | | |

Styling:
- Driving rows: normal weight, white background
- Flight/train rows: light blue background (`var(--b)`), muted text
- Detour spur rows: indented with `↳`, smaller font, muted color
- Total row: bold, yellow background (`var(--y)`), border-top
- Road column: small pill-style label
- Distance/time columns: right-aligned, tabular-nums
- Group rows by trip section with thin group-header rows

**Packing checklist** — dynamically generated from trip type, weather, activities, transport mode. Categories are trip-determined (e.g., road trip → car supplies; beach → swim gear; mountain → layers and rain gear).

**Important reminders** — **cross-cutting only**: each item must apply to ≥3 days. Day-specific warnings live in day card note-callouts. Weather strategy is ONE consolidated reminder (seasonal patterns, not day-by-day). Charging/gas strategy is ONE reminder. Bathroom reality is ONE reminder. Altitude strategy is ONE reminder.

## Completion Checklist

### Structure & Layout
- [ ] No stretch of 3+ consecutive one-night stays (cluster 2–3 nights per base)
- [ ] All day-of-week labels verified against the actual calendar year
- [ ] CSS grid uses `minmax(340px, 1fr)`; no fixed width >380px; font sizes use `clamp()` or relative units (readable at 384px viewport)
- [ ] Leaflet map loads without JS errors; both tile layers render correctly
- [ ] `fitBounds` shows all nodes with comfortable padding; no node clipped

### Anti-Duplication (Single Source of Truth)
- [ ] **No "明天" previews** in any note-callout
- [ ] **Reminders are cross-cutting only** (≥3 days). Weather strategy is ONE reminder. Day-specific warnings in day card note-callouts.
- [ ] **Weather single source**: daily forecasts in day card tags only; reminders give climate patterns, not per-day forecasts.
- [ ] **Accommodation table is a lookup index**: night + location + name + price + ONE critical constraint. No duplicated altitude warnings, restaurant tips, or check-in details.
- [ ] **Route-flow pills ≤10**: major stops as quick index; detailed sequence on map + segment table.
- [ ] **Segment data in segment table**: day card route-badge shows start→end city pair only; distance and drive time not repeated in prose.
- [ ] **Food repetition OK across day cards** (travelers eat every day), but food must NOT appear in reminders, accommodation table, or segment table.
