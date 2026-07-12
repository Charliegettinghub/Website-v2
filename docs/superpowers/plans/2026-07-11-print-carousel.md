# Print Card Photo Carousel Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Give print cards an Instagram-style swipeable photo carousel (native scroll-snap paging, arrow buttons + dot indicators on desktop), and fill in the first print card with the real "MCS Work Key Holder" project.

**Architecture:** A reusable `.print-carousel` component wraps `.print-card__photo` only for cards with 2+ photos. The photo track is a native `overflow-x: auto; scroll-snap-type: x mandatory` flex container (swipe/scroll comes from the browser for free). Vanilla JS (`script.js`) auto-builds the dot row from however many `<img class="print-carousel__slide">` tags are in the track, wires arrow/dot clicks to `scrollTo`/`scrollBy`, and keeps the active dot synced to manual scroll position. No dependencies, no build step.

**Tech Stack:** Plain HTML/CSS/vanilla JS (this is a static site, no framework, no test runner, no bundler).

## Global Constraints

- No new dependencies or CDN scripts — this site is intentionally zero-dependency, no build step.
- CSS must use the existing custom properties (`--accent`, `--bg-*`, `--ink-*`, `--ease`, `--shadow`) so it re-themes automatically under `.section--dark`.
- Cards with exactly 1 photo must render identically to today — no carousel markup, no JS attached, no visual change.
- Respect `prefers-reduced-motion` for programmatic scrolls (JS already checks this pattern elsewhere in `script.js`).
- Source photos live in `assets/prints/key-box/` (already present: `download.jpg`, `download-1.jpg`, `download-2.jpg`, `download-3.jpg`), confirmed content order:
  1. `download.jpg` → box mounted on the door with two labeled key sets hanging from it
  2. `download-1.jpg` → box detached from its wall mount, showing the internal key-slot compartments
  3. `download-2.jpg` → close-up of a hand holding a keychain tag printed with the room number/phone number, next to the mounted box
  4. `download-3.jpg` → a single key and its printed tag resting on a table

---

### Task 1: Rename photo assets and build the key-box card markup

**Files:**
- Rename (in place): `assets/prints/key-box/download.jpg` → `assets/prints/key-box/mounted.jpg`
- Rename (in place): `assets/prints/key-box/download-1.jpg` → `assets/prints/key-box/open.jpg`
- Rename (in place): `assets/prints/key-box/download-2.jpg` → `assets/prints/key-box/tag-detail.jpg`
- Rename (in place): `assets/prints/key-box/download-3.jpg` → `assets/prints/key-box/keychain.jpg`
- Modify: `index.html:165-170` (first `.print-card` in `.print-grid`)

**Interfaces:**
- Produces: the DOM structure `.print-carousel > .print-carousel__track > .print-carousel__slide` (img elements) that Task 2 (CSS) and Task 3 (JS) target by class name. Also produces `.print-carousel__arrow--prev`, `.print-carousel__arrow--next`, and an empty `.print-carousel__dots` container that Task 3's JS populates at runtime.

- [ ] **Step 1: Rename the photo files to descriptive names**

```bash
cd "/Users/charlie/Desktop/Claude Code/assets/prints/key-box"
mv download.jpg mounted.jpg
mv download-1.jpg open.jpg
mv download-2.jpg tag-detail.jpg
mv download-3.jpg keychain.jpg
rm -f .DS_Store
ls
```

Expected output: `keychain.jpg`, `mounted.jpg`, `open.jpg`, `tag-detail.jpg` (4 files, no `.DS_Store`, no `download*.jpg`).

- [ ] **Step 2: Replace the first print-card block in index.html**

In `index.html`, replace the first `<article class="print-card">...</article>` block (currently lines 165-170) with:

```html
      <article class="print-card">
        <div class="print-card__photo print-carousel">
          <div class="print-carousel__track">
            <img class="print-carousel__slide" src="assets/prints/key-box/mounted.jpg" alt="3D-printed key box mounted on the office door with two labeled key sets hanging from it">
            <img class="print-carousel__slide" src="assets/prints/key-box/open.jpg" alt="The key box detached from its wall mount, showing the internal key-slot compartments">
            <img class="print-carousel__slide" src="assets/prints/key-box/tag-detail.jpg" alt="Close-up of a keychain tag printed with the room number and phone number, held next to the mounted box">
            <img class="print-carousel__slide" src="assets/prints/key-box/keychain.jpg" alt="A single key and its printed keychain tag resting on a table">
          </div>
          <button class="print-carousel__arrow print-carousel__arrow--prev" aria-label="Previous photo">‹</button>
          <button class="print-carousel__arrow print-carousel__arrow--next" aria-label="Next photo">›</button>
          <div class="print-carousel__dots" role="tablist" aria-label="Photo navigation"></div>
        </div>
        <h3>MCS Work Key Holder</h3>
        <p class="print-card__spec">Multicolor PLA</p>
        <p>When the wall-mounted box that held our office keys broke, I kept the original mount and designed a new box to slot right onto it — along with matching keychains stamped with our room number and phone number, so a lost set finds its way back.</p>
      </article>
```

Leave print cards 2-4 (the other three `<article class="print-card">` blocks) exactly as they are.

- [ ] **Step 3: Verify the markup loads with no styling yet**

Start the dev server (`mcp__Claude_Preview__preview_start`, or open `index.html` directly if no dev server config exists — this is a static file, any static file server works). Then:

Run: `mcp__Claude_Preview__preview_network` filtered to `assets/prints/key-box/`
Expected: 4 requests, all status 200 (no broken image 404s).

Run: `mcp__Claude_Preview__preview_snapshot`
Expected: the first print card shows heading "MCS Work Key Holder", spec text "Multicolor PLA", and the description paragraph. The 4 images will be stacked/unstyled at this point (Task 2 hasn't run yet) — that's expected, don't fix styling here.

- [ ] **Step 4: Commit**

```bash
cd "/Users/charlie/Desktop/Claude Code"
git add index.html assets/prints/key-box
git commit -m "Add MCS Work Key Holder content and carousel markup to first print card"
```

---

### Task 2: Style the carousel component

**Files:**
- Modify: `styles.css` (insert new rules after the existing `.print-card__photo span { ... }` block, i.e. after line 561 in the current file — insert before `.print-card h3`)

**Interfaces:**
- Consumes: the class names produced by Task 1 (`.print-carousel`, `.print-carousel__track`, `.print-carousel__slide`, `.print-carousel__arrow--prev`, `.print-carousel__arrow--next`, `.print-carousel__dots`).
- Produces: `.print-carousel__dot` and `.print-carousel__dot.is-active` class styling that Task 3's JS toggles at runtime (JS creates these elements; this task only needs to style the class names, the elements don't need to exist yet for the CSS to be valid).

- [ ] **Step 1: Add carousel CSS rules**

Insert this block into `styles.css` immediately after the existing `.print-card__photo span { ... }` rule (right before `.print-card h3`):

```css
.print-carousel {
  position: relative;
  overflow: hidden;
  align-items: stretch;
}

.print-carousel__track {
  display: flex;
  width: 100%;
  height: 100%;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  scrollbar-width: none;
  -webkit-overflow-scrolling: touch;
}
.print-carousel__track::-webkit-scrollbar { display: none; }
.print-carousel__track:focus-visible { outline: 2px solid var(--accent); outline-offset: -2px; }

.print-carousel__slide {
  flex: 0 0 100%;
  width: 100%;
  height: 100%;
  object-fit: cover;
  scroll-snap-align: start;
}

.print-carousel__arrow {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 2rem;
  height: 2rem;
  border: none;
  border-radius: 50%;
  background: rgba(13, 14, 15, 0.55);
  color: #f1efec;
  font-size: 1.2rem;
  line-height: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  opacity: 0;
  transition: opacity 0.2s var(--ease), background-color 0.2s var(--ease);
}
.print-carousel__arrow--prev { left: 0.6rem; }
.print-carousel__arrow--next { right: 0.6rem; }
.print-carousel__arrow:hover { background: var(--accent-fill); }

@media (hover: hover) {
  .print-carousel:hover .print-carousel__arrow { opacity: 1; }
}

.print-carousel__dots {
  position: absolute;
  bottom: 0.7rem;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 0.4rem;
}

.print-carousel__dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  border: none;
  padding: 0;
  background: rgba(255, 255, 255, 0.4);
  cursor: pointer;
  transition: background-color 0.2s var(--ease), transform 0.2s var(--ease);
}
.print-carousel__dot.is-active {
  background: var(--accent);
  transform: scale(1.2);
}
```

- [ ] **Step 2: Verify visual styling**

Reload the preview. Run:

`mcp__Claude_Preview__preview_inspect` on selector `.print-carousel` with styles `["border-radius", "overflow", "aspect-ratio"]`
Expected: `border-radius` matches the site's card photo radius (8px, inherited from `.print-card__photo`), `overflow: hidden`.

`mcp__Claude_Preview__preview_inspect` on selector `.print-carousel__arrow--prev` with styles `["opacity", "position"]`
Expected: `opacity: 0` (arrows hidden until hover), `position: absolute`.

`mcp__Claude_Preview__preview_screenshot`
Expected: first print card shows a single photo (the first slide, `mounted.jpg`) cropped into the card's photo frame — no visible scrollbar, no layout break, other 3 cards unchanged.

- [ ] **Step 3: Commit**

```bash
cd "/Users/charlie/Desktop/Claude Code"
git add styles.css
git commit -m "Style the print card photo carousel (scroll-snap track, arrows, dots)"
```

---

### Task 3: Wire up carousel JS behavior

**Files:**
- Modify: `script.js` (append new block at the end of the file)

**Interfaces:**
- Consumes: `.print-carousel`, `.print-carousel__track`, `.print-carousel__slide`, `.print-carousel__arrow--prev`, `.print-carousel__arrow--next`, `.print-carousel__dots` (from Task 1's markup), `.print-carousel__dot` / `.print-carousel__dot.is-active` (styled by Task 2, created here).
- Produces: nothing consumed by later tasks — this is the last piece of the component.

- [ ] **Step 1: Add the carousel behavior script**

Append to the end of `script.js`:

```javascript
// Print card photo carousels (Instagram-style swipe/dots/arrows)
const prefersReducedMotionForCarousel = window.matchMedia(
  "(prefers-reduced-motion: reduce)"
).matches;

document.querySelectorAll(".print-carousel").forEach((carousel) => {
  const track = carousel.querySelector(".print-carousel__track");
  const slides = Array.from(track.querySelectorAll(".print-carousel__slide"));
  const prevBtn = carousel.querySelector(".print-carousel__arrow--prev");
  const nextBtn = carousel.querySelector(".print-carousel__arrow--next");
  const dotsContainer = carousel.querySelector(".print-carousel__dots");

  if (slides.length < 2) return;

  track.tabIndex = 0;

  function scrollBehavior() {
    return prefersReducedMotionForCarousel ? "auto" : "smooth";
  }

  function currentIndex() {
    return Math.round(track.scrollLeft / track.clientWidth);
  }

  function setActiveDot(index) {
    dots.forEach((dot, i) => dot.classList.toggle("is-active", i === index));
  }

  function goToSlide(index) {
    track.scrollTo({ left: track.clientWidth * index, behavior: scrollBehavior() });
  }

  const dots = slides.map((_, index) => {
    const dot = document.createElement("button");
    dot.className = "print-carousel__dot";
    dot.setAttribute("aria-label", `Photo ${index + 1} of ${slides.length}`);
    dot.addEventListener("click", () => goToSlide(index));
    dotsContainer.appendChild(dot);
    return dot;
  });
  dots[0].classList.add("is-active");

  prevBtn.addEventListener("click", () => goToSlide(Math.max(0, currentIndex() - 1)));
  nextBtn.addEventListener("click", () =>
    goToSlide(Math.min(slides.length - 1, currentIndex() + 1))
  );

  track.addEventListener("keydown", (e) => {
    if (e.key === "ArrowLeft") goToSlide(Math.max(0, currentIndex() - 1));
    if (e.key === "ArrowRight") goToSlide(Math.min(slides.length - 1, currentIndex() + 1));
  });

  let scrollRaf = null;
  track.addEventListener(
    "scroll",
    () => {
      if (scrollRaf) return;
      scrollRaf = requestAnimationFrame(() => {
        setActiveDot(currentIndex());
        scrollRaf = null;
      });
    },
    { passive: true }
  );
});
```

- [ ] **Step 2: Verify dot row is generated**

Reload the preview. Run `mcp__Claude_Preview__preview_snapshot`.
Expected: the first print card's carousel now has 4 dot buttons, each with an accessible name like "Photo 1 of 4" through "Photo 4 of 4", first one active.

- [ ] **Step 3: Verify arrow navigation**

Run `mcp__Claude_Preview__preview_click` on selector `.print-card:nth-child(1) .print-carousel__arrow--next`, then `mcp__Claude_Preview__preview_eval` with expression `document.querySelector('.print-card:nth-child(1) .print-carousel__track').scrollLeft > 0`.
Expected: `true` (track scrolled right).

Run `mcp__Claude_Preview__preview_eval` with expression:
```javascript
Array.from(document.querySelectorAll('.print-card:nth-child(1) .print-carousel__dot')).map(d => d.classList.contains('is-active'))
```
Expected: `[false, true, false, false]` (second dot now active).

- [ ] **Step 4: Verify dot-click navigation**

Run `mcp__Claude_Preview__preview_click` on selector `.print-card:nth-child(1) .print-carousel__dot:nth-child(4)`.
Run `mcp__Claude_Preview__preview_eval` with expression:
```javascript
document.querySelector('.print-card:nth-child(1) .print-carousel__track').scrollLeft ===
  document.querySelector('.print-card:nth-child(1) .print-carousel__track').clientWidth * 3
```
Expected: `true` (jumped straight to the 4th slide).

- [ ] **Step 5: Commit**

```bash
cd "/Users/charlie/Desktop/Claude Code"
git add script.js
git commit -m "Add carousel dot/arrow/keyboard navigation JS for print cards"
```

---

### Task 4: Cross-cutting verification

**Files:** none (verification only — no code changes expected unless a check below fails, in which case fix the relevant file from Tasks 1-3 and re-run that task's verification steps before continuing).

- [ ] **Step 1: Verify the other 3 print cards are unaffected**

Run `mcp__Claude_Preview__preview_snapshot`.
Expected: print cards 2, 3, and 4 still show the "add photo" placeholder text exactly as before — no `.print-carousel` markup, no dots, no arrows on those cards.

- [ ] **Step 2: Verify keyboard navigation**

Run `mcp__Claude_Preview__preview_eval` with expression:
```javascript
(() => {
  const track = document.querySelector('.print-card:nth-child(1) .print-carousel__track');
  track.focus();
  track.dispatchEvent(new KeyboardEvent('keydown', { key: 'ArrowLeft', bubbles: true }));
  return track.scrollLeft;
})()
```
Expected: a lower `scrollLeft` value than before the keypress (moved back one slide from wherever Task 3's steps left it).

- [ ] **Step 3: Verify mobile viewport layout**

Run `mcp__Claude_Preview__preview_resize` with `preset: "mobile"`.
Run `mcp__Claude_Preview__preview_screenshot`.
Expected: first print card's photo carousel still fits inside the card frame at mobile width, no horizontal page overflow, dots visible and legible.

- [ ] **Step 4: Verify contrast against the dark Prints section band**

Run `mcp__Claude_Preview__preview_inspect` on selector `.print-carousel__dot.is-active` with styles `["background-color"]`.
Expected: the accent orange color (matches `--accent: #ff9257` as re-scoped under `.section--dark`), clearly visible against both the photo and the section's near-black background.

- [ ] **Step 5: Reset viewport and take final screenshot**

Run `mcp__Claude_Preview__preview_resize` with `preset: "desktop"`.
Run `mcp__Claude_Preview__preview_screenshot`.
Expected: Prints section shows the key-box carousel card alongside the 3 unchanged placeholder cards, matching the site's existing grey/dark visual language.

No commit needed for this task (verification only, no file changes expected).
