# Agent Setup Guide: Catalog Website (end to end)

You are an agent controlling the owner's browser. Your job is to set up the **accounts and dashboards** for the catalog website described in `catalog_website_plan.md`. Follow this guide in order. Do not improvise a different approach.

Known facts (from the owner, do not re-ask):
- Domain: `omahwangi.web.id`, registered at an Indonesian registrar (panel is in Indonesian: "Pengelolaan omahwangi.web.id").
- Catalog URL (proposed): `https://omahwangi.web.id`
- Photo URL (proposed): `https://img.omahwangi.web.id`
- Stack is fixed: Google Sheet + Apps Script (data), Cloudflare R2 (photos), Cloudflare Pages (two projects: `catalog`, `catalog-input`), GitHub (repo), WhatsApp (orders).
- Cloudflare account is on the **Free** plan and must stay there.

---

## 0. Rules (read first, apply always)

### Never do these
1. **Never click "Matikan Kunci Domain"** and never open "Kode EPP". The domain lock stays **Aktif**. It does not block nameserver changes.
2. **Never enter payment details, passwords, 2FA codes or CAPTCHA answers.** If a page asks for them, stop and ask the owner to type them. Then continue.
3. **Never buy or upgrade anything.** Do not click Upgrade, Purchase Workers Paid, or any paid plan. Always choose Free.
4. **Never substitute tools.** No Vercel, Netlify, Firebase, Google Drive for photos, separate Cloudflare Worker, or API tokens. The plan already decided.
5. **Never change plan decisions** (names of sheet tab, columns, keys, file paths). Copy them exactly.
6. **Never commit `WRITE_TOKEN` to Git** or paste it into any file in the repo.
7. **Never retype code from memory.** Copy code blocks verbatim from the plan file.

### Verify before you click
Before each step, check that the page title or URL matches what the step says. If the page looks different (new label, missing button), **stop, describe what you see, and ask the owner.** Do not guess a nearby button. Dashboard labels change over time; this guide gives the intent plus the expected label.

### Be efficient
- Use direct URLs: `dash.cloudflare.com`, `sheets.new`, `github.com/new`.
- Do each step once. After a step passes its check, do not re-verify it later.
- Do not keep refreshing slow things (DNS). Move on to a step that doesn't depend on it, and come back.
- Fill forms by pasting values, not by typing character by character, where possible.

### Keep a log
Keep a running log (a file or the chat) with this table, filled in as you go. If your context resets, read it first.

| Item | Value | Done |
|---|---|---|
| Cloudflare zone status for omahwangi.web.id | | |
| Cloudflare nameserver 1 / 2 | | |
| Sheet URL | | |
| Apps Script `/exec` URL | | |
| READ_TOKEN | | |
| WRITE_TOKEN (see section 1) | | |
| R2 bucket `catalog-photos` | | |
| R2 custom domain `img.omahwangi.web.id` | | |
| GitHub repo URL | | |
| Pages project 1 name / URL | | |
| Pages project 2 name / URL | | |

---

## 1. Before you start

### 1.1 Owner logs in
Ask the owner to log in themselves (you never type passwords) to:
- Google (the account that will own the Sheet)
- GitHub
- Cloudflare (create an account if none)
- The domain registrar panel

Wait for "done" before continuing.

### 1.2 Things you need from the owner
- Store name, logo PNG, header text, about, address, contact, accent color. **If missing, use the placeholders from the plan** ("Toko Contoh"). Do not wait.
- WhatsApp number. **If missing, use the test number `6281234567890`.**
- Initial categories. If missing, use "Minuman" and "Makanan".
- Whether the code repo already exists (built from plan Phases 2 to 6). If not, stop at section 6 and tell the owner the code must be built first.
- What tools you have besides the browser (terminal with `git`?). This decides section 6.

### 1.3 Generate the tokens
Use the browser DevTools console on any blank tab (type the line by hand; Chrome blocks pasting into the console).

READ_TOKEN (32 hex characters):
```js
Array.from(crypto.getRandomValues(new Uint8Array(16)),b=>b.toString(16).padStart(2,'0')).join('')
```
WRITE_TOKEN (20 hex characters, the admin PIN):
```js
Array.from(crypto.getRandomValues(new Uint8Array(10)),b=>b.toString(16).padStart(2,'0')).join('')
```
Also generate the random Pages project name for the input page (8 hex characters; use `new Uint8Array(4)`).

Record all three in the log. **Show the WRITE_TOKEN to the owner once, in your final report, and tell them to save it in a password manager.** READ_TOKEN is public by design.

---

## 2. Move DNS to Cloudflare (needed for R2 custom domain and Pages custom domain)

### 2.1 Check what already exists (do this first)
The registrar panel has "Kelola DNS" and "Email Forwarding". Moving nameservers removes the registrar's DNS records.
1. Registrar panel: Dashboard -> Domain Saya -> `omahwangi.web.id` -> **Kelola DNS**. List every record (type, name, value).
2. Also open **Email Forwarding** and note whether any forwarding exists.
3. If there are MX records or email forwarding in use, **stop and ask the owner** whether email on this domain matters. Do not continue until answered.

### 2.2 Add the domain to Cloudflare
1. `dash.cloudflare.com` -> **Add a domain** (or "Onboard a domain") -> enter `omahwangi.web.id`.
2. Choose the **Free** plan.
3. Cloudflare scans existing DNS records. Compare with the list from 2.1. Add any missing record exactly as listed (MX and TXT records especially).
4. Continue until Cloudflare shows **two nameservers** (for example `xxx.ns.cloudflare.com`). Copy both into the log.
5. If Cloudflare shows a DNSSEC warning, or says the domain is not supported, **stop and ask.**

### 2.3 Change nameservers at the registrar
1. Registrar panel -> `omahwangi.web.id` -> left menu **Nameserver** (not "Buat Nameserver").
2. Choose the option to use custom nameservers. Enter the two Cloudflare nameservers. Remove the old ones. Save.
3. Do **not** touch "Kunci Domain".
4. Back in Cloudflare, click the "check nameservers" button once. Status shows **Pending** until it propagates (minutes to a few hours).

**Do not wait here.** Continue with sections 3 and 4 and return when you need the domain (sections 5 and 7). Before using the domain, confirm Cloudflare shows the zone as **Active**.

---

## 3. Google Sheet and Apps Script

### 3.1 Sheet
1. Open `sheets.new`. Name the file `catalog`.
2. Rename the first tab to exactly `items`.
3. Row 1 headers, exactly, one per column A to G: `id`, `name`, `price`, `image`, `category`, `description`, `stock`.
4. Add three sample rows (leave `image` empty):
   - `item_1`, any name, number price, category "Minuman", stock `5`
   - `item_2`, any name, number price, category "Makanan", stock `2`
   - `item_3`, any name, number price, category "Makanan", stock `0`

### 3.2 Apps Script code
1. Extensions -> **Apps Script**.
2. Delete the default code. Paste the Phase 1 code block **verbatim from the plan file**.
3. Replace only `PASTE_PUBLIC_READ_TOKEN` and `PASTE_SECRET_ADMIN_PIN` with the tokens from the log. (There is no folder id in this version; that line was removed.)
4. **Editor pitfall:** the editor auto-closes brackets and auto-indents when text is typed. Prefer pasting. If you must type, afterwards press Ctrl+A and check there are no duplicated `}` or `)`. Click **Save**.

### 3.3 Edit trigger
1. Left sidebar -> clock icon (**Triggers**) -> **Add Trigger**.
2. Function: `onSheetEdit`. Event source: **From spreadsheet**. Event type: **On edit**. Save.
3. Google asks for authorization. Choose the owner's account. If it shows "Google hasn't verified this app", click **Advanced** -> **Go to (project name) (unsafe)** -> **Allow**. This is the owner's own script. Mention it in your report.

### 3.4 Deploy as web app
1. **Deploy** -> **New deployment** -> gear icon -> **Web app**.
2. Execute as: **Me**. Who has access: **Anyone** (not "Anyone with Google account").
3. Deploy, authorize if asked, copy the URL that ends in `/exec`. Record it.

### 3.5 Check
Open in a new tab:
- `<exec URL>?token=<READ_TOKEN>` -> must show `"ok":true` with 3 items, each `id` starting `item_`.
- `<exec URL>?token=wrong` -> must show `"ok":false`.

If you see a Google login page or HTML instead of JSON, the access setting is not "Anyone" or you used a `/dev` URL. Fix by editing the deployment (see section 9).

**Any later code change needs Deploy -> Manage deployments -> edit (pencil) -> Version: New version -> Deploy.** Saving alone does not update the live script.

---

## 4. Cloudflare R2 bucket

1. `dash.cloudflare.com` -> **R2 Object Storage**.
2. If it asks to add a payment method or "Purchase R2" before you can continue: **stop and ask the owner.** Cloudflare commonly requires a card for verification even on the free tier; the free tier stays free within its limits (10 GB storage, 1M writes, 10M reads per month). The owner must decide and enter the card themselves. If the owner refuses, stop. Do not switch to another storage service; report back so the plan can be changed.
3. **Create bucket**, name exactly `catalog-photos`. Leave location on automatic and class on Standard.
4. Leave the `r2.dev` public URL **disabled** (it is for testing only).
5. Wait until the Cloudflare zone for `omahwangi.web.id` is **Active** (section 2). Then: bucket -> **Settings** -> **Custom Domains** -> **Add** -> `img.omahwangi.web.id` -> confirm. Wait until its status shows Active.

---

## 5. Domain order check

Do not start sections 7.3 (custom domain on Pages) or 8 until the zone is **Active** and the R2 custom domain is Active. Everything in sections 6 and 7.1 to 7.2 can be done earlier.

---

## 6. GitHub repo

Prerequisite: the monorepo exists on the PC with folders `catalog/` and `catalog-input/` (built from plan Phases 2 to 6). Confirm that `catalog-input/functions/upload.js` exists.

1. `github.com/new` -> name `catalog-store` (or the owner's choice) -> **Private** -> no template. Create.
2. Make sure a `.gitignore` containing `node_modules/` and `dist/` exists at the repo root.
3. Get the code into the repo:
   - **If you have a terminal with `git`:** `git init`, `git add .`, `git commit`, `git branch -M main`, `git remote add origin <repo URL>`, `git push -u origin main`. If it asks for credentials, ask the owner to authenticate. Do not type their password.
   - **If you only have the browser:** repo page -> **Add file -> Upload files**. Drag the folders in. Never include `node_modules/` or `dist/`. GitHub's web upload accepts about 100 files per batch, so split into batches if needed.
4. Check that the repo shows `catalog/`, `catalog-input/` and the `functions/upload.js` file on the `main` branch.
5. Search the repo for the WRITE_TOKEN value. It must not appear anywhere.

---

## 7. Cloudflare Pages: two projects

Path: **Workers & Pages** -> **Create application** -> **Pages** -> **Connect to Git**. (If the page offers only "Workers" and the Pages option is hidden, look for a "Pages" link or tab on the create page. If still not found, stop and ask.)

First time only: choose **GitHub**, then **Install & Authorize**. Grant access to **only** the `catalog-store` repo.

### 7.1 Project 1: Catalog (public)
- Project name: `omahwangi-catalog` (any name is fine; it is public)
- Production branch: `main`
- Root directory: `catalog`
- Build command: `npm run build`
- Build output directory: `dist`
- Environment variable: `NODE_VERSION` = `22`
- **Save and Deploy.** Wait for success. Record the `*.pages.dev` URL.

### 7.2 Project 2: Input page (private, hidden)
- Project name: the random 8-character name from section 1.3 (not guessable, no words like "admin" or "input")
- Production branch: `main`
- Root directory: `catalog-input`
- Build command: `npm run build`
- Build output directory: `dist`
- Environment variable: `NODE_VERSION` = `22`
- **Save and Deploy.** The first deploy may work without the secret and binding. That is expected. Record the URL.

After it is created, open Project 2 -> **Settings**:
1. **Variables and Secrets** -> Add -> type **Secret** -> name `WRITE_TOKEN` -> value = the WRITE_TOKEN from the log -> save. Set it for **Production**.
2. **Bindings** (older label: **Functions -> R2 bucket bindings**) -> Add -> **R2 bucket** -> variable name exactly `BUCKET` -> bucket `catalog-photos` -> save. Set it for **Production**.
3. **Redeploy**, because secrets and bindings only apply to new deployments: Deployments -> latest production deployment -> **Retry deployment** (or push an empty commit). Wait for success.

Do **not** attach any custom domain to Project 2.

### 7.3 Custom domain on Project 1 (after the zone is Active)
Project 1 -> **Custom domains** -> **Set up a custom domain** -> `omahwangi.web.id` -> continue / activate. Wait until it shows Active with HTTPS.

---

## 8. Put real values in config and redeploy

Edit these files in the repo and commit to `main` (this triggers new builds):

`catalog/public/config.json`:
```json
{
  "apiUrl": "<Apps Script /exec URL>",
  "readToken": "<READ_TOKEN>",
  "imageBaseUrl": "https://img.omahwangi.web.id",
  "whatsappNumber": "<62... number, or the test number 6281234567890>"
}
```

`catalog-input/public/config.json`:
```json
{
  "apiUrl": "<Apps Script /exec URL>",
  "readToken": "<READ_TOKEN>",
  "uploadUrl": "/upload"
}
```

Also update `catalog/public/content.json` and `layout.json` with the store texts and accent color if the owner supplied them. Never put `WRITE_TOKEN` in any of these files.

Wait for both Pages projects to finish deploying.

---

## 9. End-to-end test

Run in this order. Record pass or fail for each.

| # | Test | Expected |
|---|---|---|
| 1 | Open `https://omahwangi.web.id` | Dark catalog loads over HTTPS, 3 sample items in 2 categories, no console errors |
| 2 | The `stock: 0` item | Grayed out, button "Habis", disabled |
| 3 | Open Project 2 URL, enter a **wrong** PIN, submit an item | "PIN Admin salah", nothing in R2, no new Sheet row |
| 4 | Enter the correct PIN, add a test item with a photo | Success message; one new Sheet row with `image` like `items/<timestamp>-xxxx.jpg`; file visible in R2 under `items/` |
| 5 | Reload the catalog | The test item appears with its photo |
| 6 | Open `https://img.omahwangi.web.id/<that key>` in a private window | The photo loads |
| 7 | Input page: set the test item to "Habis" | Sheet stock becomes 0; catalog shows it grayed out after reload |
| 8 | Add the `stock: 2` item to cart, tap `+` three times | Blocked at 2 with "Stok maksimal tercapai" |
| 9 | Checkout with name and address | WhatsApp opens with the exact order format |
| 10 | Catalog DevTools -> Application: manifest and service worker | No errors; offline reload still shows UI and cached items |
| 11 | Search engines | Input page has `noindex` meta and `robots.txt` `Disallow: /`; catalog has no link to the input page |

**Cleanup:** delete the test row from the Sheet and the test object from R2 (R2 -> `catalog-photos` -> Objects -> select -> delete).

---

## 10. Troubleshooting (use this before trying anything else)

| Symptom | Cause | Fix |
|---|---|---|
| Cloudflare zone stays Pending | Nameservers not saved at registrar, or still propagating | Re-open registrar -> Nameserver and confirm both Cloudflare nameservers are saved. Otherwise wait. Do not touch the lock. |
| R2 asks for a payment method | Normal verification | Stop. Ask owner (see section 4). |
| Apps Script URL shows a Google login or HTML page | Access not "Anyone", or `/dev` URL used | Edit deployment: Execute as Me, access Anyone, New version. Use the `/exec` URL. |
| Apps Script change has no effect | Old version still deployed | Manage deployments -> New version |
| `/upload` returns 401 or `{"ok":false}` | Secret `WRITE_TOKEN` missing or different from Apps Script | Re-enter the secret, redeploy Project 2 |
| `/upload` returns 500 | Binding `BUCKET` missing or misspelled | Add binding named exactly `BUCKET`, redeploy |
| Input page says it cannot reach Apps Script | POST not sent as `content-type: text/plain;charset=utf-8` | Fix in `catalog-input/src/main.js` per plan Phase 2 |
| Photos 404 | R2 custom domain not Active, wrong `imageBaseUrl`, or key mismatch | Check R2 custom domain status; compare Sheet `image` value to R2 object key |
| Photos show placeholder offline | Expected (photos are cross-origin and not precached) | Not a bug |
| Pages build fails: Node or "vite not found" | Old Node version or wrong root directory | Set `NODE_VERSION=22`; check root directory is `catalog` or `catalog-input` |
| Catalog still shows old data | `localStorage` cache or old service worker | Hard reload; Application -> Service Workers -> Unregister, then reload |
| Names in the Sheet start with an apostrophe | Formula-injection guard shows the `'` | Report to the owner; the fix is to format those columns as plain text and change `safeCellText` |

If a failure is not in this table, **stop and report** what you see (page URL, exact message). Do not try a different tool or workaround.

---

## 11. Final report to the owner

Return exactly this:
1. Catalog URL, input page URL (private), image domain.
2. Sheet URL and Apps Script `/exec` URL.
3. The **WRITE_TOKEN** (admin PIN), shown once. Tell the owner to store it in a password manager and enter it on their phone on first use.
4. Test results table from section 9.
5. What is still placeholder: WhatsApp number if test number was used, store texts and logo.
6. Anything you skipped or the owner must do: test on a real Android phone and iPhone, enter the real WhatsApp number in `catalog/public/config.json`, place one real test order.
7. Warnings: Google authorization granted to the script; email forwarding status of the domain if it was relevant; whether a payment method was added to Cloudflare.
