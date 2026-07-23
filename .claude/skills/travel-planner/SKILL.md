---
name: travel-planner
description: Plan a multi-day road trip or domestic travel itinerary.
---

# Travel Planner Skill

## Pitfalls to Avoid

- **Don't overwrite existing trip plans.** Always check if `index.html` or other HTML files already exist in the project. Save new plans with descriptive filenames (see Phase 4 naming convention). The user may have multiple trips in the same project.
- **Don't assume the user wants deployment.** Phase 5 is opt-in only — ask before publishing anything online.

## Methodology

### Phase 1: Information Gathering

Default to asking questions before designing anything — unclear requirements produce bad routes. Don't stop after a fixed number of rounds; keep asking until the key dimensions are sufficiently clear. Batch 3-4 questions per round, prioritize the highest-variance dimensions first, and skip what the user has already covered. Stop when the remaining unknowns wouldn't materially fork the route design (e.g., once the region and route shape are locked in, flight price sensitivity rarely changes the plan). Rarely need more than 3 rounds for a domestic trip.

Use `AskUserQuestion` to batch questions. Key dimensions to pin down (adapt order to what's already known):

- **Travel companions** — determines pace, accommodation needs, how many can drive
- **Dates & duration** — start/end dates, hard constraints mid-trip
- **Driving tolerance** — hours per day; adjust baseline by terrain (mountain roads: 3-5h, highways: 5-8h)
- **Trip themes** — nature, culture, food, relaxation, or a mix — drives routing and activity choices
- **Route shape** — starting point, general direction, endpoint, any must-hit places
- **Transport mode** — self-drive, rental car, train, or hybrid
- **Accommodation & budget** — style, nightly range, total trip budget
- **Previous experience** — which destinations on the route have they already visited?
- **Weather attitude** — accept rain, try to avoid, or decide day-by-day
- **Regional trade-offs** — when geography and time conflict, which areas get priority?
- **Flight price sensitivity** — willing to adjust endpoint or reverse route to save on flights?

### Phase 2: Route Design

**Universal principles:**
1. Minimize backtracking — the route should flow in one general direction
2. Match daily driving distance to the user's tolerance, adjusted for terrain and road type
3. Use rail to skip highway segments with no scenery or sightseeing value
4. Prefer one-way loops over out-and-back when the geometry allows it
5. Factor in seasonal weather — rainy season affects mountain road safety, peak summer means crowded attractions and tight bookings
6. Match activities to weather requirements — some activities only work in specific conditions:
   - Sunrises, mountain panoramas, stargazing → need clear or mostly-clear skies; don't schedule these on forecasted rainy days
   - Waterfalls, river rafting → best after or during rain when water volume peaks
   - Snow/ice scenery → temperature-dependent; check seasonal viability
   - Indoor/cultural alternatives → have a backup plan for each weather-sensitive outdoor day
   - When a must-do activity is weather-dependent, allocate a float day nearby so the schedule can shift
7. Consider altitude acclimatization when the route involves rapid elevation gain
8. Place buffer days after long drives or before critical connections

**Transport mode selection logic:**
- Boring highway / no scenery → train or flight
- Scenic roads (mountain passes, coastal routes) → self-drive
- Within a city or compact region → self-drive for flexibility
- Cross-region one-way rental with high drop-off fees → split into local rentals connected by train
- For each segment, compare: self-drive (rental + fuel + tolls + drop-off fee) vs. train/flight (ticket + local transport at destination); present the trade-off when the difference is significant

**Accommodation strategy:**
- Match accommodation style to trip type (guesthouses for cultural trips, hotels for city stops, resorts for beach/relaxation)
- When moving daily, cluster stays — 2-3 nights per base with day trips beats 1-night-stands
- Pre-book for peak season; off-season can stay flexible
- For each stop, research 2-3 concrete options within the user's budget range and present them as choices — include name, rough price, location advantage, and a one-line reason it fits. Let the user pick rather than guessing their preference. Source recommendations from: Trip.com / 携程 for China domestic, Booking.com / Agoda for international, 大众点评 for local guesthouse reviews

**Phase 2 complete when:** every segment has a transport mode assigned, daily driving stays within tolerance (adjusted for terrain), all 8 universal principles are satisfied, and accommodation options have been presented for user selection.

### Phase 3: Transport & Weather Optimization

**Flight/train route optimization:**
- Map all airports and major train stations in the start and end regions — not just the obvious ones
- Hub airports are typically cheaper than tourist-town airports; a train connection from a hub can beat a direct tourist-flight on both price and schedule
- Check if reversing the route direction improves flight availability or price
- Train + flight combos often beat a single direct flight to a small airport

**Weather integration:**
- Use seasonal climate data (rainfall, temperature ranges) to allocate time per region
- For outdoor-heavy destinations, build in weather-flexible alternatives
- In rainy seasons: prioritize morning activities (afternoon storms are common in many regions), allocate float days for must-see outdoor spots

**Phase 3 complete when:** flight/train alternatives have been compared for start and end segments, cost trade-offs presented where significant, and every outdoor-heavy day has a weather-flexible fallback.

### Phase 4: HTML Output

Generate a single self-contained HTML file. The output must be **responsive** — functional on both mobile (single-column, full-width cards) and desktop (2-3 column grid).

**File naming:** Use a descriptive name that identifies the trip — never hardcode `index.html` as the output filename. Pattern: `<region>-<route-type>-<year>.html` (e.g., `western-sichuan-oct-2026.html`, `guizhou-loop-jul-2026.html`). If the project already has an `index.html`, do NOT overwrite it — save with a distinct name. Only use `index.html` during Phase 5 deployment (where it's required for GitHub Pages in a separate directory).

**Visual style — default to hand-drawn sketch, adapt to user preference:**
- Default: dot-grid paper background (`radial-gradient`), mild asymmetric border-radius, hard box-shadows, paper-like feel
- Fonts: choose a pairing that matches the aesthetic — handwritten/display font for headings, clean system font for body text. For Chinese trips: "Ma Shan Zheng" for headings, "Caveat" for Latin/numbers, system fonts for body
- Zero external dependencies except Google Fonts
- If the user prefers a different style (minimal, modern, etc.), adapt accordingly

**Route diagram — SVG timeline:**
- Prefer SVG over Leaflet — map tiles add weight and irrelevant data for a trip overview
- Vertical layout with color-coded section backgrounds by region/phase
- Line styles encode transport mode: solid = driving, dashed = train, dotted = flight
- **Layout rule** — place transport labels on one side of the route line, node text on the other, to prevent overlap. Default: labels left, text right (assuming LTR reading flow)
- Clickable nodes scroll to the corresponding day cards
- Node sizes reflect importance: major stops > overnight stays > pass-through points
- Dynamically calculate `viewBox` height to fit all nodes with padding
- SVG fills must use hardcoded hex colors — CSS custom properties are not resolved inside SVG elements
- Touch-friendly tap targets for mobile (minimum 44×44px tap area on interactive nodes)

**Day cards:**
- CSS Grid with `minmax(440px, 1fr)` for 2-3 columns on desktop, single column on mobile
- Each card contains: day number + date, route badge, quick-info tags (weather, drive time, transport mode), sections for activities / food / accommodation, and a contextual note callout
- Section labels in the traveler's language (e.g., 玩什么 / 吃什么 / 住哪里 for Chinese trips)
- Subtle random rotation on cards (`±0.5deg max`) for a handmade feel; functional UI elements (tags, badges) stay unrotated
- Pastel color variants rotated across cards

**Additional sections:**
- Route flow overview — inline pills showing the full route at a glance
- Stats bar — total days, driving distance, train/flight segments, regions covered, buffer days
- Packing checklist — generate dynamically based on trip type, weather, activities, and transport mode. Categories are determined by the trip, not hardcoded (e.g., road trip → car supplies; beach → swim gear; mountain → layers and rain gear)
- Important reminders — derived from the specific trip: altitude warnings, rainy driving, advance booking needs, food/water safety, luggage strategy

**Traveler-facing UI principle:**
- The output is for the traveler, not the planner. Every visible string in the UI should read as natural advice from a knowledgeable friend — no internal scoring, no priority rankings, no planning meta-language.
- Day-of-week labels must match the actual calendar year for the trip dates (verified in the completion checklist below).

**Completion checklist — verify before finalizing:**
- No day exceeds the user's driving tolerance (adjusted for terrain)
- No stretch of 3+ consecutive one-night stays (cluster 2-3 nights per base instead)
- Every weather-sensitive outdoor activity has a fallback or float day
- All day-of-week labels verified against the actual calendar year for the trip dates
- HTML renders readably at 384px mobile viewport width
- SVG `viewBox` height is correct — no nodes are clipped
- Transport labels and node text are on opposite sides of the route line (no overlap)

### Phase 5: Deployment

Ask whether to publish online. If yes, deploy to GitHub Pages — create a new public repo, push the HTML files, and enable Pages. Keep descriptive filenames; add an `index.html` landing page to list all trips.

## Reference Data

This section contains region-specific data used to inform route design. Treat these as defaults to verify against current conditions, not immutable facts. For each region referenced in the trip, use WebSearch to verify 2-3 key facts (car rental policy changes, seasonal road closures, new train routes, major attraction closures) before finalizing the route.

### China — Car Rental

- Major providers: 一嗨租车 (eHi), 神州租车 (CAR Inc.)
- Same-city return: no extra fee
- Same-province cross-city return: ~¥500–1,000
- Cross-province return: ¥2,000–4,000+
- When cross-province fee exceeds ~¥2,000, prefer splitting into local rentals connected by train
- Summer (July–August) is peak season — pre-book early

### China — Southwest (Yunnan, Guizhou, Sichuan, Tibet)

- **Terrain:** mountainous, winding roads; daily driving baseline 200–300km relaxed, 300–500km moderate
- **Weather:** July–August is rainy season; waterfalls at peak flow, but mountain roads have landslide risk; afternoon thunderstorms common — plan outdoor activities for mornings
- **Altitude:** Kunming ~1,900m, Lijiang ~2,400m, Shangri-La ~3,300m, Daocheng ~3,700m+; plan gradual ascent, avoid jumping from sea level to >3,000m in one day
- **Airports:** hub airports (Kunming, Guiyang, Chengdu) are significantly cheaper than tourist airports (Lijiang, Shangri-La, Daocheng Yading); a train from a hub to the destination can beat a direct flight on price
- **Road conditions:** highways connect major cities; secondary roads in mountainous areas may be single-lane or poorly maintained — factor extra time

### China — Northwest (Xinjiang, Gansu, Qinghai)

- **Terrain:** long flat highways between sparse towns; daily driving baseline 400–700km
- **Weather:** extreme temperature swings (hot days, cold nights); generally dry but sandstorms possible in spring
- **Altitude:** Qinghai plateau >3,000m; same acclimatization concerns as Southwest

### China — Coastal / Eastern

- **Terrain:** dense highway network, flat terrain; daily driving baseline 400–600km
- **Transport:** high-speed rail network is comprehensive — often faster and cheaper than driving between cities
- **Weather:** typhoon risk July–September along southeast coast; plan flexible days
