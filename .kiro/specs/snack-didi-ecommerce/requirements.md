# Requirements Document

## Introduction

Snack Didi is a local spicy snack brand owned and operated as a family business. This e-commerce website serves as the primary online storefront, allowing customers to browse products, configure their order preferences, and purchase directly through a dynamic shopping cart. The site also supports the father's consignment distribution model by providing a dedicated section for store owners who want to stock Snack Didi products. Local deliveries are handled personally by the father. The experience is enriched with interactive sound effects, smooth animations, and a vibrant, appetizing visual design built with HTML5, Tailwind CSS, and Vanilla JavaScript.

---

## Glossary

- **Website**: The Snack Didi e-commerce single-page or multi-page HTML5 website.
- **Customer**: An end-user who visits the website to browse and purchase products.
- **Store_Owner**: A business owner who visits the website seeking to stock Snack Didi products through a consignment arrangement.
- **Cart**: The client-side dynamic shopping cart managed via Vanilla JavaScript.
- **Cart_Item**: A single product entry in the Cart, containing product identity, spice level, quantity, and unit price.
- **Product_Catalog**: The collection of available products: Basreng (fried meatball chips), Kerupuk Seblak (spicy cracker chips), and Makaroni Pedas (spicy macaroni).
- **Spice_Level**: A selectable heat preference for a product (e.g., Original, Pedas, Ekstra Pedas).
- **Sound_Engine**: The Vanilla JavaScript module responsible for loading and playing audio feedback.
- **Animation_Engine**: The Vanilla JavaScript module responsible for scroll-triggered and interaction-triggered UI animations.
- **Consignment_Form**: The quick contact form targeting Store_Owners who want to partner with Snack Didi.
- **Order_Summary**: A structured display of all Cart_Items, quantities, individual prices, and the total price.
- **Delivery_Notice**: A UI element that communicates the personal local delivery service provided by the father.

---

## Requirements

### Requirement 1: Product Catalog Display

**User Story:** As a Customer, I want to browse all available Snack Didi products with clear descriptions and prices, so that I can make an informed purchase decision.

#### Acceptance Criteria

1. THE Website SHALL display all three products from the Product_Catalog: Basreng, Kerupuk Seblak, and Makaroni Pedas.
2. THE Website SHALL display a product name, a description of no more than 150 characters, a product image with a non-empty alt attribute, and a price formatted in IDR using period-separated thousands with no decimal places (e.g., Rp 15.000) for each product in the Product_Catalog.
3. WHEN a product image fails to load, THE Website SHALL display a visible placeholder element that includes the product name as accessible alt text.
4. THE Website SHALL render the Product_Catalog such that no horizontal scrollbar appears and all product information (name, description, price) remains visible at any viewport width between 320px and 1920px inclusive.

---

### Requirement 2: Spice Level Selection

**User Story:** As a Customer, I want to choose a spice level for each product, so that I can customize my order to my heat preference.

#### Acceptance Criteria

1. THE Website SHALL present exactly three Spice_Level options (Original, Pedas, Ekstra Pedas) for each product in the Product_Catalog, with no option pre-selected by default.
2. WHEN a Customer selects a Spice_Level, THE Website SHALL apply a distinct visual treatment to the selected option (such as a filled background or a highlighted border) that is visually differentiated from the unselected options.
3. WHEN a Customer selects a Spice_Level, THE Sound_Engine SHALL play an audio feedback clip with a duration no longer than 2000ms, starting within 100ms of the selection event.
4. IF a Customer attempts to add a product to the Cart without selecting a Spice_Level, THEN THE Website SHALL display an inline validation message adjacent to the Spice_Level options prompting the Customer to select a Spice_Level, and SHALL NOT add the product to the Cart.

---

### Requirement 3: Dynamic Shopping Cart

**User Story:** As a Customer, I want to manage a shopping cart that updates in real time, so that I can review and adjust my order before finalizing it.

#### Acceptance Criteria

1. WHEN a Customer adds a product to the Cart and no Cart_Item with the same product identity and Spice_Level exists, THE Cart SHALL create a new Cart_Item containing the product identity, selected Spice_Level, quantity of 1, and unit price.
2. WHEN a Customer adds a product to the Cart and a Cart_Item with the same product identity and Spice_Level already exists, THE Cart SHALL increment that Cart_Item's quantity by 1 rather than creating a duplicate Cart_Item.
3. WHEN a Customer adds a product to the Cart, THE Sound_Engine SHALL play an "add to cart" audio feedback clip within 100ms of the add event.
4. WHEN a Customer increases the quantity of a Cart_Item, THE Cart SHALL increment the Cart_Item quantity by 1 and recalculate the Order_Summary within 300ms.
5. WHEN a Customer decreases the quantity of a Cart_Item and the current quantity is greater than 1, THE Cart SHALL decrement the Cart_Item quantity by 1 and recalculate the Order_Summary within 300ms.
6. WHEN a Customer decreases the quantity of a Cart_Item and the current quantity is exactly 1, THE Cart SHALL NOT decrement the quantity below 1; instead the Customer SHALL use the explicit remove action to delete the Cart_Item.
7. WHEN a Customer removes a Cart_Item, THE Cart SHALL remove the item and recalculate the Order_Summary within 300ms.
8. THE Cart SHALL display an Order_Summary that shows each Cart_Item's name, Spice_Level, quantity, line total (quantity × unit price), and a cart total (sum of all line totals), with no taxes or delivery fees included.
9. WHEN the Cart contains zero Cart_Items, THE Website SHALL display an empty-cart state message to the Customer.
10. THE Cart SHALL persist Cart_Items in the browser's localStorage and restore the Cart state when the Customer reloads the page, retaining the persisted state until the Customer clears their browser storage.

---

### Requirement 4: Direct Purchase & Order Submission

**User Story:** As a Customer, I want to submit my order directly through the website, so that I can complete my purchase without needing a third-party platform.

#### Acceptance Criteria

1. THE Website SHALL provide a checkout action that a Customer can trigger to submit the contents of the Cart as an order.
2. IF a Customer triggers the checkout action when the Cart contains zero Cart_Items, THEN THE Website SHALL display an error message informing the Customer that the Cart is empty and SHALL NOT proceed to the checkout form.
3. WHEN a Customer triggers the checkout action and the Cart is non-empty, THE Website SHALL display a checkout form that collects the Customer's name (maximum 100 characters), phone number (maximum 20 characters, digits with optional spaces, +, -, (, ) characters only), and delivery address (maximum 300 characters).
4. IF a Customer submits the checkout form with one or more required fields empty, THEN THE Website SHALL display a field-level validation message adjacent to each empty field identifying what is required.
5. IF a Customer submits the checkout form with a phone number that does not conform to the allowed character pattern, THEN THE Website SHALL display a field-level validation message on the phone number field.
6. WHEN a Customer successfully submits the checkout form, THE Website SHALL display an on-screen confirmation message that includes the Customer's name, a list of ordered items with their quantities, the order total, and a statement that the father will personally handle local delivery.
7. WHEN a Customer successfully submits the checkout form, THE Sound_Engine SHALL play an "order confirmed" audio feedback clip that is audibly distinct from all other audio clips used in the Website, within 100ms of form submission.
8. WHEN a Customer successfully submits the checkout form, THE Cart SHALL remove all Cart_Items so that the Cart contains zero items and the cart total displays as Rp 0.

---

### Requirement 5: Personal Delivery Notice

**User Story:** As a Customer, I want to know that my order will be delivered personally, so that I feel confident about the local delivery experience.

#### Acceptance Criteria

1. THE Website SHALL display a Delivery_Notice that explicitly states that local deliveries are carried out personally by the father.
2. THE Website SHALL render the Delivery_Notice using warm, first-person or family-oriented language (e.g., referencing "Bapak" or "Ayah" delivering the order) consistent with the family-business brand identity of Snack Didi.
3. THE Website SHALL display the Delivery_Notice within the main content flow such that it is fully visible without horizontal scrolling at any viewport width between 320px and 1920px inclusive.
4. THE Delivery_Notice SHALL be distinct from the checkout confirmation message and visible on the page before a Customer initiates checkout.

---

### Requirement 6: Consignment Partner Section

**User Story:** As a Store_Owner, I want to find clear information and a quick way to contact Snack Didi about stocking their products, so that I can start a consignment partnership efficiently.

#### Acceptance Criteria

1. THE Website SHALL include a dedicated Consignment Partner section that contains at least one paragraph of explanatory text describing the consignment arrangement available to Store_Owners.
2. THE Website SHALL display a visually prominent Call-To-Action element (button or styled link) within the Consignment Partner section that, when activated, scrolls to or reveals the Consignment_Form.
3. THE Website SHALL include a Consignment_Form that collects the Store_Owner's name (required, maximum 100 characters), store name (required, maximum 150 characters), phone number (required, maximum 20 characters, digits with optional spaces, +, -, (, ) characters), and city (required, maximum 100 characters).
4. IF a Store_Owner submits the Consignment_Form with one or more required fields empty, THEN THE Website SHALL display a field-level validation message adjacent to each empty field identifying what is required, and SHALL NOT submit the form.
5. IF a Store_Owner submits the Consignment_Form with a phone number that does not conform to the allowed character pattern, THEN THE Website SHALL display a field-level validation message on the phone number field and SHALL NOT submit the form.
6. WHEN a Store_Owner successfully submits the Consignment_Form, THE Website SHALL display an on-screen confirmation message acknowledging the submission and stating that the father will follow up personally.
7. WHEN a Store_Owner successfully submits the Consignment_Form, THE Sound_Engine SHALL play a "form submitted" audio feedback clip within 100ms of submission.

---

### Requirement 7: Interactive Sound Effects

**User Story:** As a Customer, I want the website to respond to my actions with fun audio feedback, so that the experience feels engaging and lively.

#### Acceptance Criteria

1. THE Sound_Engine SHALL load all audio assets asynchronously using lazy or deferred loading so that audio asset requests do not block the DOMContentLoaded or load events of the page.
2. THE Sound_Engine SHALL play audio feedback for each of the following trigger events: general button click, Spice_Level selection, Cart add action, order confirmation, and Consignment_Form submission.
3. WHEN a user interaction triggers audio feedback, THE Sound_Engine SHALL begin playback of the corresponding clip within 100ms of the trigger event.
4. THE Website SHALL display a persistent mute/unmute toggle control in a fixed or always-visible location that a Customer can activate to disable or re-enable all Sound_Engine audio output.
5. WHEN the mute toggle is set to muted, THE Sound_Engine SHALL not begin any new audio playback for any trigger event until the toggle is set to unmuted.
6. THE Sound_Engine SHALL defer creation of the AudioContext (or equivalent) until after the first user interaction with the page, in compliance with browser autoplay policies.
7. WHEN an audio clip is currently playing and a new trigger event fires for the same clip, THE Sound_Engine SHALL restart the clip from the beginning rather than queuing a second concurrent instance.

---

### Requirement 8: UI Animations and Micro-interactions

**User Story:** As a Customer, I want the website to feel dynamic and polished, so that browsing feels enjoyable rather than static.

#### Acceptance Criteria

1. THE Animation_Engine SHALL apply a fade-in combined with an upward translate (minimum 16px) entrance animation to each content section element as it enters the visible viewport during scroll, using the IntersectionObserver API.
2. WHEN a Customer hovers over a product card, THE Animation_Engine SHALL begin a scale-up or upward-lift CSS transition that completes within 300ms and returns to the original state within 300ms when hover ends.
3. WHEN a Customer hovers over a primary call-to-action button, THE Animation_Engine SHALL begin a background-color or box-shadow CSS transition that completes within 200ms.
4. WHEN a Customer adds a product to the Cart, THE Animation_Engine SHALL apply a bounce or pulse CSS animation to the Cart icon that starts within 300ms of the add event and lasts no longer than 600ms.
5. THE Animation_Engine SHALL implement all scroll-triggered entrance animations using the IntersectionObserver API and SHALL NOT attach scroll event listeners on the window object for animation purposes.
6. WHERE the CSS media query `prefers-reduced-motion: reduce` evaluates to true, THE Animation_Engine SHALL set animation duration and transition duration to 0ms for all non-essential decorative animations and transitions.

---

### Requirement 9: Visual Design and Branding

**User Story:** As a Customer, I want the website to look vibrant, appetizing, and modern, so that it reflects the energy and quality of the Snack Didi brand.

#### Acceptance Criteria

1. THE Website SHALL apply a consistent color palette in which the dominant colors are from the warm spectrum (reds, oranges, and yellows), with the same palette token values used across all pages and sections.
2. THE Website SHALL apply at least two distinct font-weight values to typographic elements, with a heavier weight used exclusively for headings and a lighter or regular weight used for body text.
3. THE Website SHALL display a branded header that is visible at the top of every page and contains the Snack Didi logo or text wordmark and at least one navigation link to each primary section of the site.
4. THE Website SHALL render all interactive elements (buttons, links, form inputs, spice level selectors) with a minimum clickable/tappable area of 44×44 CSS pixels.
5. THE Website SHALL achieve a contrast ratio of at least 4.5:1 between the computed foreground text color and background color for all body text elements, as defined by WCAG 2.1 Success Criterion 1.4.3.

---

### Requirement 10: Responsive Layout

**User Story:** As a Customer, I want the website to work well on my phone, tablet, and desktop, so that I can shop comfortably from any device.

#### Acceptance Criteria

1. THE Website SHALL use Tailwind CSS responsive utility classes to implement layout changes at the following breakpoints: mobile (viewport width ≥320px), tablet (viewport width ≥768px), and desktop (viewport width ≥1024px).
2. THE Website SHALL render the Cart as a slide-in overlay drawer anchored to the right edge on viewport widths below 768px, and as an inline sidebar or fixed panel on viewport widths of 768px and above.
3. THE Website SHALL display the Product_Catalog as a one-column grid on viewport widths below 640px, a two-column grid on viewport widths between 640px and 1023px inclusive, and a three-column grid on viewport widths of 1024px and above.
4. THE Website SHALL display the primary navigation as a hamburger-triggered collapsible menu on viewport widths below 768px, and as a fully visible horizontal navigation bar on viewport widths of 768px and above.
