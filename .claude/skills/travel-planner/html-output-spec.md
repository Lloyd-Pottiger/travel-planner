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

## Route diagram — SVG timeline

- Prefer SVG over Leaflet — map tiles add weight and irrelevant data for a trip overview
- Vertical layout with color-coded section backgrounds by region/phase
- Line styles encode transport mode: solid = driving, dashed = train, dotted = flight
- **Layout rule** — place transport labels on one side of the route line, node text on the other, to prevent overlap. Default: labels left, text right (assuming LTR reading flow)
- Clickable nodes scroll to the corresponding day cards
- Node sizes reflect importance: major stops > overnight stays > pass-through points
- Dynamically calculate `viewBox` height to fit all nodes with padding
- SVG fills must use hardcoded hex colors — CSS custom properties are not resolved inside SVG elements
- Touch-friendly tap targets for mobile (minimum 44×44px tap area on interactive nodes)

## Day cards

- CSS Grid with `minmax(340px, 1fr)` for 2-3 columns on desktop, single column on mobile
- Each card contains: day number + date, route badge, quick-info tags (weather, drive time, transport mode), sections for activities / food / accommodation, and a contextual note callout
- Section labels in the traveler's language (e.g., 玩什么 / 吃什么 / 住哪里 for Chinese trips)
- Subtle random rotation on cards (`±0.5deg max`) for a handmade feel; functional UI elements (tags, badges) stay unrotated
- Pastel color variants rotated across cards

## Additional sections

**Route flow overview** — inline pills showing the full route at a glance.

**Stats bar** — total days, driving distance, train/flight segments, regions covered, buffer days.

**Packing checklist** — generate dynamically based on trip type, weather, activities, and transport mode. Categories are determined by the trip, not hardcoded (e.g., road trip → car supplies; beach → swim gear; mountain → layers and rain gear).

**Important reminders** — derived from the specific trip: altitude warnings, rainy driving, advance booking needs, food/water safety, luggage strategy.

## Completion checklist

Verify before finalizing the HTML output:

- No stretch of 3+ consecutive one-night stays (cluster 2-3 nights per base per universal principle 4)
- All day-of-week labels verified against the actual calendar year for the trip dates
- CSS grid uses `minmax(340px, 1fr)`, no element has a fixed width exceeding 380px, and font sizes use `clamp()` or relative units — ensures readability at 384px mobile viewport
- SVG `viewBox` height is correct — no nodes are clipped
- Transport labels and node text are on opposite sides of the route line (no overlap)
