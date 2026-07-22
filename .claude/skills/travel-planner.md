# Travel Planner Skill

Use this skill when the user wants to plan a multi-day road trip or domestic travel itinerary. The output is an interactive HTML travel planner with a hand-drawn sketch aesthetic.

## Methodology

### Phase 1: Information Gathering

Use `AskUserQuestion` to batch questions in groups of 3-4 at a time. Adapt the order and content to what the user has already volunteered — skip dimensions they've already covered. The goal is to pin down these key variables, but not necessarily in this order:

- **Travel companions** — determines pace, accommodation needs, and how many can drive
- **Dates & duration** — start/end dates, any hard constraints mid-trip
- **Driving tolerance** — hours per day (3-4h relaxed, 5-6h moderate, 7-8h aggressive)
- **Trip themes** — nature, culture, food, relaxation, or a mix
- **Route direction** — starting point, general direction, endpoint, any must-hit places
- **Transport** — self-drive, rental car (异地还车 fees?), high-speed rail, or hybrid
- **Accommodation & budget** — style and nightly range; total budget for the trip
- **Previous experience** — which destinations on the route have they already visited?
- **Weather attitude** — accept rain, try to avoid, or flexible day-by-day
- **Regional trade-offs** — when geography and time conflict, which areas get priority?
- **Flight price sensitivity** — willing to adjust route or endpoint to save on flights?

Don't ask all of these in one session. Feel out which ones are most relevant based on the trip the user describes. If the user explicitly says `batch-grill-me`, front-load the most decision-critical questions.

### Phase 2: Route Design

**Principles:**
1. Minimize backtracking — the route should flow in one general direction
2. Match driving distance to daily tolerance (200-300km for relaxed, 300-500km moderate)
3. Use high-speed rail to skip boring highway segments
4. Car rental segments should minimize cross-province one-way fees
5. Consider altitude acclimatization (gradual ascent better than sudden)
6. Factor in seasonal weather patterns — e.g., July-August in SW China = rainy season, waterfalls at peak flow but mountain roads have risks

**Transport mode selection:**
- Highway segments with no scenery → high-speed rail
- Scenic mountain/coastal roads → self-drive
- Within a city/region → self-drive for flexibility
- Cross-province one-way rental → if fee exceeds ~¥2000, consider splitting into local rentals with a train segment in between

**Car rental in China:**
- 一嗨租车 and 神州租车 are the main players
- Same-city return: no extra fee; same-province cross-city: ~¥500-1000; cross-province: ¥2000-4000+
- Pre-book early for summer — July-August is peak season

### Phase 3: Flight & Weather Optimization

- Compare flight prices for all airports in the endpoint region
- Hub airports (Kunming, Guiyang) are generally cheaper than tourist airports (Lijiang, Shangri-La)
- Check if reversing the route would save on flights
- Train+flight combos can beat direct tourist-airport flights
- Use weather data (rainfall, temperature) to inform how much time each area gets

### Phase 4: HTML Output

Generate a single self-contained HTML file with:

**Route diagram:** SVG timeline (NOT Leaflet — too heavy, too much irrelevant map data)
- Vertical layout with color-coded section backgrounds
- Solid lines for driving, dashed for trains, dotted for flights
- Transport mode + duration labels on every segment
- Clickable nodes that scroll to the relevant day cards
- Node sizes vary by importance (hubs > stays > transits)
- viewBox must accommodate all nodes — calculate tallest Y + padding

**Day cards** (CSS Grid, `minmax(440px, 1fr)` for 2-3 per row):
- Each card: day number + date, route badge, quick-info tags (weather, drive time, transport mode), sections for 玩什么 / 吃什么 / 住哪里, and a `📌` note callout
- Cards get subtle random rotation (--r: ±0.5deg max)
- Pastel color variants: yellow, pink, blue, green, purple, orange

**Hand-drawn sketch aesthetic:**
- Dot-grid paper background (`radial-gradient`)
- "Ma Shan Zheng" for Chinese headings, "Caveat" for English/numbers, system fonts for body text
- Mild asymmetric border-radius on cards (`12px 8px`, not extreme `18px 4px`)
- Functional tags (route labels, info badges): NO rotation, balanced border-radius
- Hard box-shadows, not blurry glow
- Zero external dependencies except Google Fonts

**Additional sections:**
- Route flow overview — inline pills showing the entire route at a glance
- Stats bar — total days, driving distance, train segments, provinces, buffer days
- Packing checklist — organized by category (证件, 衣物 by layer, 药品, 车用品, 电子设备, 日用品)
- Important reminders — altitude, rainy driving, advance bookings, food safety, luggage strategy

**Key CSS/SVG rules:**
- SVG fills must use hardcoded hex colors — CSS variables don't work in SVG
- Transport labels go on the LEFT side of the main route line, node text on the RIGHT
- Never put meta-language in traveler-facing UI (no "优先级A<B", no colored ranking dots)
- Always verify day-of-week against the actual calendar year

### Phase 5: Deployment (Optional)

If the user wants to publish:
- Rename to `index.html`, `git init`, push to GitHub
- Enable GitHub Pages: `gh api repos/user/repo/pages -X POST --input - <<<'{"source":{"branch":"main","path":"/"}}'`
- Site will be live at `https://<user>.github.io/<repo>/` within ~30s
