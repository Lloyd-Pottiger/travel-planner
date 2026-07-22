# Travel Planner Skill

Use this skill when the user wants to plan a multi-day road trip or domestic travel itinerary in China. The output is an interactive HTML travel planner with hand-drawn sketch aesthetics.

## Methodology

### Phase 1: Batch-Grill-Me (Information Gathering)

Ask 3-4 questions at a time using `AskUserQuestion`. Group related questions. Cover these dimensions in order:

**Round 1 — Basics:**
- Travel companions (solo, couple, family w/ kids, friends) — determines pace and logistics
- Daily driving tolerance (3-4h relaxed, 5-6h moderate, 7-8h aggressive)
- Trip themes (nature scenery, minority culture, food, relaxation/escape heat)
- Accommodation style + budget per night

**Round 2 — Route & Logistics:**
- Car: own car (ship back via 拖车) vs rental (异地还车 fee?) vs mix with high-speed rail
- For each province/region on the route: which specific sub-regions interest them?
- Previous travel experience — have they been to any of the destinations before?
- Must-see items or hard constraints (dates, meeting friends, etc.)

**Round 3 — Refinement:**
- Total budget range (for 2 people, excluding flights)
- Weather attitude (accept rain, try to avoid, flexible)
- Regional priority trade-offs when geography conflicts with time
- Flight price sensitivity — willing to adjust route to save on flights?

### Phase 2: Route Design

**Principles:**
1. Minimize backtracking — the route should flow in one general direction
2. Match driving distance to daily tolerance (200-300km for relaxed, 300-500km moderate)
3. Use high-speed rail to skip boring highway segments (e.g., Guangzhou→Guilin, Kunming→Lijiang)
4. Car rental segments should minimize cross-province one-way fees
5. Consider altitude acclimatization (gradual ascent: 0→1900→2400→3300m)
6. July-August in SW China = rainy season → waterfalls at peak flow, but mountain roads have risks

**Transport mode selection:**
- Highway segments with no scenery → high-speed rail
- Scenic mountain/coastal roads → self-drive
- Within a city/region → self-drive for flexibility
- Cross-province one-way rental → check if fee exceeds ¥2000, if so, split into local rentals with train in between

**Car rental strategy (China):**
- 一嗨租车 (eHi) and 神州租车 (CAR Inc) are the main players
- Same-city return: no extra fee
- Same-province cross-city: moderate fee (~¥500-1000)
- Cross-province: high fee (~¥2000-4000+) — avoid unless budget allows
- Pre-book early for summer (July-August is peak season)

### Phase 3: Flight & Weather Optimization

- Compare flight prices for ALL airports in the endpoint region
- General rule: hub airports (Kunming, Guiyang) are cheaper than tourist airports (Lijiang, Shangri-La)
- Check if reversing the route would save on flights
- Combine train+flight if it's cheaper and not much slower
- Weather: July-August rainfall patterns should inform which areas get more/less time

### Phase 4: HTML Output

Generate a single self-contained HTML file with:

**Map section:** SVG route diagram (NOT Leaflet — too heavy, too much irrelevant info)
- Vertical timeline layout with color-coded sections
- Solid lines for driving, dashed for trains, dotted for flights
- Transport mode + duration on every segment
- Clickable nodes that scroll to day cards
- Node sizes: large for hubs/key destinations, medium for stays, small for transits
- viewBox should accommodate the tallest possible route (~960px for 15 nodes)

**Day cards (grid layout, `minmax(440px, 1fr)` for 2-3 per row):**
- Each card: date, route badge, quick-info tags (weather, drive time, transport), then sections for 玩什么/吃什么/住哪里, and a note callout
- Cards get slight random rotation (--r: -0.5deg to 0.5deg) for hand-drawn feel
- Pastel color variants: yellow, pink, blue, green, purple, orange

**Hand-drawn sketch aesthetic:**
- Dot-grid paper background (`radial-gradient`)
- Handwriting fonts: "Ma Shan Zheng" for Chinese headings, "Caveat" for English/numbers
- System fonts for body text (readability > style)
- Asymmetric border-radius on cards (mild, not extreme: `12px 8px` not `18px 4px`)
- Card rotation: max ±0.5deg (subtle)
- Route labels and info tags: NO rotation, balanced border-radius
- Hard box-shadows (not blurry glow)

**Additional sections:**
- Route flow overview (inline pills showing the entire route at a glance)
- Stats bar (total days, driving distance, train segments, provinces, buffer days)
- Packing checklist organized by category (证件, 衣物 by layer, 药品, 车用品, 电子设备, 日用品)
- Important reminders (altitude, rainy driving, advance bookings, food safety, luggage strategy)

**CSS rules:**
- No `transform: rotate()` on functional tags (route labels, info badges)
- SVG fills must use hardcoded colors (CSS variables don't work in SVG)
- Responsive: single column on mobile, 2-3 columns on desktop
- Print-friendly: hide backgrounds, avoid card breaks
- Zero external dependencies except Google Fonts

### Phase 5: Deployment

- Rename to `index.html`
- `git init` + push to GitHub as `username/repo-name`
- Enable GitHub Pages via API: `gh api repos/user/repo/pages -X POST --input - <<<'{"source":{"branch":"main","path":"/"}}'`
- Verify with curl after ~30s build time

## Common Pitfalls

1. **Over-rotating elements**: Max 0.5° for cards, 0° for functional tags. The hand-drawn feel comes from fonts and border-radius, not extreme tilting.

2. **SVG viewBox too small**: Calculate the Y coordinate of the last node + its radius + padding. For 15 nodes at ~60px spacing, viewBox height needs ~960px.

3. **SVG CSS variables**: `fill="var(--color)"` does NOT work in SVG. Use hardcoded hex values.

4. **Too many cards per row**: `minmax(350px, 1fr)` gives 4-5 cards on a wide screen. Use `minmax(440px, 1fr)` for 2-3 max.

5. **Forgetting transport labels**: Every segment between nodes needs a transport mode icon + duration. These go on the LEFT side of the main route line.

6. **Leaflet maps are heavy and low-info**: For a fixed route, an SVG diagram conveys more useful information (duration, mode, nights) than a geographic map full of irrelevant place names.

7. **Day-of-week errors**: Always calculate the actual day of week for the trip year. 2026-07-22 is a Wednesday, so July 24 is Friday.

8. **Priority meta-language in UI**: Never write "优先级：A < B < C" or colored dot rankings in the travel plan itself. Those are internal planning notes, not traveler-facing content.
