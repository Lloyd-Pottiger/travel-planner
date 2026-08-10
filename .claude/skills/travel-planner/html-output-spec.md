# HTML Output Specification

Full layout and content rules for the trip itinerary HTML file generated in Phase 4.

## Design Philosophy — Hand-Drawn Sketch

The output embodies **不完美的秩序** (imperfect order): clear grid and information hierarchy, but every element feels touched by a human hand. Think of a well-kept notebook or classroom whiteboard brought into the browser — not a polished corporate dashboard.

**Core principles to guide every layout decision:**

- **Materials, not glass.** Background reads as paper (dot grid, subtle texture). Cards are sticky notes with hard-edge shadows — paper cast on a desk, not glass floating in space. Pastel palettes (淡黄/淡粉/浅蓝/浅绿) with deep blue-gray ink text.
- **Lines breathe.** Borders aren't machine-straight; asymmetric `border-radius` and slight rotation (±0.5deg) simulate hand-drawn edges. SVG elements (arrows, diagram lines, icons) favor sketch-like contours over perfect geometry. Keep jitter subtle — too much reads as noise, not charm.
- **Typography anchors the tone.** Headings use expressive handwriting fonts (Ma Shan Zheng for Chinese, Caveat for Latin). Body text stays in readable system fonts. Size, weight, and generous whitespace maintain scannability — the user should find key info instantly even in a "doodle" interface.
- **Interaction feels like paper.** Hover lifts a card 2-4px with a shadow deepen — like lifting a sticky note with a fingertip. Click presses it back. Animations use elastic easing at 300–500ms; snappy enough to feel responsive, slow enough to feel physical.
- **Know the boundary.** This style suits creative, educational, and planning contexts. For authority-heavy interfaces (finance, medical), it would undermine trust. The test: if you printed this on A4 and tucked it in a notebook, would it read as a carefully kept draft? If yes, the style is working.

## Route map — Interactive Leaflet map

Use Leaflet on real map tiles as the base layer, with the trip route and stops overlaid. Travelers see actual geography — road networks, lakes, mountain passes, city locations — without any manual drawing.

### Dependencies (load in `<head>`)

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
  integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
  integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
```

### Tile layers

Provide two base layers with `L.control.layers` (radio-style):

1. **高德地图** (default for China trips):
   - URL: `https://webrd0{s}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}`
   - `subdomains: '1234'`, `attribution: '© 高德'`
   - Much better Chinese road/POI labels than OSM. No API key needed.

2. **OpenStreetMap** (fallback):
   - URL: `https://tile.openstreetmap.org/{z}/{x}/{y}.png`
   - `attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'`

Default to 高德 for China domestic trips. Keep OSM as a layer-switcher option.

### Map container

Wrap the map in the existing `.diagram-wrap` class to preserve the hand-drawn frame:

```html
<div class="diagram-wrap" id="map-wrap">
  <div id="map" style="height:520px;width:100%;border-radius:12px 2px 12px 2px/2px 12px 2px 12px;"></div>
</div>
```

Map height: 480–560px depending on the route's north-south span. Taller for north-south routes (e.g., 川西), shorter for east-west routes (e.g., 甘青).

### Node data

Collect `lat`/`lng` for every stop during Phase 2 route design. This includes:
- **Overnight stays**: every place the traveler sleeps
- **Major pass-through points**: mountain passes, scenic lookouts, charging stops, road junctions that define the route shape
- **Transport hubs**: airports and train stations where the trip transitions mode

Every node gets: `{name, lat, lng, type, emoji, section}` where:

| type | meaning | marker radius | fill |
|------|---------|:------------:|------|
| `base` | multi-night stay (≥2 nights) | 10–11px | section pastel color |
| `overnight` | single-night stop | 8px | section pastel color |
| `pass-through` | scenic stop / pass / charging stop | 5–6px | `#fff` with colored border |
| `hub` | start/end city (flight/train connection) | 10px | `#fff` with dashed border |

All markers use `L.circleMarker` with `stroke: '#2c3e50'`, `weight: 2`. Color-code by trip section using hardcoded hex (CSS custom properties don't resolve in Leaflet):
- 🏔️ 山区/草原: `#f0faf4` fill, `#4a7c5e` border
- 🏜️ 戈壁/沙漠: `#fff7ed` fill, `#c7854a` border
- 💧 高原/盐湖: `#f7f2ff` fill, `#7c5aa8` border

### Route polylines

Draw each segment with intermediate waypoints so the line **roughly follows actual roads** — not a straight "as the crow flies" line. For every segment, collect 2–5 intermediate `[lat, lng]` points where the road makes significant turns.

Line styles (all use `L.polyline`):

| transport | color | weight | dashArray | opacity |
|-----------|-------|:------:|-----------|:-------:|
| 🚗 自驾 (drive) | `#2c3e50` | 3 | — | 0.85 |
| 🚄 高铁 (train) | `#6366f1` | 2 | `8 6` | 0.6 |
| ✈️ 飞行 (flight) | `#22c55e` | 2 | `4 8` | 0.5 |
| ↳ 支线往返 (detour) | `#e8734a` | 2.5 | `6 4` | 0.7 |

**Detour identification**: when a day trip goes out from a base and returns (e.g., 大柴旦→水上雅丹→大柴旦, 德令哈→托素湖→德令哈), draw it as a spur branching off the main loop with the detour line style. Add a `↳` prefix in tooltips and labels.

**Direction arrows**: place small triangle markers (`L.marker` with a custom `divIcon` of `▶` rotated to match heading) at 2–3 evenly spaced points along each drive segment.

### Marker interactivity

- **Tooltip** (`L.tooltip` on hover): stop name + brief info (e.g., "祁连 · D2-3 · 2晚 · 2800m")
- **Click**: `map.setView([lat, lng], 13)` to zoom in, then `document.querySelector('#day-N').scrollIntoView({behavior:'smooth'})` to scroll to the corresponding day card
- **Permanent labels**: for major stops (base + hub types), show the emoji as a permanent tooltip (`permanent: true, direction: 'center'`) so the stop is identifiable without hovering

### Legend

A custom `L.control` at `'bottomleft'` showing:
```
● Solid line  = 自驾
- - Dashed    = 高铁
· · Dotted    = 飞行
- · Dash-dot  = 支线往返
○ Circle size = 连住多晚 > 住1晚 > 途经
```

### Initial view

```js
const bounds = L.latLngBounds(allCoords);
map.fitBounds(bounds.pad(0.15));  // 15% padding on all sides
```

### Map verification checklist

- [ ] All markers at correct lat/lng: spot-check against known geography (e.g., 西宁 east of 青海湖, 敦煌 far west of 张掖, 格尔木 south of 大柴旦)
- [ ] Polyline waypoints follow visible roads on the map, not straight lines across mountains/lakes
- [ ] Zoom/pan works on mobile (touch events enabled by default in Leaflet)
- [ ] Layer switcher toggles between 高德 and OSM; both render without errors
- [ ] No marker clusters overlap at default zoom; if two stops are <5km apart, offset their tooltip directions
- [ ] Fit-bounds shows all nodes at initial load with comfortable padding

## Day cards

- CSS Grid with `minmax(340px, 1fr)` for 2-3 columns on desktop, single column on mobile
- Each card contains: day number + date, route badge, quick-info tags (weather, drive time, transport mode), sections for activities / food / accommodation, and a contextual note callout
- Section labels in the traveler's language (e.g., 玩什么 / 吃什么 / 住哪里 for Chinese trips)
- Subtle random rotation on cards (`±0.5deg max`) for a handmade feel; functional UI elements (tags, badges) stay unrotated
- Pastel color variants rotated across cards
- **Note-callout content rules**:
  - ✅ Practical warnings for THIS day only: weather hazards, road conditions, altitude adjustment
  - ✅ Insider tips that enhance THIS day's experience (e.g., "丹霞下午4点后进，正午光线太硬")
  - ✅ Historical/cultural context for THIS day's attractions
  - ❌ "明天去XXX…" itinerary previews — next-day info already exists in the next day card and the segment table
  - ❌ Repeated transport details that are in the segment table
  - ❌ Hotel check-in/checkout logistics (belongs in accommodation summary, not repeated across day cards)
  - ❌ General regional advice that applies to ≥3 days (belongs in Reminders)

## Additional sections

**Route flow overview** — inline pills showing the route at a glance. Keep it scannable: ≤10 pills covering major stops only. Detailed stop sequence lives on the map and in the segment table; pills are a 5-second index, not a full itinerary.

**Stats bar** — total days, driving distance, train/flight segments, regions covered, buffer days.

**Segment summary table** — placed between the route map and day cards. **This is the single source of truth for all segment-level data.** Every driving/train/flight leg gets one row.

Table structure:

| 日期 | 路段 | 距离(km) | 驾车 | 道路 | 备注 |
|------|------|:------:|:----:|------|------|
| D1 | 广州✈️兰州🚄西宁 | — | — | — | 飞行3h+高铁1h |
| D2 | 西宁→门源→祁连 | 280 | 5h | G227 | 达坂山▲3792m |
| D8 | 敦煌→冷湖→大柴旦 | 500 | 6h | G215 | 翻当金山▲3648m |
| ↳ D9 | 大柴旦→水上雅丹(往返) | 440 | 5h | G315 | 支线往返 |
| **合计** | | **~3000** | **~50h** | | |

Styling rules:
- Driving rows: normal weight, white background
- Flight/train rows: light blue background (`var(--b)`), slightly muted text
- Detour spur rows: indented with `↳` prefix, smaller font (`font-size:.78rem`), muted color. These are day-trip round-trips from a base (e.g., 水上雅丹 往返 大柴旦)
- Total row: bold, yellow background (`var(--y)`), border-top
- Road column: small pill-style label (`background:var(--paper); border-radius:4px; padding:1px 6px`)
- Distance and time columns: right-aligned, tabular-nums
- Group rows by trip section (matching section dividers in day cards) with thin group-header rows

CSS class: `.seg-table`. Style definition in `style-template.css`.

**Packing checklist** — generate dynamically based on trip type, weather, activities, and transport mode. Categories are determined by the trip, not hardcoded (e.g., road trip → car supplies; beach → swim gear; mountain → layers and rain gear).

**Important reminders** — **cross-cutting only**: each reminder must apply to ≥3 days of the trip. Day-specific warnings live in day card note-callouts. Weather strategy lives in ONE consolidated reminder (seasonal patterns, not a day-by-day list). Charging/gas strategy is ONE reminder. Bathroom reality is ONE reminder. Altitude acclimatization STRATEGY is ONE reminder (not repeated per day).

## Completion checklist

Verify before finalizing the HTML output:

### Structure & layout
- No stretch of 3+ consecutive one-night stays (cluster 2-3 nights per base per universal principle 4)
- All day-of-week labels verified against the actual calendar year for the trip dates
- CSS grid uses `minmax(340px, 1fr)`, no element has a fixed width exceeding 380px, and font sizes use `clamp()` or relative units — ensures readability at 384px mobile viewport
- Leaflet map loads without JS errors; both tile layers (高德 + OSM) render correctly
- Map `fitBounds` shows all route nodes with comfortable padding; no node clipped at edges

### Anti-duplication (single source of truth)

- **No "明天" previews in note-callouts**: no note-callout contains "明天去…" or equivalent next-day itinerary previews. That information already lives in the next day card and the segment table.
- **Reminders are cross-cutting only**: every item in the reminders section applies to ≥3 days. Day-specific warnings stay in day card note-callouts. Weather strategy is ONE consolidated reminder (seasonal patterns), not a day-by-day weather list.
- **Weather single source**: daily weather forecasts appear ONLY in day card tags. Reminders give climate patterns, not per-day forecasts.
- **Accommodation table is a lookup index**: it lists night + location + name + price + ONE critical constraint. Do not repeat altitude warnings, nearby restaurants, or check-in tips already in day cards.
- **Route-flow pills ≤10 items**: major stops only as a quick index. The detailed stop sequence lives on the map and in the segment table.
- **Segment data lives in the segment table**: day card route-badge shows only the start→end city pair. Distance and drive time come from the segment table — day cards should not repeat them in prose.
- **Food repetition is OK across day cards** (travelers eat every day), but food must NOT appear in reminders, accommodation table, or segment table.
