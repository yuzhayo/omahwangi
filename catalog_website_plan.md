# Catalog Website: End-to-End Plan

This file has two parts.

* **Part A: Product Requirements (PRD).** What the product is, who uses it, and exactly what it must do. Requirements have ids (FR-1, NFR-1, US-1) so they can be checked.

* **Part B: Technical Plan.** How to build it, in phases. Every phase has: what it needs, exact steps, and success criteria that can be checked yes/no. Do the phases in order. Do not start a phase until the success criteria of the one before it pass.

Labels used below: **(decided)** came from the owner. **(proposed)** was filled in to complete the document and can be changed.

# Part A: Product Requirements (PRD)

## 1. Product summary

A single-page catalog website for a store. Visitors browse items by category on their phone, put items in a cart, and send the order to the store through WhatsApp. There is no online payment: the order is processed in WhatsApp. The store owner adds items and updates stock through a separate private page: item photos are stored in a Cloudflare R2 bucket, and item data (including the photo filename/key) is kept in a Google Sheet.

Mobile first, dark theme, Indonesian language, IDR prices, targeting **Android 9+ (Chrome 88+) and iOS 16+ (Safari 16+)**.

## 2. Users and roles

| Role | Who | What they do | 
| ----- | ----- | ----- | 
| Customer | A visitor on a phone (Android 9+ or iOS 16+). Speaks Indonesian. | Browses, searches, fills the cart, sends the order on WhatsApp. | 
| Store admin | The store owner or staff. | Adds items with photos and updates stock through the input page, edits or removes items in the Google Sheet, receives and processes orders in WhatsApp. | 
| Builder | A developer or AI agent. | Builds and deploys the site by following Part B. | 

## 3. Goals and non-goals

**Goals**

* G1. A customer can find an item and send an order to WhatsApp quickly and without creating an account.

* G2. The site works smoothly on entry-level phones (Android 9+ / iOS 16+) and slow mobile connections.

* G3. The admin can add items or mark items out of stock from a phone in under a minute, with no complicated backend.

* G4. Running cost stays at or near zero (Cloudflare Pages, Cloudflare R2 free tier, Google Sheet, Google Apps Script).

**Proposed success measures (confirm or change)**

* A first-time customer can go from opening the site to the WhatsApp message in under 2 minutes.

* On a repeat visit, items are visible within about 1–2 seconds even on a slow connection (cached app shell + cached items).

* Cached backend reads from Google Apps Script respond in under 300 ms, and photos load from the Cloudflare R2 CDN edge in under 100 ms.

* The admin can add one item with a photo in under 1 minute.

**Non-goals (not in this version)**

* Online payment or a payment gateway.

* Customer accounts, login, or server-side order history.

* Full user management for admins (protected via a secret write token / PIN saved locally on the admin's phone).

* Automatic stock reduction when a customer clicks checkout (since checkout only opens WhatsApp).

* Order tracking or status inside the website.

* Languages other than Indonesian, currencies other than IDR.

## 4. User stories

**Customer**

* US-1. I can browse items grouped by category on my phone.

* US-2. I can search items by name or category.

* US-3. I can see which items are out of stock and cannot accidentally order more than the available stock.

* US-4. I can add items to a cart, change the quantity (up to available stock), and see the total.

* US-5. I can send my order, with my name, address, and optional notes, to the store on WhatsApp.

* US-6. If I come back later, my cart and my delivery name/address are still on my phone.

* US-7. I can jump to a section of the page from a menu without scrolling manually, and use my phone's Back button (or the on-screen close button) to close full-screen menus/cart cleanly without leaving the site.

**Store admin**

* US-8. I can add a new item with a photo from my phone, using a separate private page.

* US-9. I can quickly update an item's stock (or mark it "Habis") from the input page on my phone, or edit/remove items directly in the Google Sheet.

* US-10. I receive orders on WhatsApp in a clear, readable format.

* US-11. I can change store texts and layout through JSON files without editing code.

## 5. Functional requirements

Each requirement says what must be true. The phase that builds it is shown in the traceability table in section 9.

**Navigation and search**

* FR-1. A sticky bar stays at the top while scrolling. It holds the main buttons (Home, Product), a search button, and a cart button that shows the item count. (decided)

* FR-2. A hamburger menu lists these entries: header, category/products (with each category listed), footer (contact, about, etc.). Tapping an entry closes the menu and jumps to that section without the sticky bar covering the title. (decided)

* FR-3. The search bar is hidden by default. The search button shows it. Typing filters items by name and category, ignoring upper and lower case, and hides empty categories. If nothing matches, show a message. (decided, empty message proposed)

**Catalog**

* FR-4. Items are read from the Google Sheet through the Apps Script web app using a public read token. (decided)

* FR-5. Items are shown in sections, one per category. (decided)

* FR-6. Each item card is part of an e-commerce style grid and shows image, name, price, and a short two-line description, plus an Add button. (decided; exact card content proposed)

* FR-7. An item with stock $0$ or less is grayed out and cannot be added. Its button reads "Habis". (decided: grayed; label proposed)

* FR-8. Item photos come from a Cloudflare R2 bucket (served via a custom domain such as `https://img.yourdomain.com/<key>`), load lazily using native `loading="lazy"`, and retry once on failure. A placeholder shows while loading and when a photo is missing or fails. (decided: R2; placeholder proposed)

* FR-9. The last good item list is saved in the browser (`localStorage`). It is shown first on the next visit, and again replaced when fresh items arrive. If loading fails, the saved list stays visible. (decided: caching)

**Cart and checkout**

* FR-10. The customer can add an item to the cart. Adding an out-of-stock item is blocked, and the quantity in the cart cannot exceed the item's available `stock`. (decided + stock cap added)

* FR-11. In the cart the customer can change quantity (capped at `item.stock`), remove an item, and see the total in Rp. Pressing either the phone's hardware Back button or the on-screen close (`X`) button while the cart or hamburger menu is open closes the panel and keeps browser history in sync without leaving the website.

* FR-12. The cart is kept in the browser and survives a page reload. (decided)

* FR-13. When fresh catalog data loads, if a cart item no longer exists or its stock dropped to $0$, it is removed from the cart; if its stock dropped below the cart quantity, the cart quantity is reduced to match available stock.

* FR-14. Checkout asks for customer/recipient name and address (both required) and optional notes ("Catatan"). Name and address are saved in the customer's browser `localStorage` so repeat orders auto-fill. (decided + local auto-fill & notes proposed)

* FR-15. Checkout creates a WhatsApp message in this format and opens WhatsApp (with a clickable fallback link if the automatic redirect is blocked): customer name, each line (item name, qty, line price), total price, address, and optional notes. (decided; exact wording in Phase 5)

* FR-16. The WhatsApp number comes from a config file, so it can be changed without editing code. (decided: number supplied later)

**Header, footer, text**

* FR-17. The header shows the store logo and some opening text. (decided)

* FR-18. The footer shows contact and address information, and about. (decided)

* FR-19. All words shown to users are in Indonesian and come from `text.json`. Prices use the format `Rp15.000`. (decided: Indonesian and IDR; format proposed)

* FR-20. Position and look, text, and content are kept in JSON files (`layout.json`, `text.json`, `content.json`) so a box can be moved or reworded without editing code. (decided)

**Input page**

* FR-21. A separate page lets the admin enter their secret Write PIN (saved in `localStorage` after first entry), plus item name, price, category, description, stock, and a photo. It also lists existing items with a quick control to update stock or mark "Habis". (decided: separate page; PIN and quick stock update added)

* FR-22. The photo can be positioned (1-finger drag + range slider zoom) and previewed in a square frame, then compressed to an $800 \times 800\text{ px}$ JPEG binary blob before sending. (decided: adapted from TRACKING-ICE)

* FR-23. Sending first uploads the compressed photo binary to Cloudflare R2 (via a Cloudflare Pages Function / Worker that checks the PIN and returns the R2 `key`), and then sends the item row + `image` key to the Google Sheet via Apps Script. If the photo upload succeeds but the Sheet call fails, retrying reuses the already-uploaded R2 `key` without uploading the photo twice. The page says success only when both confirm. On failure it shows an Indonesian error and keeps the form filled.

* FR-24. The input page has a different link and a completely different name, is not linked from the catalog, and is hidden from search engines. Write operations require the secret `WRITE_TOKEN` (Admin PIN) on both the R2 upload endpoint and Apps Script. (decided)

**Install**

* FR-25. The catalog can be installed on the phone like an app (PWA) and caches the app shell (Network-First for HTML/JSON, Cache-First for hashed assets) so it updates immediately on new deploys while still opening offline to display the cached catalog. (decided)

## 6. Screens and UI requirements

The whole catalog is one page. Sections from top to bottom: header, catalog (one block per category), footer. The sticky bar is always on top.

| Screen or state | Requirement | 
| ----- | ----- | 
| Main page | Sticky bar, header, category sections with item grids, footer. Dark theme, native system font, mobile first (max content width $1100\text{ px}$ on desktop). | 
| Hamburger menu | Opens from the sticky bar. Lists header, categories, and footer. Closes after a tap, on `X` button, or on Android Back button. | 
| Search | Hidden bar that appears under the sticky bar when the search button is pressed. | 
| Cart screen | A full-screen panel opened from the cart button. Lists items with qty controls (capped at stock), remove, total, and the checkout form (name, address, optional notes). Closes on `X` or Back button. | 
| Loading | Short Indonesian loading text. Cached items, if any, are shown immediately instead. | 
| Load error | Indonesian error with a retry button, only when there is no cached list. | 
| Empty states | No search match, empty cart, and empty catalog each show a short Indonesian message. | 
| Input page | One simple form on a phone-sized screen: Admin PIN field (saved locally), item fields, photo picker with square crop preview (drag + zoom slider), submit button, success/error message, plus a collapsible quick-stock list at the bottom. | 

Touch targets are at least $44\text{ px}$ high. No sideways scrolling on a small phone screen.

## 7. Data requirements

**Google Sheet, tab `items`** (row 1 headers, exact)

| Column | Type | Notes | 
| ----- | ----- | ----- | 
| id | text | Filled automatically by the script as `'item_' + timestamp` so Sheets always treats it as text. Used by the cart. | 
| name | text | Item name (sanitized against leading `=`, `+`, `-`, `@` formula injection). | 
| price | number | IDR, whole number $\ge 0$. | 
| image | text | Cloudflare R2 object key (for example `items/1731200000-ab12.jpg`), not a full URL. | 
| category | text | Groups the item into a section. | 
| description | text | Short product details. | 
| stock | number | Whole number. $0$ or less means out of stock. | 

**Browser storage (catalog)**

* `items:v1`: last good item list.

* `cart:v1`: object `{ "<itemId>": qty }`. Only string ids and numeric quantities, never names or prices.

* `checkout:v1`: object `{ "name": "...", "address": "..." }` stored only on the customer's device to auto-fill future orders.

**Config and text files** (in `public/`)

* `config.json`: Apps Script URL, public `readToken`, `imageBaseUrl` (`https://img.yourdomain.com`), WhatsApp number.

* `text.json`: all Indonesian UI strings.

* `layout.json`: CSS theme variables (colors, spacing, grid columns) applied on boot.

* `content.json`: section order, visibility flags, and store metadata (header text, footer contact/address/about).

## 8. Non-functional requirements

* NFR-1. Compatibility: works on **Android 9+ (Chrome 88+) and iOS 16+ (Safari 16+)** as build targets. (decided)

* NFR-2. Mobile first: designed for a small portrait screen first, then larger screens. (decided)

* NFR-3. Performance: cached repeat visit shows items within about 1–2 seconds; Apps Script caches sheet data in `CacheService` (wrapped in `try/catch` in case payload exceeds $100\text{ KB}$); photos are served from Cloudflare R2 with immutable edge caching and `loading="lazy"`.

* NFR-4. Slow backend resilience: Apps Script cold starts can be slow, so reading allows 30 seconds and sending allows 45 seconds (via `AbortController`), and the browser cache hides the wait.

* NFR-5. Privacy: no accounts, no cookies, no analytics. Customer name and address never touch a server; they are saved only in local browser storage and sent directly to WhatsApp.

* NFR-6. Security: `READ_TOKEN` is public and only blocks basic bots from reading. `WRITE_TOKEN` is private (stored in the Worker/Pages secret settings and in Apps Script, entered as a PIN by the admin on the input page, and never committed to git).

* NFR-7. Accessibility basics: readable contrast on the dark theme, labels on form fields, buttons usable by touch.

* NFR-8. Cost: runs on Cloudflare Pages, Cloudflare R2 (Free Standard tier: $10\text{ GB}$ storage, $1\text{M}$ writes/mo, $10\text{M}$ reads/mo), and Google Sheets/Apps Script at zero cost.

* NFR-9. Maintainability: single monorepo containing `/catalog` and `/catalog-input`, one file per feature, state owned by a main screen, texts and layout in JSON.

* NFR-10. Search engines: the catalog can be indexed; the input page cannot (`noindex` + `robots.txt`).

## 9. Traceability: requirement to phase

| Requirements | Built in | 
| ----- | ----- | 
| FR-4, FR-21 to FR-24 (backend side), NFR-6 | Phase 1, Phase 2 | 
| NFR-1, NFR-2, FR-19, FR-20 | Phase 3 | 
| FR-1 to FR-9, FR-17, FR-18 | Phase 4 | 
| FR-10 to FR-16 | Phase 5 | 
| FR-25 | Phase 6 | 
| NFR-8, NFR-10 | Phase 7 | 
| All, final check | Phase 8 | 

The product is accepted when all Phase 8 success criteria pass.

# Part B: Technical Plan

## 10. How the pieces fit

```
Item input page (separate Pages project & link, requires WRITE_TOKEN PIN)
   |
   +--> 1. POST /upload (photo blob + PIN) --> Cloudflare Pages Function / Worker --> R2 Bucket (catalog-photos)
   |                                                                                       |
   +--> 2. POST (item data + R2 image key + PIN) --> Google Apps Script --> Google Sheet   |
                                                            ^                              |
                                                            | GET (items JSON)             | GET https://img.yourdomain.com/<key>
                                                            |                              v
                                                      Catalog site (Cloudflare Pages, your custom domain)
                                                            |
                                                            v checkout
                                                      WhatsApp (wa.me link with the order message)
```

One GitHub monorepo with two folders (`/catalog` and `/catalog-input`), deployed as two separate Cloudflare Pages projects. `catalog-input` includes a Pages Function (`functions/upload.js`) bound directly to the `catalog-photos` R2 bucket so photo uploads require zero CORS configuration and no separate Worker repository.

## 11. Fixed decisions

| Area | Decision | 
| ----- | ----- | 
| Stack | Vanilla JavaScript (ES Modules), one file per feature, Vite targeting `chrome88` and `safari16`. No framework, no legacy Babel plugin needed. | 
| Layout | Single page with sections: header (logo + opening text), catalog by category, footer (contact, address, about). | 
| Navigation | Sticky home bar (main buttons: Home -> header, Product -> first category; plus search and cart) and a hamburger drawer. Supports Android Back button and `X` button synced with `history.back()`. | 
| Search | Button in the sticky bar opens a search bar. Hidden by default. Client-side filter by name and category. | 
| Items | E-commerce style grid: name, image, details. Sheet columns: `id`, `name`, `price`, `image`, `category`, `description`, `stock`. | 
| Out of stock | Grayed out (`stock <= 0`), button reads "Habis". Cart quantity capped at `stock`. | 
| Look | Dark theme (`#121212` bg, `#25d366` default accent). Standard native system font stack. | 
| Language and money | Indonesian, IDR (`Rp15.000`). | 
| Cart | Stored in the browser (`localStorage`, in-memory fallback). | 
| Checkout | Customer/recipient name, address, and optional notes ("Catatan"), then a WhatsApp message. Name and address cached locally on device. | 
| WhatsApp message | Customer/recipient name, each line (item name, qty, line price), total price, address, optional notes. Includes a manual fallback link if auto-redirect is blocked. | 
| Data | Google Sheet read through a Google Apps Script web app (JSON cached with `CacheService`), cached client-side in `localStorage`. | 
| Input page | Separate page and link (`catalog-input`), adapted from TRACKING-ICE concepts. Crops (drag + zoom slider) and uploads photos to R2, writes row to Sheet, and supports quick stock updates. Protected by `WRITE_TOKEN` PIN. | 
| Images | Stored in Cloudflare R2 (`800x800` JPEG), served via custom domain (`https://img.yourdomain.com/<key>`), loaded lazily (`loading="lazy"`) with 1 automatic retry on failure. | 
| Layout files | JSON-driven layout (`layout.json`, `text.json`, `content.json`) with defined schemas so a box can be moved or reworded without editing code. | 
| PWA | Installable, with Service Worker using Network-First for HTML/JSON and Cache-First for `/assets/*`. | 
| Compatibility | Android 9+ (`chrome88`) and iOS 16+ (`safari16`). | 
| Hosting | Single GitHub repo connected to two Cloudflare Pages projects (`catalog` on custom domain, `catalog-input` on private `.pages.dev` URL with R2 binding). | 
| Security | Split tokens: `READ_TOKEN` (public in catalog config) and `WRITE_TOKEN` (secret PIN stored in Cloudflare env vars and Apps Script, entered by admin on input page). | 

## 12. Rules for the builder (Android 9 / iOS 16 baseline & safety)

Follow these in every phase:

* **Build Target:** Set Vite `build.target` and `build.cssTarget` to `['chrome88', 'safari16']`. Do **not** install `@vitejs/plugin-legacy` or `terser`.

* **Allowed Modern CSS:** `gap` (on both Flex and Grid), `aspect-ratio: 1 / 1`, `inset: 0`, `position: sticky`, `scroll-margin-top`, and CSS variables (`var(--x)`) are fully supported and should be used.

* **Avoid Ultra-New CSS/JS (Post-2022):** Do not use CSS `:has()`, native CSS nesting (unless processed by Vite), anchor positioning, or `Array.prototype.toSorted()` / `Object.groupBy()`.

* **Multi-line truncation:** For the 2-line card description, use `display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden;`.

* **Network timeouts:** Use `AbortController` with `setTimeout` for the 30-second GET timeout and 45-second POST timeout.

* **Safe DOM rendering:** Always insert sheet data (`name`, `category`, `description`) using `element.textContent`, **never** `innerHTML`, to prevent XSS or broken markup from special characters.

* **Formula Injection Prevention:** In Apps Script `doPost`, sanitize text fields (`name`, `category`, `description`) by prefixing a single quote `'` if the string starts with `=`, `+`, `-`, or `@`.

* **Price formatting:** Format IDR using a clean helper function `String(Math.round(n)).replace(/\B(?=(\d{3})+(?!\d))/g, '.')` prefixed with `Rp`.

* **Storage:** Wrap every `localStorage` read and write in `try/catch` with an in-memory fallback.

* **Secrets:** Never put `WRITE_TOKEN` in any file committed to Git. Only `READ_TOKEN` goes in `public/config.json`.

## 13. Proposed file tree (monorepo)

```
repo-root/
  catalog/
    index.html
    package.json
    vite.config.js          targets chrome88, safari16
    public/
      config.json           Apps Script URL, readToken, imageBaseUrl, whatsappNumber
      layout.json           theme colors, grid columns, spacing variables
      text.json             all Indonesian UI words
      content.json          section order, visibility, store header/footer content
      manifest.webmanifest, sw.js, icons (192 and 512 png)
      _headers              Cloudflare Pages cache headers
    src/
      main.js               boot, load JSON configs, mount mainscreen, register sw.js
      mainscreen/
        mainscreen.js       owns state: items, cart, query, activeModal
      stickybar/            sticky top bar
      hamburger/            section navigation drawer
      searchbar/            collapsible search bar
      header/               logo + opening text
      categories/           category sections
      itemgrid/             grid + item card (gray when stock <= 0)
      cart/                 full-screen cart panel + stock-capped qty controls
      checkout/             name + address + notes form, builds WhatsApp link
      footer/               contact, address, about
      lib/
        api.js              network calls with AbortController timeout
        cart-store.js       localStorage cart + stock reconciliation
        format.js           Rp formatting
        whatsapp.js         message builder + wa.me link
  catalog-input/
    index.html              includes <meta name="robots" content="noindex">
    package.json
    vite.config.js
    functions/
      upload.js             Cloudflare Pages Function: verifies PIN, writes JPEG to R2 bucket
    public/
      config.json           Apps Script URL, readToken, uploadUrl ("/upload")
      robots.txt            Disallow: /
    src/
      main.js               form, PIN storage, canvas crop (drag + slider), upload, quick stock list
```

**JSON Schemas (in `catalog/public/`):**

* `config.json`: `{ "apiUrl": "...", "readToken": "...", "imageBaseUrl": "https://img.yourdomain.com", "whatsappNumber": "6281234567890" }`

* `layout.json`: `{ "colors": { "bg": "#121212", "surface": "#1e1e1e", "text": "#f5f5f5", "muted": "#aaaaaa", "accent": "#25d366" }, "gridColumnsMobile": 2, "gridColumnsDesktop": 4, "maxWidthPx": 1100, "stickyBarHeightPx": 56 }`

* `content.json`: `{ "sectionsOrder": ["header", "categories", "footer"], "header": { "logoUrl": "/logo.png", "title": "Toko Contoh", "subtitle": "..." }, "footer": { "about": "...", "address": "...", "contact": "..." } }`

* `text.json`: flat key-value object of all Indonesian labels (e.g. `addBtn`, `soldOutBtn`, `cartTitle`, `emptyCart`, `searchPlaceholder`, etc.).

## 14. Build phases

### Phase 0: Prerequisites

**Depends on:** nothing.

**Steps**

1. Make sure there is a Google account, a GitHub account, and a Cloudflare account with the domain available and R2 enabled (Free Standard tier).

2. Collect the content (or use builder placeholders initially): store logo (png), opening text, contact info, address, about text, and initial category names.

3. Install Node.js (current LTS), Git, and a code editor on the computer.

4. Leave a test WhatsApp number (`6281234567890`) until launch.

**Success criteria**

* [ ] `node -v`, `npm -v`, and `git --version` print versions.

* [ ] You can sign in to Google, GitHub, and Cloudflare.

* [ ] Placeholders or real store texts and logo are ready in a folder.

### Phase 1: Backend (Google Sheet, Apps Script, and Cloudflare R2)

**Depends on:** Phase 0.

**Steps**

1. Create a Google Sheet. Rename the first tab to `items`.

2. In row 1 write these headers exactly: `id`, `name`, `price`, `image`, `category`, `description`, `stock`.

3. Add 3 sample rows by hand (`id` = `item_1`, `item_2`, `item_3`; leave `image` empty), with numbers in `price` and `stock`, and at least two different categories. Make one row `stock` = `0` and one row `stock` = `2`.

4. In the Cloudflare dashboard, go to **R2 Object Storage** and create a bucket named `catalog-photos`.

5. In the `catalog-photos` bucket settings, under **Public Access -> Custom Domains**, connect `img.<yourdomain>` so objects in the bucket are publicly readable via Cloudflare's CDN.

6. In the Google Sheet, open Extensions -> Apps Script. Paste the code below. Replace `READ_TOKEN` (public random string) and `WRITE_TOKEN` (secret admin PIN/password).

7. Set up an `onEdit` trigger in Apps Script (Triggers -> Add Trigger -> `onSheetEdit` -> From spreadsheet -> On edit) so manual edits in the Google Sheet immediately clear the cache.

8. Click Deploy -> New deployment -> Web app. Set Execute as: **Me**. Set Who has access: **Anyone**. Deploy and authorize. Copy the web app URL ending in `/exec`.

9. *(Builder Note: If building before the owner deploys Apps Script, test against a local mock server implementing the exact `doGet`/`doPost` contract below).*

```javascript
var SHEET_NAME = 'items';
var READ_TOKEN = 'PASTE_PUBLIC_READ_TOKEN';
var WRITE_TOKEN = 'PASTE_SECRET_ADMIN_PIN';
var CACHE_KEY = 'catalog_items_v1';

function sheet() {
  return SpreadsheetApp.getActive().getSheetByName(SHEET_NAME);
}

function json(o) {
  return ContentService.createTextOutput(JSON.stringify(o))
    .setMimeType(ContentService.MimeType.JSON);
}

function clearItemsCache() {
  try {
    CacheService.getScriptCache().remove(CACHE_KEY);
  } catch (e) {}
}

function onSheetEdit() {
  clearItemsCache();
}

function safeCellText(val) {
  var s = String(val || '').trim();
  if (/^[=+\-@]/.test(s)) {
    return "'" + s;
  }
  return s;
}

function doGet(e) {
  if (!e || !e.parameter || e.parameter.token !== READ_TOKEN) {
    return json({ ok: false, error: 'token' });
  }
  var cache = CacheService.getScriptCache();
  var cached = cache.get(CACHE_KEY);
  if (cached) {
    return ContentService.createTextOutput(cached)
      .setMimeType(ContentService.MimeType.JSON);
  }

  var rows = sheet().getDataRange().getValues();
  if (rows.length <= 1) return json({ ok: true, items: [] });
  rows.shift(); // remove header

  var items = [];
  rows.forEach(function (r) {
    var id = String(r[0] || '').trim();
    var name = String(r[1] || '').trim();
    var price = Number(r[2]);
    if (!id || !name || isNaN(price)) return; // skip blank or invalid rows
    items.push({
      id: id,
      name: name,
      price: price,
      image: String(r[3] || '').trim(),
      category: String(r[4] || '').trim() || 'Lainnya',
      description: String(r[5] || '').trim(),
      stock: isNaN(Number(r[6])) ? 0 : Number(r[6])
    });
  });

  var payload = JSON.stringify({ ok: true, items: items });
  try {
    // CacheService limit is 100 KB; ignore cache write failure if catalog grows huge
    cache.put(CACHE_KEY, payload, 21600);
  } catch (err) {}

  return ContentService.createTextOutput(payload)
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.waitLock(15000);
  try {
    var d = JSON.parse(e.postData.contents);
    if (d.token !== WRITE_TOKEN) return json({ ok: false, error: 'token' });

    var sh = sheet();

    // Quick stock update action
    if (d.action === 'updateStock') {
      var rows = sh.getDataRange().getValues();
      for (var i = 1; i < rows.length; i++) {
        if (String(rows[i][0]) === String(d.id)) {
          var newStock = Math.max(0, Math.round(Number(d.stock) || 0));
          sh.getRange(i + 1, 7).setValue(newStock);
          clearItemsCache();
          return json({ ok: true, id: d.id, stock: newStock });
        }
      }
      return json({ ok: false, error: 'not_found' });
    }

    // Default action: add new item (image key comes from R2 upload)
    var id = 'item_' + new Date().getTime();
    var name = safeCellText(d.name);
    var category = safeCellText(d.category || 'Lainnya');
    var description = safeCellText(d.description || '');
    var imageKey = safeCellText(d.image || '');
    var price = Math.max(0, Math.round(Number(d.price) || 0));
    var stock = Math.max(0, Math.round(Number(d.stock) || 0));

    sh.appendRow([id, name, price, imageKey, category, description, stock]);
    clearItemsCache();
    return json({ ok: true, id: id, image: imageKey });
  } catch (err) {
    return json({ ok: false, error: String(err) });
  } finally {
    lock.releaseLock();
  }
}
```

**Success criteria**

* [ ] Opening `<web app URL>?token=<READ_TOKEN>` in a browser shows JSON with `"ok":true` and the 3 sample items, where every `id` is a string starting with `item_`.

* [ ] Opening the same URL with a wrong token shows `"ok":false`.

* [ ] Trying to POST using `READ_TOKEN` fails with `"error":"token"`; POSTing with `WRITE_TOKEN` succeeds and sanitizes any leading `=` in text fields.

### Phase 2: Item input page & R2 Upload Endpoint (`catalog-input`)

**Depends on:** Phase 1. Source of concepts: https://github.com/yuzhayo/TRACKING-ICE.

**Steps**

1. Create folder `catalog-input` with its own `package.json`, `index.html`, and `vite.config.js` targeting `['chrome88', 'safari16']`.

2. Create the Cloudflare Pages Function `catalog-input/functions/upload.js` to handle `POST /upload`:

   * Expects header `x-admin-pin` (compared against `context.env.WRITE_TOKEN`) and raw binary body (`image/jpeg`).

   * Generates a unique object key: ``items/${Date.now()}-${Math.random().toString(36).slice(2, 6)}.jpg``.

   * Writes to the bound R2 bucket `context.env.BUCKET.put(key, request.body, { httpMetadata: { contentType: 'image/jpeg', cacheControl: 'public, max-age=31536000, immutable' } })`.

   * Returns `{ ok: true, key }`.

3. Build one mobile-friendly form in `catalog-input/src/main.js`:

   * **PIN Admin (`WRITE_TOKEN`)**: password input, saved in `localStorage` (`admin_pin:v1`) so the owner only enters it once on their phone.

   * **Item fields**: name (required), price (whole number $\ge 0$), category (text input with `<datalist>` populated from existing categories), description, stock (whole number $\ge 0$), and photo picker.

4. Implement canvas square crop and compression:

   * Show a fixed square crop frame with **1-finger drag to pan** and an **`<input type="range">` slider to zoom**.

   * Export an $800 \times 800\text{ px}$ JPEG binary `Blob` via `canvas.toBlob(cb, 'image/jpeg', 0.82)`.

5. Implement the two-step submit flow (with retry resilience):

   * Disable the submit button while sending.

   * **Step A (Photo Upload):** If a photo is picked and does not yet have an uploaded `pendingR2Key` for the current crop, `POST` the JPEG `Blob` to `/upload` with header `x-admin-pin: <PIN>`. Save the returned `key` in `pendingR2Key`.

   * **Step B (Sheet Append):** Send a `POST` to the Apps Script URL (`content-type: text/plain;charset=utf-8`, 45-second `AbortController` timeout) with JSON body `{ token: pin, name, price, category, description, stock, image: pendingR2Key || '' }`.

   * If Step A succeeds but Step B fails, keep `pendingR2Key` in memory so tapping Submit again skips Step A and only retries Step B.

   * On `ok: true`, clear the item fields and `pendingR2Key` (keep the Admin PIN) and show an Indonesian success message. On `error: 'token'`, show "PIN Admin salah".

6. Below the form, fetch the current items (using `readToken` from `public/config.json`) and display a simple list where the admin can tap a button to set an item's stock to `0` ("Set Habis") or enter a new stock number (sends `action: 'updateStock'`).

7. Add `<meta name="robots" content="noindex">` in `index.html` and `public/robots.txt` with `Disallow: /`. Do not link this page from the catalog.

**Success criteria**

* [ ] Submitting with a wrong PIN fails at the `/upload` or Apps Script step with "PIN Admin salah" and writes nothing to R2 or the Sheet.

* [ ] Submitting a valid item uploads a square JPEG (under $300\text{ KB}$) to R2 under `items/...` and appends one row to the Sheet with that R2 key in the `image` column.

* [ ] If the Apps Script call fails after the photo uploads, clicking Submit again reuses the existing R2 key without uploading a duplicate image.

* [ ] Tapping "Set Habis" on an existing item updates its stock to `0` in the Google Sheet and clears the cache.

### Phase 3: Catalog foundation (`catalog`)

**Depends on:** Phase 1.

**Steps**

1. In `/catalog`, run `npm init -y` and `npm i -D vite`.

2. Add `vite.config.js`:

   ```javascript
   export default {
     build: {
       target: ['chrome88', 'safari16'],
       cssTarget: ['chrome88', 'safari16']
     }
   };
   ```

3. Add scripts in `package.json`: `"dev": "vite"`, `"build": "vite build"`, `"preview": "vite preview"`.

4. Create `index.html` with `<meta name="viewport" content="width=device-width, initial-scale=1">`, `lang="id"`, Apple PWA meta tags (`<link rel="apple-touch-icon" href="/icon-192.png">`, `<meta name="apple-mobile-web-app-capable" content="yes">`), and `<div id="app"></div>`.

5. Create the file tree from section 13.

6. Create `public/config.json`, `public/layout.json`, `public/text.json`, and `public/content.json` matching the schemas in section 13.

7. In `main.js`, load the JSON configuration files, apply the CSS variables from `layout.json` onto `document.documentElement.style`, and mount `mainscreen.js` with state `{ items: [], cart: {}, query: '', activeModal: null }`.

**Success criteria**

* [ ] `npm run dev` opens a dark-themed page with no console errors.

* [ ] `npm run build` finishes cleanly with no errors.

* [ ] Changing a color in `layout.json` or a word in `text.json` updates the UI without editing any `.js` or `.css` file.

### Phase 4: Catalog features

**Depends on:** Phases 1 and 3.

**Steps**

1. `api.js`: write `fetchItems()` that calls `{apiUrl}?token={readToken}` using `AbortController` with a 30-second timeout and returns the `items` array.

2. Caching: save the last good result in `localStorage` key `items:v1`. On start, render cached items immediately (if any), then fetch fresh items in the background and re-render. If the fetch fails and there is a cache, keep showing the cache; if there is no cache, show an Indonesian error with a retry button.

3. `stickybar`: `position: sticky; top: 0; z-index: 50;` bar with a hamburger button, Home button (scrolls to Header), Product button (scrolls to first Category), a search toggle button, and a cart button showing the total item count badge.

4. `hamburger`: drawer listing Header, each Category, and Footer. Tapping an entry closes the menu and scrolls to the target section. Use `scroll-margin-top` on sections equal to `stickyBarHeightPx` so the sticky bar never covers section titles.

5. `searchbar`: hidden by default under the sticky bar. Toggling the search button shows/focuses it. Typing filters items by `name` and `category` (case-insensitive) and hides categories with 0 matching items. Show an empty-search message if no items match.

6. `header` & `footer`: render logo, title, subtitle, about, address, and contact text from `content.json` using `textContent`.

7. `categories` & `itemgrid`: group items by `category` (cap max container width at `maxWidthPx` on desktop). Render each card with a square image container (`aspect-ratio: 1 / 1`), item name, formatted price (`Rp15.000`), 2-line truncated description, and an Add button.

8. Images: load `${config.imageBaseUrl}/${item.image}` with `<img loading="lazy">`. Add an `onerror` handler that retries loading the image once after $1.5\text{ seconds}$, and if it fails again (or if `item.image` is empty), displays a clean dark placeholder box.

9. Out of stock (`stock <= 0`): add a `soldout` class (`opacity: 0.45`), disable the Add button, and set its label to `text.soldOutBtn` ("Habis").

**Success criteria**

* [ ] All sample items appear in their respective categories with R2 images or placeholders.

* [ ] The item with `stock: 0` is grayed out and its button reads "Habis" and is disabled.

* [ ] Jumping to any section via the hamburger menu or sticky bar buttons lands cleanly with the section title visible below the sticky bar.

* [ ] Filtering via the search bar updates categories in real time and shows an Indonesian message when nothing matches.

* [ ] Reloading the page with the network disabled in DevTools immediately renders the cached catalog from `items:v1`.

### Phase 5: Cart and checkout

**Depends on:** Phase 4.

**Steps**

1. `cart-store.js`: save the cart in `localStorage` key `cart:v1` as `{ "<itemId>": qty }`. Wrap in `try/catch` with an in-memory fallback.

2. **Stock cap & reconciliation:**

   * When `onAdd(itemId)` or `+` in the cart is pressed, check current `item.stock`. If `currentQty >= item.stock`, do not increment and show a brief toast/message (e.g., "Stok maksimal tercapai").

   * When fresh items arrive from `fetchItems()`, reconcile the cart: remove any `itemId` no longer in the catalog or whose `stock <= 0`, and clamp `cart[itemId] = Math.min(cart[itemId], item.stock)`.

3. **Back button & Close (`X`) button history sync:**

   * When opening the full-screen Cart panel or Hamburger menu, call `history.pushState({ modal: 'cart' }, '')`.

   * Listen to `window.addEventListener('popstate', ...)` so pressing the Android Back button closes the active modal without leaving the page.

   * When the user taps the on-screen close (`X`) button (or taps a hamburger link while a modal state is pushed), call `history.back()` if `history.state && history.state.modal` is active so no orphan history state is left behind.

4. `checkout` form inside the Cart panel:

   * Inputs: Recipient Name (required), Delivery Address (required), and Notes / "Catatan (opsional)".

   * Load saved `{ name, address }` from `localStorage` (`checkout:v1`) on mount, and save them when updated.

5. `whatsapp.js`: build the message using labels from `text.json`:

   ```
   Nama: <customer name>
   Pesanan:
   - <item name> x<qty> - Rp<line price>
   - <item name> x<qty> - Rp<line price>
   Total: Rp<total>
   Alamat: <address>
   Catatan: <notes> (omit this line if notes is empty)
   ```

6. Build the link as `https://wa.me/<number>?text=` + `encodeURIComponent(message)`. Open via `window.location.href = url` and also reveal a direct clickable `"Buka WhatsApp"` link in case the browser blocks automatic redirects. Keep the cart after opening WhatsApp and provide a "Kosongkan keranjang" button.

**Success criteria**

* [ ] For the sample item with `stock: 2`, the customer can add up to 2 units, and tapping `+` a third time is blocked with a message.

* [ ] Refreshing the page preserves the cart and auto-fills the customer's saved name and address.

* [ ] Opening the cart and closing it via **either** the on-screen `X` button or the Android Back button keeps browser history clean (pressing Back once more exits cleanly, no double-back required).

* [ ] Checkout opens WhatsApp with the properly formatted message, accurate line/total prices, and special characters (`&`, `#`, `?`) intact.

### Phase 6: PWA & Offline App Shell

**Depends on:** Phase 4.

**Steps**

1. Add `public/manifest.webmanifest` with app name, short name, `start_url: "/"`, `display: "standalone"`, `orientation: "portrait"`, dark `background_color` and `theme_color` (`#121212`), and $192 \times 192$ and $512 \times 512$ PNG icons.

2. Add `public/sw.js` with two caching rules for same-origin requests (ignore cross-origin requests):

   * **Network-First (fallback to cache):** for navigation (`index.html`) and `/*.json` config files. This guarantees new deploys or JSON edits appear immediately on the very next reload when online, while still booting from cache when offline.

   * **Cache-First:** for Vite's hashed `/assets/*` bundles and icons.

3. Register `/sw.js` in `main.js`.

4. Add `public/_headers`:

   ```
   /*.html
     Cache-Control: no-cache
   /sw.js
     Cache-Control: no-cache
   /*.json
     Cache-Control: no-cache
   /assets/*
     Cache-Control: public, max-age=31536000, immutable
   ```

   *(Note: Do NOT add a `/* /index.html 200` `_redirects` file, so missing assets properly return 404).*

**Success criteria**

* [ ] Chrome DevTools -> Application -> Manifest and Service Workers show no errors.

* [ ] Turning on Airplane Mode / Offline in DevTools and reloading the page still loads the full UI and renders cached items from `localStorage`.

* [ ] Deploying a change updates the site on the **first** reload when online.

### Phase 7: Deploy

**Depends on:** Phases 5 and 6.

**Steps**

1. Push the monorepo (`/catalog` and `/catalog-input`) to a private GitHub repository. Ensure `.gitignore` excludes `node_modules/` and `dist/`.

2. In Cloudflare Pages, create Project 1 (**Catalog**):

   * Connect to the GitHub repo, set **Root directory** to `catalog`, build command `npm run build`, output directory `dist`.

   * Attach the custom domain (for example `yourdomain.com`) and verify HTTPS is active.

3. In Cloudflare Pages, create Project 2 (**Input Page**):

   * Connect to the same GitHub repo, set **Root directory** to `catalog-input`, build command `npm run build`, output directory `dist`.

   * Give the project an unrelated random name so its `*.pages.dev` URL cannot be guessed. Do not attach the custom domain to this project.

   * In Project 2 **Settings -> Variables and Secrets**, add secret `WRITE_TOKEN` matching the PIN in Apps Script.

   * In Project 2 **Settings -> Functions -> R2 bucket bindings**, bind variable name `BUCKET` to the `catalog-photos` R2 bucket.

**Success criteria**

* [ ] The custom domain serves the catalog over `https` and `https://img.<yourdomain>/<key>` serves photos from R2.

* [ ] The input page opens at its separate `*.pages.dev` URL, has `noindex` headers/meta, and has zero links pointing to it from the catalog.

* [ ] Adding an item with a photo on the live input page uploads the photo to R2, writes the row to the Sheet, and displays the item + photo on the live catalog upon reload.

### Phase 8: Test and launch

**Depends on:** Phase 7.

**Steps**

1. Test on a real Android phone (Chrome) and an iPhone (Safari iOS 16+).

2. Run the checklist below.

3. Put the real WhatsApp business number and real store texts into `catalog/public/config.json` and `content.json`, push to GitHub, and place one real end-to-end test order.

**Success criteria**

* [ ] Layout has no horizontal overflow on a $360\text{ px}$ wide phone screen.

* [ ] Sticky bar, hamburger section jumps, search filter, stock caps, and grayed-out items all work smoothly.

* [ ] Pressing the phone's Back button or the `X` button closes the cart or hamburger drawer cleanly.

* [ ] Offline start: opening the site with mobile data/Wi-Fi turned off loads the app shell and displays the cached catalog.

* [ ] Checkout opens WhatsApp with the exact order format and right store number.

* [ ] No console errors on mobile Chrome or Safari.

## 15. Still to decide (Owner inputs needed before final launch)

1. **Store content (can use placeholders until Phase 8):** Store name, logo PNG, header opening text, footer about/address/contact text, and accent color preference.

2. **WhatsApp business number (needed before Phase 8 launch):** In international format starting with `62`.

3. **Custom domain name (needed in Phase 1 / Phase 7):** The domain on Cloudflare for the catalog (`yourdomain.com`) and the R2 image subdomain (`img.yourdomain.com`).