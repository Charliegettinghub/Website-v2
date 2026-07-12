# Instagram-style photo carousel for print cards

## Problem

The Prints section (`index.html` `#prints`) has four `.print-card` slots, each with a single static photo placeholder. Charlie has photos of a real project (a 3D-printed wall-mount key box) and wants to show multiple photos per card, browsable one-at-a-time with an obvious "there's more" affordance — swipe on touch, arrows/dots on desktop — the same interaction pattern as an Instagram multi-photo post.

## Goals

- Reusable multi-photo carousel component for any print card, not just this one.
- Native-feeling swipe on touch devices; visible arrow buttons + dot indicators on desktop.
- No new dependencies, no build step — plain HTML/CSS/vanilla JS, matching the rest of the site.
- Cards with only one photo are completely unaffected (no carousel chrome, no extra markup weight).
- Fill in the first print card with real content: the "MCS Work Key Holder" project.

## Non-goals

- Autoplay, lightbox/fullscreen zoom, pinch-zoom, video support — none requested.
- A CMS/JSON-driven data model — cards stay hand-authored HTML like the rest of the site.

## Approach

**CSS scroll-snap carousel.** The photo track is a horizontally scrolling flex container (`overflow-x: auto; scroll-snap-type: x mandatory`). Touch/trackpad swipe paging comes from the browser natively — no hand-rolled touch-event/drag-physics JS. Arrow buttons and dots are thin JS wrappers around `scrollBy()` / `scrollTo()`; the active dot is kept in sync by listening to the track's `scroll` event.

Rejected alternatives:
- **JS transform-driven carousel** (index-based `translateX`, manual touch handlers) — reimplements swipe physics the browser gives for free; more code and edge cases for no real benefit here.
- **CDN carousel library (e.g. Embla)** — the site is deliberately zero-dependency/no-build-step; a CDN script is an unnecessary new failure mode for a fairly simple UI need.

## Component design

### Markup

A `.print-carousel` wrapper replaces `.print-card__photo`'s inner content *only* for cards with 2+ photos. Single-photo cards keep today's plain `<div class="print-card__photo"><span>add photo</span></div>` (or a plain `<img>` once filled in) — untouched.

```html
<div class="print-card__photo print-carousel">
  <div class="print-carousel__track">
    <img class="print-carousel__slide" src="assets/prints/key-box/photo-1.jpg" alt="...">
    <img class="print-carousel__slide" src="assets/prints/key-box/photo-2.jpg" alt="...">
    <img class="print-carousel__slide" src="assets/prints/key-box/photo-3.jpg" alt="...">
    <img class="print-carousel__slide" src="assets/prints/key-box/photo-4.jpg" alt="...">
  </div>
  <button class="print-carousel__arrow print-carousel__arrow--prev" aria-label="Previous photo">‹</button>
  <button class="print-carousel__arrow print-carousel__arrow--next" aria-label="Next photo">›</button>
  <div class="print-carousel__dots" role="tablist" aria-label="Photo navigation"></div>
</div>
```

The `.print-carousel__dots` container starts **empty** in HTML — JS populates one dot button per `.print-carousel__slide` found, so adding photos to a future project is just adding `<img>` tags; nobody has to hand-count dots.

### Visuals

- Slides use `object-fit: cover` inside the card's existing photo aspect-ratio frame (4:3 on odd cards, 1:1 on even cards per current `nth-child` rule) — so carousel cards sit flush with single-photo cards, no layout special-casing.
- Arrow buttons: small circular buttons, dark translucent background, positioned vertically centered on the left/right edges of the photo. Hidden by default, faded in via `@media (hover: hover)` + `.print-carousel:hover` — no dead hover state stuck visible on touch devices.
- Dots: centered along the bottom edge of the photo, inactive = translucent white/grey dot, active = `var(--accent)` (the site's existing orange accent), consistent with accent usage elsewhere (spec line, link hovers).
- Scrollbar hidden on the track (`scrollbar-width: none` / `::-webkit-scrollbar { display: none }`) since dots are the position indicator.

### Behavior (script.js)

- On load, `initPrintCarousels()` finds every `.print-carousel`, reads its slides, and:
  - builds the dot row (one button per slide, first marked `is-active`)
  - wires prev/next arrow clicks to `track.scrollBy({ left: ±trackWidth, behavior: 'smooth' })`
  - wires dot clicks to `track.scrollTo({ left: slideIndex * trackWidth, behavior: 'smooth' })`
  - listens to the track's `scroll` event (throttled via `requestAnimationFrame`, same pattern as the existing cursor-spotlight handler) to compute the nearest slide from `scrollLeft / clientWidth` and keep the active dot in sync during manual swipe/drag
- Track is focusable (`tabindex="0"`) and responds to `ArrowLeft`/`ArrowRight` keys, calling the same scroll-to-slide logic as the buttons.
- Respects `prefers-reduced-motion` (already read once in `script.js`) by using `behavior: 'auto'` instead of `'smooth'` for programmatic scrolls.
- Recomputing on resize isn't needed — slide position is derived from `scrollLeft / clientWidth` ratio at interaction time, which stays correct across viewport widths.

### Content: first print card

- Source photos: `assets/prints/key-box/` (`download.jpg`, `download-1.jpg`, `download-2.jpg`, `download-3.jpg` as dropped in — will be renamed to sequential, descriptive filenames during implementation).
- Title: **MCS Work Key Holder**
- Spec line: **Multicolor PLA** (print time/scale intentionally omitted — unknown, and a bare material line reads cleaner than placeholder brackets)
- Description: *"When the wall-mounted box that held our office keys broke, I kept the original mount and designed a new box to slot right onto it — along with matching keychains stamped with our room number and phone number, so a lost set finds its way back."*
- Photo order (mounted product → mechanism → label detail → close-up):
  1. Box mounted on the door with two labeled key sets hanging from it
  2. Box detached from its wall mount, showing the internal key-slot compartments
  3. Close-up of a hand holding a keychain tag printed with the room number and phone number, next to the mounted box
  4. A single key and its printed tag resting on a table

## Edge cases / error handling

- Zero-JS environments: without JS, the track is still a scrollable flex container with `scroll-snap` — swipe/scroll still works, just no dots/arrows. Degrades gracefully.
- A card with exactly 1 photo never gets `.print-carousel` markup — no JS runs against it, no dot row, no cost.
- `IntersectionObserver`/reduced-motion checks already exist in `script.js`; new code reuses those same feature checks rather than re-detecting.

## Testing

Manual verification in the browser preview (no test framework on this static site):
- Desktop: hover reveals arrows, arrow clicks page through all 4 photos, dots reflect position and are clickable, keyboard arrow keys work when track is focused.
- Touch/mobile viewport: swipe gesture pages through photos, dots update.
- Reduced-motion emulation: scroll jumps instead of animating.
- Dark theme (site default) and the darker `.section--dark` Prints band: verify arrow/dot contrast is visible against both real photos and the dark background.
- Confirm the other 3 single-photo cards are visually and functionally unchanged.
