# Handoff: Asia Trip Visual Itinerary (asia-trip-visual-itinerary.html)

Instructions for an LLM (or developer) picking up work on this file. Read this fully before editing. The HTML file accompanying this document is the single source of truth for the itinerary — everything (styles, markup, JS) lives in that one self-contained file.

## 1. What this is

A single-file HTML visual itinerary for an 11-day Asia trip, **Aug 20–30, 2026** (Houston → Taipei → Seoul → Kyoto/Osaka → Tokyo → home via LAX). It is a dark, animated, interactive page intended for viewing in a browser (desktop + mobile) with a clean light print stylesheet for a paper backup.

Owner context: traveler departs from Houston (IAH), United 1K flyer. The page may be used on a phone mid-trip, so mobile layout and legibility matter.

## 2. Change history (most recent last)

1. **v1 — original**: light editorial design, static. Included a Nara day trip on Aug 29.
2. **v2 — dynamic redesign**: rebuilt as a dark "night market at dusk" theme. Added: live countdown → live trip-status badge, animated SVG flight arc with looping plane, scroll-reveal cards, scroll-spy jump links, scroll progress bar, event type filters, stat count-ups, hover micro-interactions, `prefers-reduced-motion` support. Print stylesheet forces a light layout. All v1 content preserved verbatim.
3. **v3**: **Nara removed entirely** and the Kyoto days rebalanced:
   - **Aug 27 (Day 8)**: Sanjusangendo removed from this day; afternoon retimed (Kiyomizu-dera 2:00, Sannenzaka 3:30) for a slower pace ending in Gion for the 7:30 p.m. Kobe Beef Mouriya dinner.
   - **Aug 29 (Day 10)**: retitled "Fushimi Inari + Gion + Sushi". New flow: Fushimi Inari at dawn → optional Tofuku-ji 9:00 → hotel/coffee recovery block 10:30 → Pontocho lunch 12:30 → Sanjusangendo 2:00 (relocated from Aug 27) → Kennin-ji 3:30 → hotel reset 4:45 → Gion Sushi Matsumoto 7:00.
   - Global updates: subtitle, destinations stat (5→4), jump link label ("Kyoto + Osaka"), route-overview heading ("Four stops…"), aria-labels, and the JS `cityByDay` map for Aug 29. Zero "Nara" strings remain in the file.
4. **v4 — current**: **Market stall-guide layer + new Myeongdong night.**
   - Added a fifth market-food night: **Myeongdong street food, Mon Aug 24, 7:15–9:45 p.m.** The Day 5 evening was split into "Check in at Grand Hyatt Seoul" (6:15 p.m., `tag hotel`) and "Myeongdong street food" (7:15 p.m., `tag fixed`), plus a note-strip warning that carts pack up ~10 p.m. Subtitle updated: "five dedicated market-food nights".
   - Added an interactive **stall-guide bottom sheet** (modal) for all five markets — Raohe, Shilin, Ningxia, Myeongdong, Gwangjang — sourced from an August 2026 deep-research food guide. Each market event has a "Stall guide" button that opens a slide-up sheet with: meta chips (transit, cash, budget), a numbered hit list in walking order (names, prices, ★ must-gets), "if you only have time for 3", tips, and defensive warnings (Myeongdong tourist markups; Gwangjang's Oct–Nov 2025 overcharging controversy).

## 3. File anatomy (find by selector, not line number)

- `<style>` in `<head>` — all CSS. Ordered: tokens (`:root`) → base → `.ambient` → masthead/arc/countdown → toolbar → overview → day cards/events/tags → filters → footer → `prefers-reduced-motion` → responsive (`900px`, `620px`) → `@media print`.
- `<div class="ambient">` — fixed background glow layer, decorative only.
- `<header class="masthead">` — eyebrow, `h1`, subtitle, **animated route arc SVG**, **countdown module**, stat grid.
- `<nav class="toolbar">` — sticky; jump links (scroll-spy), filter chips, print button, scroll progress bar.
- `<section class="overview-grid">` — route-visual card + key reservations card.
- `<section class="days">` — eleven `<article class="day-card">` elements, one per calendar day.
- `<div class="sheet-overlay">` + five `<template id="guide-{market}">` elements — the market stall-guide layer (see §6a).
- `<footer class="trip-footer">`, then one `<script>` (IIFE) with all behavior.

## 4. Design tokens

- Background `--page: #0b111c`; panels `--paper: #131c2b` / `--paper-2: #17233a`; text `--ink: #eef3f9`; muted `#93a2b6`.
- City accents: Taipei teal `#34d1b6`, Seoul coral `#ff6f59`, Kyoto gold `#f3b23a`, travel blue `#5ea4ff`, homebound violet `#a98ef5`. Each has a `-soft` translucent variant for tag backgrounds.
- Type: display = **Fraunces** (Google Fonts, falls back to Georgia if offline), body = **Inter**. Fonts load from network; the page must remain fully usable with fallbacks.

## 5. Content model — editing rules

**Day card pattern** (city class controls the accent color):
```html
<article class="day-card kyoto">            <!-- taipei | seoul | kyoto | travel-day | homebound -->
  <header class="day-header">
    <time class="date-tile" datetime="2026-08-27"><span>Thu</span><strong>27</strong></time>
    <div class="day-title"><h3>Title</h3><p>Hotel · Night X of Y</p></div>
    <span class="day-number">Day 8</span>
  </header>
  <div class="schedule"> ...events... </div>
  <div class="note-strip"><strong>Label</strong><span>Note text.</span></div>  <!-- optional -->
</article>
```

**Event pattern** (events must stay in chronological order; times are local to the destination):
```html
<div class="event">
  <div class="event-time">2:00 p.m.</div>
  <div class="event-body">
    <div class="event-topline"><strong>Name</strong><span class="tag flexible">Temple</span></div>
    <p>One or two sentences of guidance.</p>
  </div>
</div>
```

**Tag taxonomy** — exactly one tag per event; the class drives color and filtering:
- `tag travel` — flights, trains, transfers
- `tag fixed` — reservations/tickets that cannot move
- `tag flexible` — movable plans
- `tag hotel` — check-in/out, recovery blocks (not filterable via chips; that is intentional)

The visible tag text is free-form ("Flight", "Food night", "Temple"); only the class matters functionally.

## 6. JavaScript behaviors (single IIFE at the bottom)

1. **Countdown / live status**: `departure = 2026-08-20T09:00:00-05:00`, `tripEnd = 2026-08-30T12:30:00-07:00`. Before departure: DD/HH/MM/SS tiles tick every second. During the trip: tiles hide and a pulsing badge shows "Trip in progress · Day N · {city}" from the `cityByDay` map (keys are calendar dates 20–30). After: "Trip complete" badge. **If dates or the Kyoto plan change, update both the Date constants and `cityByDay`.**
2. **Stat count-up**: `.count-up[data-count]` animates 0→N on load (skipped under reduced motion). The four stats are calendar days 11, destinations 4, hotel nights 9, fixed events 5 — recount if reservations change.
3. **Scroll reveal**: IntersectionObserver adds `.in` to `.day-card`/`.overview-card`; events inside stagger in with inline styles that are cleared afterward (do not leave inline opacity on `.event`, it breaks filtering).
4. **Scroll progress + scroll spy**: progress bar `#progress-bar` fills with scroll depth; jump links highlight based on `data-target` matching card ids `day-1`, `taipei`, `seoul`, `kyoto`, `homebound`. **Do not remove or rename these ids.**
5. **Filters**: chips set `body[data-filter]`; CSS dims non-matching events via `:not(:has(.tag.X))`. Requires `:has()` support (all evergreen browsers). Clicking the active chip or "All" clears the filter.
6. **Stall-guide sheet**: `.guide-open` buttons (with `data-guide` + `data-city`) clone the matching `<template id="guide-{id}">` into `#sheet-content` and open the `#sheet-overlay` bottom sheet. Closes via the × button, backdrop click, or Escape; focus returns to the trigger; `body.sheet-open` locks page scroll.

## 6a. Stall-guide layer — how it works and how to edit

- **Trigger pattern** (inside an event's `.event-body`, after the `<p>`):
  ```html
  <button class="guide-open" type="button" data-guide="raohe" data-city="taipei">Stall guide</button>
  ```
  `data-guide` must match a `<template id="guide-{value}">`. `data-city` is `taipei` or `seoul` and sets the sheet's accent color (add a `.sheet.{city}` rule if introducing a new city).
- **Template content structure** (keep this shape for consistency): `.guide-kicker` (market type · day · window) → `<h2 id="sheet-title">` (name; the id feeds the dialog's `aria-labelledby` — exactly one instance is ever in the DOM) → `.guide-meta` chips (transit / cash / budget) → `.guide-lede` (2–3 sentence honest verdict + timing constraint) → `<h3>` + `.stall-list` (numbered `<li>` in **walking order** — the numbers are the route; each has `.stall-top` with name, `.stall-price`, optional `.stall-star` ★ for must-gets, then a one-to-two-sentence `<p>`) → `.guide-cols` with two `.guide-block`s ("If you only have time for 3" + tips/skips) → optional `.guide-warn` for scam/price warnings.
- **Currency**: use `&#8361;` for ₩ and plain `NT$`; prices are directional 2025–2026 figures from the source research.
- **Content provenance**: distilled from an Aug 2026 deep-research guide. Key facts preserved: Michelin "Bib Gourmand" labels on most Raohe/Ningxia stalls lapsed 2019–2024 (the guides note this; don't re-add present-tense medal claims); current genuine Michelin recognition is Seoul sit-downs (Buchon Yukhoe, Myeongdong Kyoja since 2017, Hadongkwan — daytime only); Shilin B1 food court reopened April 2025 with 539 stalls; Gwangjang Oct–Nov 2025 overcharging scandal (order defensively); day-of-week open/closed status was verified for the actual visit dates (all ✓).
- **Print**: the sheet and buttons are hidden in print (`.sheet-overlay, .guide-open { display:none }`). If a printable version of the guides is requested, render templates as appendix pages rather than un-hiding the modal.

## 7. Invariants — do not break

- Single self-contained file; no build step, no external JS, no localStorage/sessionStorage.
- Keep the `@media print` block working: light background, hero/toolbar/countdown hidden or simplified, 2-column day cards, `print-color-adjust: exact` on colored elements.
- Keep `prefers-reduced-motion` support (arc pre-drawn, plane hidden, reveals instant).
- The plane animation's `offset-path` in `.arc-plane` must remain **identical** to the `d` attribute of `.arc-path` — if you edit the arc geometry, change both.
- Preserve aria-labels/roles, `aria-pressed` on filter chips, and the guide sheet's `role="dialog"` / `aria-modal` / Escape + backdrop close behavior.
- The five market nights (Raohe Aug 21, Shilin Aug 22, Ningxia Aug 23, Myeongdong Aug 24, Gwangjang Aug 25) each carry exactly one `.guide-open` button wired to exactly one template — keep buttons, templates, and dates in sync if markets change.
- Times/route facts below are booked reality — do not alter them when restyling.

## 8. Fixed facts (source of truth)

- Flights: UA 871 IAH→SFO→TPE (dep Aug 20, arr TPE Aug 21 6:45 p.m.); OZ 712 TPE→ICN Aug 24 12:25 p.m. (arr 3:55 p.m.); OZ 1165 GMP→KIX Aug 26 5:40 p.m. (arr 7:20 p.m.); Nozomi 70 Kyoto→Tokyo Aug 30 7:06 a.m. (arr 9:15 a.m.); UA 38 HND→LAX Aug 30 6:10 p.m. (arr LAX 12:30 p.m. same calendar day).
- Hotels: Grand Hyatt Taipei (3 nights), Grand Hyatt Seoul (2), The Chapter Kyoto (4).
- Fixed reservations (5): Seoul city tour Aug 25 ~10:00 a.m. (tentative); Withlocals Kyoto tour Aug 27 ~9:30 a.m. (tentative); Kobe Beef Mouriya, Gion, Aug 27 7:30 p.m.; Hanshin Tigers vs. Yomiuri Giants, Koshien, Aug 28 6:00 p.m. (2 tickets, SMBC Seat, 3rd-base side; return via Hanshin Koshien → Osaka-Umeda → JR to Kyoto); Gion Sushi Matsumoto Aug 29 7:00 p.m. (2 guests, ¥28,600 course, reservation under "Rodrigo").

## 9. Backlog / candidate next features (not yet built)

- "Now" marker: highlight the current event in real time using each city's timezone (Taipei UTC+8, Seoul/Japan UTC+9); auto-scroll to today's card on load during the trip.
- Printable appendix version of the five stall guides (render templates as print-only pages).
- In-guide checkboxes to tick off stalls as eaten (in-memory only — no browser storage).
- Map links: wrap stall and event names in Google Maps search URLs (`https://www.google.com/maps/search/?api=1&query=...`); for Seoul stalls prefer Naver Map links.
- Light/dark toggle (persist in-memory only — no browser storage).
- Collapse/expand past days once the trip is underway.

## 10. How to work

Make surgical edits; do not regenerate the whole file unless restructuring. After any edit: (a) verify tag balance, (b) grep for stale references when removing a place (as was done for "Nara"), (c) re-check the print view and mobile width ~380px, (d) confirm the countdown constants and `cityByDay` still match the itinerary content.
