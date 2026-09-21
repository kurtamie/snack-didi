# Implementation Plan: Snack Didi E-Commerce Website

## Overview

Incremental implementation of a single-page HTML5 storefront using Tailwind CSS (CDN) and Vanilla JavaScript ES Modules. Test infrastructure is established early so tests can be run after each module. Property-based and unit tests are co-located with the modules they test. Every step builds on the previous so there is no orphaned code.

---

## Tasks

- [ ] 1. Project scaffolding and directory structure
  - [ ] 1.1 Create `index.html` with Tailwind Play CDN script tag, custom Tailwind config block (warm color palette, two font weights), `<meta viewport>`, and all top-level structural sections: `#header`, `#catalog`, `#delivery-notice`, `#consignment`, `#cart-drawer`, `#checkout-modal`, and the `<script type="module" src="src/main.js">` bootstrap entry point
    - Sections should be empty shells at this stage — content comes in later tasks
    - Apply warm-spectrum base colors (reds, oranges, yellows) in the Tailwind config
    - _Requirements: 9.1, 9.2, 9.3, 10.1_
  - [ ] 1.2 Create the source directory layout: `src/`, `src/data/`, `src/modules/`, `tests/`, and `assets/audio/` placeholder directories; add a `.gitkeep` in `assets/audio/` so the directory is tracked

- [ ] 2. Test infrastructure
  - [ ] 2.1 Create `package.json` with Vitest and fast-check as dev dependencies; create `vitest.config.js` with `jsdom` environment pointing to the `tests/` directory; verify `npx vitest --run` exits cleanly with zero test files as baseline
    - This must be completed before any module test sub-tasks so the test runner is available throughout implementation

- [ ] 3. Static product data
  - [ ] 3.1 Create `src/data/products.js` exporting a `PRODUCTS` array with the three products (Basreng, Kerupuk Seblak, Makaroni Pedas), each with `id`, `name`, `description` (≤150 chars), `price` (IDR integer), `imageSrc`, and `imageAlt` fields matching the `Product` interface in the design
    - _Requirements: 1.1, 1.2_

- [ ] 4. Utility functions
  - [ ] 4.1 Create `src/utils.js` and implement `formatIDR(price)` that returns a string in the form `"Rp X"` with period-separated thousands groups and no decimal places (e.g., `Rp 15.000`, `Rp 1.250.000`)
    - _Requirements: 1.2_
  - [ ]* 4.2 Write property test for `formatIDR` in `tests/utils.property.test.js`
    - **Property 1: IDR Price Formatting**
    - **Validates: Requirements 1.2**
    - Use `fc.integer({ min: 1 })` to generate positive integers; assert the return value matches `/^Rp \d{1,3}(\.\d{3})*$/`
    - _Requirements: 1.2_

- [ ] 5. EventBus
  - [ ] 5.1 Create `src/modules/EventBus.js` that exports thin wrappers around `document.dispatchEvent` and `document.addEventListener` for the namespaced `CustomEvent` contract defined in the design (`snd:*`, `cart:*`, `anim:*`)
    - _Requirements: 3.3, 7.2, 7.3, 8.4_

- [ ] 6. CartManager module
  - [ ] 6.1 Create `src/modules/CartManager.js` implementing the `CartManager` class with `addItem`, `increaseQuantity`, `decreaseQuantity`, `removeItem`, `clearCart`, `getState`, and `getOrderSummary` methods, plus static `fromItems` and `loadFromStorage` factory methods as shown in the design
    - Persist state to `localStorage` key `snackdidi_cart` after every mutation
    - Include schema `version` field; reset to empty cart on version mismatch or malformed JSON
    - After each mutation dispatch `cart:changed` and (on add) `snd:cart-add` via EventBus
    - _Requirements: 3.1, 3.2, 3.4, 3.5, 3.6, 3.7, 3.8, 3.10_
  - [ ]* 6.2 Write property test for Property 2 (Cart Add — New Item Creation) in `tests/cartmanager.property.test.js`
    - **Property 2: Cart Add — New Item Creation**
    - **Validates: Requirements 3.1**
    - For any `(productId, spiceLevel)` not in cart, `addItem` creates exactly one new item with `quantity === 1`
    - _Requirements: 3.1_
  - [ ]* 6.3 Write property test for Property 3 (Cart Add — Deduplication) in `tests/cartmanager.property.test.js`
    - **Property 3: Cart Add — Deduplication**
    - **Validates: Requirements 3.2**
    - For any cart with an existing `(productId, spiceLevel)`, calling `addItem` with the same key increments quantity by 1 and does not add a new item
    - _Requirements: 3.2_
  - [ ]* 6.4 Write property test for Property 4 (Quantity Lower Bound Invariant) in `tests/cartmanager.property.test.js`
    - **Property 4: Quantity Lower Bound Invariant**
    - **Validates: Requirements 3.6**
    - For any item with `quantity === 1`, `decreaseQuantity` leaves it at 1
    - _Requirements: 3.6_
  - [ ]* 6.5 Write property test for Property 5 (Order Summary Arithmetic) in `tests/cartmanager.property.test.js`
    - **Property 5: Order Summary Arithmetic**
    - **Validates: Requirements 3.8**
    - For any cart state, each `lineTotal === quantity * unitPrice` and `cartTotal === sum(lineTotals)`; no taxes added
    - _Requirements: 3.8_
  - [ ]* 6.6 Write property test for Property 6 (Cart Persistence Round-Trip) in `tests/cartmanager.property.test.js`
    - **Property 6: Cart Persistence Round-Trip**
    - **Validates: Requirements 3.10**
    - Serialize any cart state to a mock storage, deserialize it, and assert structural equivalence (same items, quantities, spice levels, prices)
    - _Requirements: 3.10_

- [ ] 7. Checkpoint — CartManager
  - Run `npx vitest --run` and confirm all CartManager property tests pass.

- [ ] 8. Audio asset placeholders and SoundEngine module
  - [ ] 8.1 Create 5 silent placeholder audio files in `assets/audio/`: `btn-click.mp3`, `spice-select.mp3`, `cart-add.mp3`, `order-confirm.mp3`, `consignment-submit.mp3` — these prevent `fetch()` 404s during development; replace with real audio files before shipping
    - _Requirements: 7.1, 7.2_
  - [ ] 8.2 Create `src/modules/SoundEngine.js` implementing the `SoundEngine` class with `init()`, `play(clipName)`, `setMuted(muted)`, and `isMuted()` methods
    - Defer `AudioContext` creation to first `pointerdown` or `keydown` user interaction to comply with browser autoplay policies
    - Implement a `GainNode` between all `BufferSourceNode`s and `destination` for mute control (gain 0 = muted, 1 = unmuted)
    - Lazy-load each clip via `fetch()` + `decodeAudioData()` on first `play()` call for that clip (not on page load)
    - Implement clip restart: if a `BufferSourceNode` for a clip is active, `.stop()` it before creating a fresh `BufferSourceNode`
    - Persist mute state to `localStorage` key `snackdidi_mute`; restore on `init()`
    - Subscribe to all `snd:*` EventBus events and call `play()` with the corresponding `ClipName`
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7_
  - [ ] 8.3 Add suspended `AudioContext` recovery to `SoundEngine.play()`: at the start of each `play()` call, check `AudioContext.state`; if `'suspended'`, call `context.resume()` and await the Promise before creating the `BufferSourceNode`; if `resume()` rejects, swallow the error silently so that sound failure never propagates
    - _Requirements: 7.3, 7.6_
  - [ ]* 8.4 Write unit tests for `SoundEngine` in `tests/soundengine.unit.test.js`
    - Requires task 2.1 (test infrastructure) and task 8.2 (SoundEngine class)
    - Test that `AudioContext` is not created before first user interaction
    - Test that `play()` while muted does not start any `BufferSourceNode`
    - Test clip restart: calling `play()` on an in-progress clip stops the previous node and starts fresh
    - Test that mute state round-trips through `localStorage`
    - _Requirements: 7.5, 7.6, 7.7_
  - [ ]* 8.5 Write property test for Property 12 (Mute Suppresses All Audio) in `tests/soundengine.property.test.js`
    - Requires task 2.1 (test infrastructure) and task 8.2 (SoundEngine class)
    - **Property 12: Mute Suppresses All Audio**
    - **Validates: Requirements 7.5**
    - For any `ClipName` dispatched while `isMuted() === true`, assert no `BufferSourceNode.start()` is invoked
    - _Requirements: 7.5_

- [ ] 9. AnimationEngine module
  - [ ] 9.1 Create `src/modules/AnimationEngine.js` implementing the `AnimationEngine` class with an `init()` method
    - Read `window.matchMedia('(prefers-reduced-motion: reduce)')` once at init; store as `reducedMotion`
    - Set up a single `IntersectionObserver` (`threshold: 0.15`) watching all `[data-animate]` elements; on intersection add class `animate-in` (opacity 1, translateY 0); initial state set inline at DOMContentLoaded
    - When `reducedMotion` is true, set all animation/transition durations to 0ms; never attach `window.scroll` listeners
    - Subscribe to `anim:cart-badge-pulse` and add a CSS animation class to the cart icon, auto-removing it after 600ms
    - _Requirements: 8.1, 8.5, 8.6_
  - [ ] 9.2 Attach hover animation listeners in `AnimationEngine.init()` for product cards (scale-up/lift transition ≤300ms) and primary CTA buttons (background-color/box-shadow transition ≤200ms)
    - _Requirements: 8.2, 8.3_
  - [ ]* 9.3 Write unit tests for `AnimationEngine` in `tests/animationengine.unit.test.js`
    - Test that `[data-animate]` elements receive `animate-in` class on IntersectionObserver callback
    - Test that `anim:cart-badge-pulse` adds pulse class and auto-removes after 600ms
    - Test that when `prefers-reduced-motion` is true, animation durations are 0ms
    - _Requirements: 8.1, 8.4, 8.6_

- [ ] 10. Product Catalog UI
  - [ ] 10.1 Create `src/modules/CatalogUI.js` that renders product cards into `#catalog` from the `PRODUCTS` array
    - Each card: product image (`<img>` with non-empty `alt`), name, description, price formatted with `formatIDR`, three spice-level `<button>` elements with `aria-pressed` toggling, and a "Tambah ke Keranjang" add-to-cart button
    - Apply `data-animate` attribute to each card for scroll entrance animation
    - Minimum 44×44 CSS pixels for all interactive elements (spice buttons, add button)
    - _Requirements: 1.1, 1.2, 2.1, 2.2, 9.4_
  - [ ] 10.2 Implement the `onerror` image fallback in `CatalogUI.js`: replace the failed `<img>` with a styled `<div role="img" aria-label="{productName}">` placeholder with warm-colored background and visible product name text
    - _Requirements: 1.3_
  - [ ] 10.3 Wire spice level selection in `CatalogUI.js`: selecting a level applies visual treatment to the active button, clears it from siblings, and dispatches `snd:spice-select` via EventBus; set `data-snd-handled` on the spice buttons
    - Guard the add-to-cart button: if clicked with no spice level selected, inject `<p role="alert">` adjacent to the spice selector and do not call `CartManager.addItem`
    - On valid add-to-cart click: call `CartManager.addItem`, dispatch `anim:cart-badge-pulse`, and set `data-snd-handled` on the add button
    - _Requirements: 2.1, 2.2, 2.4, 3.1, 3.3, 8.4_

- [ ] 11. Cart UI
  - [ ] 11.1 Create `src/modules/CartRenderer.js` that listens to `cart:changed` and rebuilds the cart UI inside `#cart-drawer`
    - Display each `CartItem`: name, spice level, quantity, line total formatted with `formatIDR`, a "+" increase button, a "−" decrease button (disabled at quantity 1), and a remove "×" button
    - Display the `OrderSummary` cart total formatted with `formatIDR`
    - When `items.length === 0`, render the empty-cart state message
    - All buttons minimum 44×44 CSS pixels
    - _Requirements: 3.4, 3.5, 3.6, 3.7, 3.8, 3.9, 9.4_
  - [ ] 11.2 Implement the cart drawer toggle in `index.html` and `CartRenderer.js`
    - On mobile (<768px): drawer slides in from the right (`translate-x-full` → `translate-x-0`) via a toggle button in the header
    - On tablet/desktop (≥768px): cart is always visible as an inline sidebar or fixed panel (no toggle needed)
    - _Requirements: 10.2_

- [ ] 12. Checkpoint — Cart interaction
  - Open the site in a browser, add a product, verify the cart drawer updates and the badge pulses; confirm `localStorage` key `snackdidi_cart` is populated.

- [ ] 13. CheckoutForm module
  - [ ] 13.1 Create `src/modules/CheckoutForm.js` implementing `open(cartSummary)` and `close()` methods
    - `open()`: if `items.length === 0`, show inline error near checkout button and return without opening the overlay
    - Render the checkout overlay (`#checkout-modal`) with name (max 100), phone (max 20, pattern `/^[0-9 +\-()\s]{1,20}$/`), and address (max 300) fields; set `data-snd-handled` on the submit button
    - Validate on submit; inject `<p role="alert">` error messages adjacent to each failing field
    - On success: render confirmation into `#order-confirmation` (includes customer name, item list with quantities, total, personal delivery statement), dispatch `snd:order-confirm`, call `CartManager.clearCart()`, close overlay
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8_
  - [ ]* 13.2 Write property test for Property 7 (Checkout Form Validation — Required Field Coverage) in `tests/checkoutform.property.test.js`
    - **Property 7: Checkout Form Validation — Required Field Coverage**
    - **Validates: Requirements 4.4**
    - For any non-empty subset of the three required fields left blank, the form shows a per-field validation message and does not proceed to confirmation
    - _Requirements: 4.4_
  - [ ]* 13.3 Write property test for Property 8 (Phone Number Pattern Validation — checkout) in `tests/validation.property.test.js`
    - **Property 8: Phone Number Pattern Validation**
    - **Validates: Requirements 4.5, 6.5**
    - For any string not matching `/^[0-9 +\-()\s]{1,20}$/`, submitting either form shows a phone-field error and does not proceed
    - _Requirements: 4.5, 6.5_
  - [ ]* 13.4 Write property test for Property 9 (Checkout Confirmation Content Completeness) in `tests/checkoutform.property.test.js`
    - **Property 9: Checkout Confirmation Content Completeness**
    - **Validates: Requirements 4.6**
    - For any valid submission (name, phone, address, non-empty cart), the confirmation HTML contains the customer's name, each item with its quantity, the cart total, and the personal delivery statement
    - _Requirements: 4.6_
  - [ ]* 13.5 Write property test for Property 10 (Cart Cleared After Checkout) in `tests/checkoutform.property.test.js`
    - **Property 10: Cart Cleared After Checkout**
    - **Validates: Requirements 4.8**
    - For any non-empty cart, a successful checkout submission leaves `CartManager.getState().items.length === 0` and `cartTotal === 0`
    - _Requirements: 4.8_

- [ ] 14. ConsignmentForm module
  - [ ] 14.1 Create `src/modules/ConsignmentForm.js` managing the consignment section form in `#consignment`
    - Fields: owner name (required, max 100), store name (required, max 150), phone (required, max 20, same pattern as checkout), city (required, max 100); set `data-snd-handled` on the submit button
    - Validate on submit; inject `<p role="alert">` error messages adjacent to each failing field; do not submit on error
    - On success: show inline confirmation message (form is NOT cleared), dispatch `snd:consignment-submit`
    - _Requirements: 6.3, 6.4, 6.5, 6.6, 6.7_
  - [ ]* 14.2 Write property test for Property 11 (Consignment Form Validation — Required Field Coverage) in `tests/consignmentform.property.test.js`
    - **Property 11: Consignment Form Validation — Required Field Coverage**
    - **Validates: Requirements 6.4**
    - For any non-empty subset of the four required fields left blank, the form shows a per-field validation message and does not submit
    - _Requirements: 6.4_

- [ ] 15. Checkpoint — Form submissions
  - Open the site in a browser; submit the checkout form with a blank field and confirm a per-field error appears; submit a valid checkout and confirm the cart clears and confirmation shows the customer's name and order total.

- [ ] 16. Responsive layout polish
  - [ ] 16.1 Implement responsive product grid in `#catalog` using Tailwind responsive prefixes: `grid-cols-1` below 640px, `sm:grid-cols-2` at 640px–1023px, `lg:grid-cols-3` at ≥1024px
    - _Requirements: 10.3_
  - [ ] 16.2 Implement hamburger nav in `#header`: show toggle button below 768px, hide it at `md:` breakpoint; clicking the toggle shows/hides the nav link list; full horizontal nav always visible at `md:` and above
    - _Requirements: 10.4_

- [ ] 17. Visual design, branding, and Delivery Notice
  - [ ] 17.1 Apply branded header in `#header`: Snack Didi logo/wordmark, navigation links to each primary section, and the persistent mute/unmute toggle button — all in the warm color palette, visible at all viewport widths 320px–1920px
    - _Requirements: 9.3, 7.4, 10.1_
  - [ ] 17.2 Build the `#delivery-notice` section with warm, family-oriented language referencing "Bapak" or "Ayah" delivering the order personally; apply `data-animate` for scroll entrance; ensure it is fully visible 320px–1920px and distinct from the checkout confirmation message
    - _Requirements: 5.1, 5.2, 5.3, 5.4_
  - [ ] 17.3 Build the `#consignment` section with explanatory paragraph text, a visually prominent CTA button/link that scrolls to or reveals `ConsignmentForm`, and the `ConsignmentForm` fields wired to `ConsignmentForm.js`; apply `data-animate` to the section
    - _Requirements: 6.1, 6.2_
  - [ ] 17.4 Audit all interactive elements (buttons, links, form inputs, spice selectors) to confirm minimum 44×44 CSS pixel tap targets and check body text contrast ratio ≥4.5:1 against backgrounds; fix any failures with Tailwind utility adjustments
    - _Requirements: 9.4, 9.5_

- [ ] 18. Main entry point wiring
  - [ ] 18.1 Create `src/main.js` that imports and initialises all modules in order: `EventBus`, `CartManager`, `SoundEngine.init()`, `AnimationEngine.init()`, `CatalogUI`, `CartRenderer`, `CheckoutForm`, `ConsignmentForm`; attach the checkout button handler that calls `CheckoutForm.open(cartManager.getOrderSummary())`
    - _Requirements: 4.1, 6.2, 7.2, 8.1_
  - [ ] 18.2 Attach a delegated `click` listener on `document` in `src/main.js` that dispatches `snd:btn-click` via EventBus for any `<button>` click not already handled by a module-level `snd:*` event; use `event.target.closest('button')` and skip buttons with the `data-snd-handled` attribute
    - _Requirements: 7.2_

- [ ] 19. DOM unit tests spanning multiple modules
  - [ ]* 19.1 Write unit tests for cross-module DOM behaviors in `tests/dom.unit.test.js`
    - `formatIDR(15000)` → `"Rp 15.000"`
    - Spice level selector: clicking one deselects the other two
    - Cart empty state renders correct message when items array is empty
    - Image `onerror` inserts placeholder `<div>` with correct `role="img"` and `aria-label`
    - Mute toggle in header dispatches correct state change
    - Hamburger menu toggles nav visibility
    - _Requirements: 1.3, 2.1, 3.9, 7.4, 10.4_

- [ ] 20. Checkpoint — Full-page browser smoke-test
  - Open the site at 320px and 1024px viewports; confirm all modules initialise without console errors, the mute toggle silences audio, and no horizontal scrollbar appears at any tested width.

- [ ] 21. Final checkpoint — All tests pass
  - Run `npx vitest --run` and confirm all property-based and unit tests pass. Ask the user if any questions arise.

---

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP; run `npx vitest --run` (not watch mode) for single-pass execution
- Task 2.1 (test infrastructure) is a prerequisite for every test sub-task — ensure `npx vitest --run` exits cleanly before executing any `*`-marked task
- All modules communicate exclusively through EventBus — no direct imports between sibling modules
- The `CartManager.fromItems(items)` static factory (implemented in task 6.1) is required by property tests 6.2–6.6 and 13.5; ensure it is exported before those tasks run
- The `data-snd-handled` attribute must be set consistently in `CatalogUI.js` (task 10.3), `CartRenderer.js` (task 11.1), `CheckoutForm.js` (task 13.1), and `ConsignmentForm.js` (task 14.1) on any button that already dispatches a specific `snd:*` event, so the delegated handler in `main.js` (task 18.2) does not double-fire
- Audio files in `assets/audio/` are created as silent placeholders in task 8.1 and should be replaced with real audio clips before shipping
- Tasks 8.2 and 8.3 are ordered so the full `SoundEngine` class is created first (8.2), then the suspended-`AudioContext` recovery path is added to `play()` (8.3)

---

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "1.2"] },
    { "id": 1, "tasks": ["2.1", "3.1"] },
    { "id": 2, "tasks": ["4.1", "5.1"] },
    { "id": 3, "tasks": ["6.1"] },
    { "id": 4, "tasks": ["4.2", "6.2", "6.3", "6.4", "6.5", "6.6", "8.1"] },
    { "id": 5, "tasks": ["8.2"] },
    { "id": 6, "tasks": ["8.3", "9.1"] },
    { "id": 7, "tasks": ["8.4", "8.5", "9.2", "10.1"] },
    { "id": 8, "tasks": ["9.3", "10.2", "10.3"] },
    { "id": 9, "tasks": ["11.1", "11.2"] },
    { "id": 10, "tasks": ["13.1", "14.1", "16.1", "16.2", "17.1", "17.2", "17.3"] },
    { "id": 11, "tasks": ["13.2", "13.3", "13.4", "13.5", "14.2", "17.4", "18.1"] },
    { "id": 12, "tasks": ["18.2"] },
    { "id": 13, "tasks": ["19.1"] }
  ]
}
```
