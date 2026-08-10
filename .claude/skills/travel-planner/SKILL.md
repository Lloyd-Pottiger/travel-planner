---
name: travel-planner
description: Plan a multi-day road trip or domestic travel itinerary within China (中国).
---

# Travel Planner Skill

## Methodology

### Phase 1: Information Gathering

Ask questions before designing — unclear requirements produce bad routes. Batch 3–4 questions per round via `AskUserQuestion`, prioritizing the highest-variance dimensions first. Keep going until remaining unknowns wouldn't materially fork the route. Skip what the user has already covered.

Key dimensions (adapt order to what's known):

- **Travel companions** — pace, accommodation needs, number of drivers
- **Dates & duration** — start/end dates, hard constraints mid-trip
- **Driving tolerance** — hours/day; baseline: mountain roads 3–5h, highways 5–8h
- **Trip themes** — nature, culture, food, relaxation, or mix — drives routing and activity choices
- **Route shape** — starting point, direction, endpoint, must-hit places
- **Transport mode** — self-drive, rental, train, or hybrid
- **Accommodation & budget** — style, nightly range, total budget
- **Previous experience** — which destinations on the route already visited?
- **Weather attitude** — accept rain, avoid, or decide day-by-day
- **Regional trade-offs** — when geography and time conflict, which areas get priority?
- **Flight price sensitivity** — willing to adjust endpoint or reverse route to save on flights?

### Phase 2: Route Design

**Universal principles:**
1. Minimize backtracking — route flows in one general direction
2. Match daily driving to user's tolerance, adjusted for terrain and road type
3. Use rail to skip highway segments with no scenery or sightseeing value
4. Prefer one-way loops over out-and-back when geometry allows
5. Factor in seasonal weather — rainy season affects mountain road safety; peak summer means crowded attractions and tight bookings
6. Match activities to weather requirements:
   - Sunrises, mountain panoramas, stargazing → need clear or mostly-clear skies; don't schedule on forecasted rainy days
   - Waterfalls, river rafting → best after/during rain when water volume peaks
   - Snow/ice scenery → temperature-dependent; check seasonal viability
   - For every weather-sensitive outdoor day, have an indoor/cultural backup and consider allocating a float day nearby
7. Consider altitude acclimatization when route involves rapid elevation gain
8. Place buffer days after long drives or before critical connections

**Transport mode selection:**
- Boring highway / no scenery → train or flight
- Scenic roads (mountain passes, coastal routes) → self-drive
- Within a city or compact region → self-drive for flexibility
- Cross-region one-way rental with high drop-off fees → split into local rentals connected by train
- For each segment, compare self-drive (rental + fuel + tolls + drop-off) vs. train/flight (ticket + local transport); present the trade-off when the difference is significant

**Accommodation strategy:**
- Match style to trip type (guesthouses for cultural trips, hotels for city stops, resorts for beach/relaxation)
- Cluster stays — 2–3 nights per base with day trips beats 1-night-stands
- Pre-book for peak season; off-season can stay flexible
- For each stop, present 2–3 concrete options within budget: name, rough price, location advantage, one-line reason it fits. Source from Trip.com/携程 (China domestic), Booking.com/Agoda (international), 大众点评 (local guesthouse reviews)

**Real-world validation:** Use `/xhs-search` to find real traveler experiences — route pitfalls, accommodation reviews, restaurant recommendations, seasonal road conditions, hidden gems.

**Geo-coordinate collection:** For every stop — overnight stays AND major pass-through points (mountain passes, scenic lookouts, charging stops, road junctions) — collect approximate `[lat, lng]` coordinates. Also collect 2–5 intermediate waypoints per road segment so Leaflet polylines follow actual roads, not straight lines.

**Phase 2 complete when:** every segment has a transport mode, daily driving stays within tolerance (terrain-adjusted), all 8 universal principles are satisfied, accommodation options have been presented, and lat/lng coordinates + waypoints collected for all stops.

### Phase 3: Transport & Weather Optimization

**Flight/train optimization:**
- Map all airports and major train stations in start/end regions — not just the obvious ones
- Hub airports are typically cheaper than tourist-town airports; a train connection from a hub can beat a direct tourist-flight on price and schedule
- Check if reversing the route direction improves flight availability or price
- Train + flight combos often beat a single direct flight to a small airport

**Weather integration:**
- Use seasonal climate data (rainfall, temperature ranges) to allocate time per region
- For outdoor-heavy destinations, build in weather-flexible alternatives
- In rainy seasons: prioritize morning activities (afternoon storms are common), allocate float days for must-see outdoor spots

**Phase 3 complete when:** flight/train alternatives compared for start and end segments, cost trade-offs presented where significant, and every outdoor-heavy day has a weather-flexible fallback.

### Phase 4: HTML Output

Generate a single self-contained HTML file. Must be **responsive** — functional on mobile (single-column) and desktop (2–3 column grid).

**File naming:** `<region>-<route-type>-<year>.html` (e.g., `western-sichuan-oct-2026.html`).

**CSS:** Embed `style-template.css` from the skill directory in a `<style>` block. The template is the default "hand-drawn sketch" style; adapt fonts, colors, and border-radius to match the user's aesthetic preference.

**Structure:** See [`html-output-spec.md`](html-output-spec.md) for the full specification. Key sections in order:

1. **Header** — title, subtitle, stats bar, route-flow pills (≤10 pills, major stops only)
2. **Interactive Leaflet map** — 高德 + OSM layer switcher, route polylines with intermediate waypoints, circle markers click-to-scroll to day cards, detour spurs in distinct dash style
3. **Segment summary table** — every leg with distance, time, road number. **Single source of truth** for segment data; day cards reference but don't repeat this data
4. **Day cards** — grouped by section. Each card: activities / food / accommodation / note-callout. Note-callouts must NOT contain "明天去…" previews
5. **Accommodation summary** — lookup table: night, location, name, price, ONE critical constraint
6. **Packing checklist** — dynamically generated from trip context
7. **Important reminders** — cross-cutting only (each item applies to ≥3 days)
8. **Budget estimate** — major categories + total range
9. **Footer**

**Anti-duplication rules:**
- Segment data lives in the segment table; day cards don't repeat distances or drive times in prose
- Note-callouts never preview the next day (that info is in the next day card + segment table)
- Reminders are cross-cutting (≥3 days); day-specific warnings stay in day card note-callouts
- Weather: daily in day card tags, seasonal patterns in reminders (not a day-by-day list)
- Accommodation table is a lookup index, not a reprint of day card lodging sections
- Route-flow pills: ≤10 items for a 5-second scan

**Traveler-facing principle:** Every visible string should read as natural advice from a knowledgeable friend — no internal scoring, no priority rankings, no planning meta-language. Day-of-week labels must match the actual calendar year for the trip dates.

**Privacy:** The HTML is a public travel guide. If a line isn't about the trip itself, it doesn't belong.

**Phase 4 complete when:** the completion checklist in `html-output-spec.md` is fully satisfied.

### Phase 5: Deployment

Ask whether to publish online. If yes, deploy to GitHub Pages — create a new public repo, push the HTML files, enable Pages. Keep descriptive filenames; add an `index.html` landing page to list all trips.

**Before deploying:** run a privacy scan — re-read the HTML with the question: "does anything here not belong in a public travel guide?" Strip anything that doesn't.

**Phase 5 complete when:** user has been asked and either declined, or the deployed URL is confirmed live and the privacy scan passes.

## Reference Data

Region-specific defaults to verify against current conditions. For each region referenced, use WebSearch to verify 2–3 key facts (car rental policy changes, seasonal road closures, new train routes, major attraction closures) before finalizing the route.

### China — Car Rental

- Major providers: 一嗨租车 (eHi), 神州租车 (CAR Inc.)
- Same-city return: no extra fee
- Same-province cross-city return: ~¥500–1,000
- Cross-province return: ¥2,000–4,000+
- When cross-province fee exceeds ~¥2,000, split into local rentals connected by train
- Summer (July–August) peak season — pre-book early

### China — Southwest (Yunnan, Guizhou, Sichuan, Tibet)

- **Terrain:** mountainous, winding roads; daily driving 200–300km relaxed, 300–500km moderate
- **Weather:** July–August rainy season; waterfalls at peak flow, but mountain roads have landslide risk; afternoon thunderstorms common — plan outdoor activities for mornings
- **Altitude:** Kunming ~1,900m, Lijiang ~2,400m, Shangri-La ~3,300m, Daocheng ~3,700m+; gradual ascent, avoid jumping from sea level to >3,000m in one day
- **Airports:** hub airports (Kunming, Guiyang, Chengdu) significantly cheaper than tourist airports (Lijiang, Shangri-La, Daocheng Yading); train from hub to destination can beat direct flight on price
- **Road conditions:** highways connect major cities; secondary roads in mountainous areas may be single-lane or poorly maintained — factor extra time

### China — Northwest (Xinjiang, Gansu, Qinghai)

- **Terrain:** long flat highways between sparse towns; daily driving 400–700km
- **Weather:** extreme temperature swings (hot days, cold nights); generally dry but sandstorms possible in spring
- **Altitude:** Qinghai plateau >3,000m; same acclimatization concerns as Southwest

### China — Coastal / Eastern

- **Terrain:** dense highway network, flat terrain; daily driving 400–600km
- **Transport:** high-speed rail network is comprehensive — often faster and cheaper than driving between cities
- **Weather:** typhoon risk July–September along southeast coast; plan flexible days
