# Adventure Lab — Claude Code Workspace Context

Read this file at the start of every session before doing anything.

---

## Who I Am

**Owner:** Omri Mohr  
**Business:** Adventure Lab — tour operation based in Bacalar, Mexico  
**Google account:** ceo.gmiexperience@gmail.com

---

## AI Coding Tools — Routing Rules

Two AI tools are used for development. Both MUST read this file at the start of every session and update it at the end.

### Claude Code (careful, step-by-step)

Use for:
- QBO API integration (OAuth, delayed charges, payments)
- Revenue calculation logic
- Bug fixes in existing scripts
- Database schema changes (new columns, new tabs)
- Approval workflow logic
- Anything where a mistake could corrupt data or break the pipeline

### MiniMax (fast execution)

Use for:
- Dashboard UI (HTML/CSS/JS)
- Frontend styling and layout
- Simple API endpoints that only read data
- Documentation updates
- Creating new static files
- Refactoring code that doesn't change logic

### Sync Rule

- Both tools work on the same codebase at `D:\adventure-lab`
- CONTEXT.md is the bridge — always update it after every session
- Review changes before pushing (`git diff` for dashboard, review code for apps-script)
- Never push from both tools simultaneously

---

## This Workspace Has Two Projects

### 1. `dashboard/` — Frontend (GitHub)
- **What it is:** The owner + coordinator dashboard. HTML/CSS/JS, hosted on GitHub Pages (free, public URL).
- **GitHub repo:** https://github.com/omrimohr/adventurelab-dashboard
- **How to push changes:** Use git inside the `dashboard/` folder:
  ```
  git add .
  git commit -m "describe what you changed"
  git push
  ```
- **Important:** Always commit and push after making changes so the live site updates.

### 2. `apps-script/` — Backend (Google Sheets)
- **What it is:** The Google Apps Script that powers the data pipeline. Reads from FareHarbor, writes to Google Sheets, serves data to the dashboard.
- **Google Sheet:** ADV. LAB - OP DASHBOARD  
  - Sheet ID: `1YzTUE1EEDUdxnZRCOdfxqPA3pj2nDdEN534XkjOcn6c`
- **Script ID:** `1-O7IcZNMU4IGTIevLnU6NNpSeNacvhAl2RWBw32a-Ly-eSSKx4fLgX4x`
- **Script files:**
  - `Config.js` — constants, sheet names, API keys
  - `Helpers.js` — utility functions
  - `Webhook.js` — receives FareHarbor webhooks
  - `Tours.js` — tour booking logic
  - `DailySummary.js` — daily rollup calculations
  - `KPI.js` — KPI scoring for coordinators
  - `ManualInputs.js` — manual data entry forms
  - `CrewPayments.js` — crew payment calculations (Wed→Wed cycle)
  - `API.js` — serves data to the dashboard frontend
  - `QBO.js` — QuickBooks Online sync (Phase 1 complete)
  - `Corrections.js` — manual booking corrections + day authorization
- **How to push changes:** Run `clasp push` from inside the `apps-script/adv-lab-dashboard/` folder (that's where `.clasp.json` lives).
- **How to pull latest from Google:** Run `clasp pull` from inside the `apps-script/adv-lab-dashboard/` folder.

---

## Key People

| Name | Role |
|------|------|
| Omri Mohr | Owner — talks to you |
| Edder Martínez | Coordinator — daily dashboard user |
| Jesus Velazquez Cruz | Coordinator — daily dashboard user |
| Angel | Commercial manager — FareHarbor config |

---

## Project Status (update this as things get done)

- [x] FareHarbor affiliate price sheet fix — completed (Angel)
- [x] Coordinator SOP written
- [x] QBO Staging layer (Phase 1) — complete
- [x] clasp local dev workflow — set up
- [x] GitHub repo connected to Claude Code
- [x] QBO Phase 2 — Schema + backend (Ops/Commercial/Admin approval chain) — COMPLETE, verified end-to-end (admin_approve_day tested both pass and block cases)
- [x] Owner commercial dashboard — Facturación page with staging/approval/QBO workflow — COMPLETE
- [x] QBO Phase 3 logic (customer/product matching, quincena Invoice accumulation, payment recording) — COMPLETE, code only, redesigned 2026-06-22 from daily-invoice to quincena-accumulation model
- [x] QBO OAuth2 — CONNECTED and posting successfully to the sandbox, full real approval chain verified end-to-end on real bookings (2026-06-23: 7/7 affiliates posted, 0 errors) as of 2026-06-24. The earlier "Google redirect interception" blocker turned out to be a different bug (missing `action=` on redirect), now fixed. A separate DocNumber-too-long bug (see session log below) also fixed.
- [x] QBO sandbox automation — `morningRun` now auto-builds + auto-posts QBO staging for yesterday, daily at 4am Cancun, gated entirely by `CFG.SANDBOX_MODE` (see session log below). **Must flip to `false` before connecting a real production QBO company.**
- [ ] Commercial corrections UI (edit gross/net/commission per booking) — pending
- [ ] Coordinator limited view (Edder & Jesus) — future

### Session log (2026-06-11)
- Fixed revenue calc bugs: Commission = Gross - Net (Tax no longer subtracted); Gross = invoice_price when it exceeds receipt_total (Habitas/Mia/Solana).
- Removed duplicate Bookings row for booking #353980577 (kept Casa Hormiga Hotel row, deleted stale Direct row).
- `writeBookingToSheet()` now upserts by Booking PK (not UUID) to prevent future duplicates when a booking's UUID changes across edits/rebooks.

### Session log (2026-06-13) — Part 2: Pax fix
- Pax now comes directly from `booking.customer_count` — removed the old keyword-based `extractPax()`/`derivePax()` custom-field logic and unused `PEOPLE_KEYWORDS` config.
- Added `withRetry()` helper (Helpers.js) for transient "Service Spreadsheets timed out" errors.
- `rebuildBookingsFromRaw_Start()` now clears the Bookings sheet instead of delete+recreate (avoided repeated timeouts).
- Rebuild completed successfully: 1164 written, 28 skipped, 0 errors.
- Removed a leftover time-driven trigger on `rebuildBookingsFromRaw_Continue` that was re-running the full rebuild every ~5 min.

### Session log (2026-06-13/14) — Parts 3-6: Tours columns, performance fix, corrections workflow
- **Part 3:** Added new Tours columns (category, duration, marina, pax discrepancy, revenue/commission/IVA totals, booking count & PKs, crew, resources, flags, booking notes) plus a `custom_fields` column carried over from Bookings.
- **Part 4:** `rebuildTours()` was timing out ("Service Spreadsheets timed out" / "Exceeded maximum execution time"). Rewrote it to build all tour rows in memory and write the whole Tours sheet in one bulk `setValues()` call instead of row-by-row writes. Verified: 573 tours rebuilt in ~9.4s.
- **Part 5:** Added correction/authorization columns to Bookings (`Corrected Pax`, `Last Corrected By`, `Last Corrected At`, `Authorized`, `Authorized By`, `Authorized At`) and a new `Corrections Log` tab (append-only audit trail) — both defined in `COLS` in Config.js.
- **Part 6:** New `Corrections.js` with 3 API endpoints:
  - `bookings_for_date` — returns bookings for a date with `Effective Pax` (Corrected Pax if set, else Customer Count).
  - `save_correction` — writes a correction to Bookings + appends an entry to Corrections Log.
  - `authorize_day` — marks all bookings on a date as Authorized.
  - All three tested successfully against the live deployment.

### Session log (2026-06-21) — QBO Phase 2: Schema, backend + Facturación UI
**Files changed:** Config.js, QBO.js, API.js, dashboard/index.html

**Config.js changes:**
- Added Admin/QBO columns to BOOKINGS: `Admin Approved`, `Admin Approved By`, `Admin Approved At`, `QBO Posted`, `QBO Posted At`, `QBO Reference`
- Rewrote QBO_STAGING schema with full approval chain: `Affiliate Shortname`, `Booking Lines JSON`, `Ops Approved/By/At`, `Commercial Approved/By/At`, `Admin Approved/By/At`, `QBO Posted/At`, `QBO Delayed Charge ID`

**QBO.js changes:**
- Rewrote `buildQBOStagingForDate()` — now reads shortname from Raw JSON, uses Effective (corrected) values, populates Ops + Commercial approval flags from Bookings columns, new schema
- Added `getQBOStagingForDate(dateStr)` — returns staging rows for a date with summary stats
- Added `adminApproveDay(dateStr, approvedBy)` — enforces Ops + Commercial must both be Y before Admin approval
- Added `postDelayedChargeToQBO(stagingRow)` — Phase 3 skeleton (logs placeholder, no QBO API calls yet)
- Added `postQBODate(dateStr)` — batch poster
- Added `findOrCreateQBOCustomer(affiliateName, shortname)` — Phase 3 skeleton
- Added `findOrCreateQBOProduct(tourName, itemPK)` — Phase 3 skeleton
- Added `getQBOToken()`, `getQBORealmId()`, `qboApiRequest()` — QBO auth infrastructure
- Added `addQBOStagingColumns()` — one-time migration for existing QBO Staging sheets
- Updated `approveQBOInvoice()` — now writes to `Commercial Approved` columns (not old generic `Status`/`Approved By`/`Approved At`)
- Updated `getQBOStagedInvoices()` — parses `Booking Lines JSON` column
- **NOTE:** `approveQBODate()` still writes to `Status = approved` — it should write to `Commercial Approved` columns. The old `Status`-based approve workflow is deprecated; use `qbo_approve_date` endpoint which calls `approveQBOInvoice()` which correctly writes to commercial columns.

**API.js changes:**
- Added new endpoints documented in header: `qbo_staging_for_date`, `admin_approve_day`, `post_to_qbo`, `qbo_status`
- Added `qbo_staging_for_date` case → calls `getQBOStagingForDate()`
- Added `admin_approve_day` case → calls `adminApproveDay()`
- Added `post_to_qbo` case → calls `postQBODate()`
- Added `qbo_status` case → calls `getQBOStagingForDate()`

**dashboard/index.html changes:**
- Added "Finanzas" sidebar section with "Facturación" nav item
- Added "Facturación" bottom nav button (mobile)
- Added full `#page-facturacion` page with: date picker, 4 action buttons (Generar/Aprobar Comercial/Aprobar Admin/Enviar QBO), summary stats, expandable invoice rows
- Added JS: `loadFacturacion()`, `loadFactData()`, `renderFacturacion()`, `renderInvoiceRow()`, `factToggleDetail()`, `factChangeDate()`, `factBuild()`, `factCommApprove()`, `factAdminApprove()`, `factPostQBO()`
- Button state logic: Comercial disabled until Ops approved; Admin disabled until Ops+Comercial approved; QBO disabled until Admin approved (all enforced on backend too)

**One-time migrations to run after clasp push:**
1. Run `addMissingBookingsColumns()` in Apps Script editor → adds new Admin/QBO columns to Bookings
2. Run `addQBOStagingColumns()` in Apps Script editor → adds new columns to QBO Staging sheet

**PENDING (Phase 3):**
- QBO OAuth2 auth not configured — `postDelayedChargeToQBO()` and `findOrCreateQBOCustomer/Product()` are skeletons
- `approveQBODate()` still uses old `Status=approved` pattern — deprecated, use `qbo_approve_date` endpoint instead
- Owner commercial dashboard corrections UI (editing gross/net/commission per booking) not yet in dashboard

---

## Rules for Claude Code

1. **Read this file first** before every session.
2. **Always ask before writing or changing code.** Show a plan first.
3. **Work one step at a time.** Commit after each completed section.
4. **For dashboard changes:** edit files in `dashboard/`, then `git push`.
5. **For Apps Script changes:** edit files in `apps-script/`, then `clasp push`.
6. **Never push broken code.** Test logic before committing.
7. **Keep responses short.** Omri prefers concise answers and step-by-step instructions.
8. **End of every session:** Update this CONTEXT.md to reflect what was done, then commit and push it to GitHub. No exceptions.

---

## Architecture Summary

```
FareHarbor  →  Webhook.js  →  Google Sheets (database)
                                     ↓
                               Apps Script API.js
                                     ↓
                          dashboard/index.html (GitHub Pages)
                          viewed by Omri + coordinators
```

---

## Important Notes

- Omri is not a developer. Explain git/terminal steps clearly and one at a time.
- When giving terminal commands, always specify which folder to run them in.
- On Windows, use `cd /d D:\path\to\folder` to switch drives.
- The clasp `.clasp.json` must be inside `apps-script/` — not in the user home folder.


---

## Session Log — 2026-06-21

### What was done
- Debugged why `qbo_build` returned "No booked bookings" for dates that had confirmed bookings in `bookings_for_date`
- **Root cause:** `buildQBOStagingForDate()` was capped at `CFG.LIMITS.API_MAX_ROWS = 500` rows, but the Bookings sheet has 1283 rows — June 2026 bookings were past row 500
- **Fix:** Changed `readSheetAsObjects(ss, BOOKINGS, CFG.LIMITS.API_MAX_ROWS)` → `readSheetAsObjects(ss, BOOKINGS, bSheet.getLastRow())` in `buildQBOStagingForDate()`; same fix for Tours read
- **Bonus find + fix:** Pre-existing CFG.TZ bug — was `'America/Mexico_City'` (UTC-6) but sheet timezone is `'America/Cancun'` (UTC-5). This caused all dates read through `readSheetAsObjects` to appear one day earlier than actual. Fixed to `'America/Cancun'`
- Added `qbo_debug_date` and `qbo_full_test` debug actions to API.js (for one-shot testing, one URL = one approval)
- Verified: `qbo_full_test` for 2026-06-20 now returns 5 correct affiliate invoices with accurate dates

### Deployments
| Version | URL suffix | Notes |
|---------|-----------|-------|
| @27 (current) | `AKfycbxR9uAlteCmJF_dAfR07zTiC9d8iCbmxRUebpn_Yf8w6OJJljsDkzODs6XqwJ7Q63nH1w` | Fix CFG.TZ to America/Cancun; full sheet reads |
| @26 | `AKfycbxNiM2p25xHq1zvFx6EkAHcEvRjJEuiEFzdw0p4D12n1RV7HfQ2juKddJDVt_lh401qxg` | Add qbo_full_test action |
| @25 | `AKfycbzhE1RkLsuJeIzIIkGtCmYcB7scdQ3qzYfBbe3ngDJvf4jz4FlwuPE9WEbghC-YFuHY6w` | Fix row cap in buildQBOStagingForDate |
| @24 | `AKfycbxnviU6StJ4lq9CrVie8kG1vqXjxy0W2N8jKq_XAsiOPIZunTGyrfodIKGdC0Yr67_LNQ` | (earlier work) |

### Current state
- Dashboard: `https://omrimohr.github.io/adventurelab-dashboard/` (API URL updated to @27)
- Apps Script URL: `https://script.google.com/macros/s/AKfycbxR9uAlteCmJF_dAfR07zTiC9d8iCbmxRUebpn_Yf8w6OJJljsDkzODs6XqwJ7Q63nH1w/exec`
- QBO Staging: 5 affiliate invoices for 2026-06-20 ( Cliente Directo, Hotel Amainah, The Yak Lake House, Adventure Lab PDV Yak, Hotel Aires)
- CFG.TZ fixed to 'America/Cancun' (was 'America/Mexico_City' — caused all dates to appear 1 day early)
- buildQBOStagingForDate now reads full sheet (not capped at 500 rows)

### Dashboard fixes (2026-06-21)
- `getTourFlags()`: added deduplication by label — flags could appear twice if custom Flag text overlapped with auto-detected conditions
- `.td-lbl`: added `min-width:80px; flex-shrink:0` — fixed field titles drifting far from values on wide screens
- "Ingreso" label: added tooltip "(Net después de comisión)" — clarifies it = Net (MXN), what AL receives after paying affiliate

### About "Ingreso" (Revenue field)
- = sum of **Net (MXN)** from all BOOKED bookings per tour/day
- Formula: Gross (MXN) − Commission (MXN) = Net (MXN)
- Does NOT include Tax (MXN) or Discount (MXN)
- Example: Gross 1105, Commission 221 → Ingreso = 884 MXN
- This is what Adventure Lab keeps; affiliate commission is already deducted
- Shell permission system requires allowOnce per unique URL — wildcard `*` in settings.local.json does NOT work for query param variations

### Known issues
- Permission prompt fatigue: every new deployment = new URL = fresh approvals. No permanent fix yet.
- `qbo_debug_date` (read-only, harmless) still in API.js. `qbo_full_test` (destructive — deleted all QBO Staging rows on every call) was **removed** 2026-06-21, deployed @35.

### Session log (2026-06-22) — QBO redesign: daily Delayed-Charge model → quincena Invoice accumulation
**The prompt spec changed** from "one Invoice per affiliate per day" (built 2026-06-21, see entry below) to **one accumulating Invoice per affiliate per quincena** (1st-15th / 16th-end of month), appended to daily, tracked via a `CustomerMemo` marker (`"ABIERTO — Q1 Jun 2026"`). The day-per-invoice functions below are superseded — `postDelayedChargeToQBO()` no longer exists, replaced by `postDayToQBO()`.

**Config.js / Helpers.js:**
- Added `CFG.QUINCENA_CUTOFF = 15` and `CFG.QBO.OPEN_INVOICE_MARKER = 'ABIERTO'`.
- Added `getPeriodString(date)` (Helpers.js) — returns `"Q1 Jun 2026"` / `"Q2 Jun 2026"`. Verified live.
- QBO_STAGING: added `Period`, `Booking PKs` columns; **renamed** `QBO Delayed Charge ID` → `QBO Invoice ID` (renamed the live header cell directly — not a duplicate column; old staging data preserved).
- BOOKINGS: added `QBO Invoice ID`, `QBO Period` columns (old `QBO Reference` left in place, deprecated/unused).
- `buildQBOStagingForDate()` now populates `Period` + `Booking PKs` on every staging row.

**QBO.js — new accumulation logic:**
- `findOrCreateQBOInvoice(qboCustomerId, periodString)` — searches for an open invoice (`CustomerMemo` contains "ABIERTO" + period); if none, counts all invoices (open+closed) for that period to pick the next letter suffix (Q1, Q1b, Q1c...). Returns a memo string for the caller to create with — doesn't create itself, since QBO requires a non-empty `Line[]` on create and only the caller has that day's lines ready.
- `postDayToQBO(stagingRow)` (replaces `postDelayedChargeToQBO`) — finds/creates the period's invoice, dedupes by scanning existing line `Description`s for `"Booking #<PK>"` (skips already-posted bookings — re-run safe), builds only the new lines, adds one commission line per day (deduped by exact description so re-runs don't double it), then either creates the invoice (if new) or does a sparse update with the **full** existing+new `Line[]` (QBO sparse update doesn't merge sub-arrays).
- `postQBODate()` now also calls new `markBookingsQBOPosted()` — writes `QBO Posted`/`QBO Invoice ID`/`QBO Period` back onto each individual Bookings row (Part 7 steps 9-10), not just the staging row.
- `getQBOInvoiceStatus(period)` (new) — groups staging rows by affiliate per period; if QBO is connected, fetches the live invoice to report `open`/`closed` (via the "ABIERTO" marker); otherwise `pending`/`not_created`. New endpoint: `qbo_invoice_status`.

**Tested live, safely, in disconnected mode** (built staging on throwaway test dates 2026-06-16/17/18, force-approved via temporary debug endpoints, ran `post_to_qbo` and `qbo_invoice_status`, confirmed graceful `pending`/`not_created` results with clear per-affiliate reasons, no exceptions — then deleted all test data and removed the debug endpoints). Real production staging data (2026-06-20 manual, 2026-06-21 from this morning's `morningRun` cron) was left untouched throughout.

**Still pending (unchanged):** real QBO OAuth2 connection. `recordQBOPayments()` reused as-is (already targets whatever invoice ID it's given, so works the same under accumulation).

### Session log (2026-06-21, continued) — QBO Phase 2 verification + Phase 3 logic
**Verified Part 2 (admin approval) end-to-end** on real staging data (2026-06-20), reverting all test values afterward:
- `adminApproveDay()` correctly sets Admin Approved=Y when Ops + Commercial are both Y.
- Correctly **blocks** with per-affiliate reasons (e.g. "Hotel Amainah: commercial not approved") when either gate is missing.
- Found and fixed a real bug along the way: `authorizeDay()`/`commercialApproveDay()` (Corrections.js) wrote to Bookings in a loop without `SpreadsheetApp.flush()` — writes were silently lost across separate web-app requests. Added `flush()` after every multi-row write loop (Corrections.js, QBO.js).

**Researched QBO's API and confirmed: there is no "Delayed Charge" entity in QuickBooks Online's public REST API** (UI-only feature; confirmed via Intuit developer forums). Since our own Ops/Commercial/Admin chain already holds a day before billing, we post a real **Invoice** instead once admin-approved — functionally equivalent, no Delayed-Charge-then-convert step needed.

**Built Parts 4-7 (QBO.js), code-complete but not connected to live QBO:**
- `findOrCreateQBOCustomer()` — real query/create logic (CompanyName=shortname → DisplayName → create; "Cliente Directo" maps to fixed "Ventas Directas" customer).
- `findOrCreateQBOProduct()` — real query/create logic (Sku=itemPK → Name=tourName → create as Service item).
- `findOrCreateQBOCommissionItem()` — find-only (per spec, should already exist in QBO).
- `postDelayedChargeToQBO()` — rewritten to build and POST a real Invoice (line per booking + commission line with TaxCodeRef), returns `qbo_invoice_id` + `qbo_customer_id`. Stored in the `QBO Delayed Charge ID` column (name kept for compatibility — it actually holds an Invoice ID now, noted in Config.js).
- `recordQBOPayments()` (new) — for ADV-collected bookings only (cash/card/online), creates one QBO Payment per payment type, linked to the invoice via `LinkedTxn`.
- `postQBODate()` — rewritten to write results back to the QBO Staging sheet (QBO Customer ID, QBO Posted/At, Invoice ID, Payments Posted/At) instead of just counting.
- Added `payment_lines_json` + `payments_posted`/`payments_posted_at` columns to QBO_STAGING; `buildQBOStagingForDate()` now populates payment lines from ADV-collected bookings.
- Added `CFG.QBO` config block (Config.js) — Realm ID, income account, IVA/exempt tax code IDs, commission item name, direct-customer mapping, payment method refs. All blank placeholders until OAuth is connected.
- All new functions degrade gracefully (`status: 'pending'`) when `qboApiRequest()` has no token — verified live: `post_to_qbo` on a real date returns `posted:0` with a clear per-affiliate "QBO not connected" reason for each, no exceptions.
- New endpoint: `qbo_migrate_columns` — runs `addQBOStagingColumns()` via API (no need to open the Apps Script editor).

**Deliberately not done:** real QBO OAuth2 connection (Realm ID, access token, tax code IDs, income account ref) — `getQBOToken()` returns null until `QBO_ACCESS_TOKEN` is set in Script Properties. This was explicit scope: build everything except actually connecting.

### Next steps
1. Connect real QBO OAuth2 credentials (Script Properties: `QBO_ACCESS_TOKEN`, `QBO_REALM_ID`) + fill in `CFG.QBO` tax code / income account IDs once known.
2. Commercial corrections UI (editing gross/net/commission per booking in Facturación).
3. Coordinator limited view for Edder & Jesus.

---

## Session log (2026-06-22) — QBO OAuth2 Connection Attempt (INCOMPLETE)

**Goal:** Connect the Apps Script to QBO Sandbox via OAuth2 so `post_to_qbo` actually sends invoices.

### What worked
- QBO sandbox company created: Realm ID `9341457328235620`
- Apps Script deployed with `oauthScopes` in `appsscript.json`: `script.external_request`, `spreadsheets.currentonly`
- New scopes authorized by running functions directly in Apps Script editor
- `post_to_qbo` endpoint correctly returns "No admin-approved invoices ready for QBO" (no crashes — logic works)
- `qbo_debug_staging` endpoint confirmed: staging sheet has 8 rows for 2026-06-20, dates correct, Admin Approved = empty

### What failed — OAuth token exchange
All redirect-based approaches failed to capture the authorization code:

1. **Apps Script redirect URL** (`https://script.google.com/macros/s/.../exec`) → Google shows an intermediate auth confirmation page BEFORE redirecting, so the code appears in the Google page's URL, not our redirect
2. **Google OAuth Playground** → "invalid_client" — client ID not recognized because Intuit's app is registered as a web app, not a generic OAuth client
3. **`localhost:8080/callback` as redirect** → Browser refuses to connect (nothing listening on localhost)
4. **Browser popup from HTML file** → QBO's popup was blocked or the code capture via `window.location` failed because cross-origin
5. **Authorization code reuse** → OAuth codes are single-use; reusing old codes gives "Invalid authorization code"

### Root cause of OAuth problem
Intuit's OAuth flow includes Google's account selection/consent page which loads INSIDE the redirect chain, so the final `?code=XXX` lands on Google's page, not our redirect URI. The Apps Script web app's "authorize" prompt appears AFTER the redirect, not before.

### What was built in Apps Script
- `exchangeOAuthCode(code)` — exchanges code for token, stores in Script Properties (WORKING but never received a valid code)
- `qbo_test` endpoint — tests if token is stored, tries to reach QBO API
- `qbo_debug_staging` endpoint — inspects staging sheet contents
- All new scopes (UrlFetchApp, Spreadsheets) deployed and authorized
- `postDayToQBO()` fixed: null guard, Period-based date lookup (timezone-safe), graceful QBO not-connected handling

### Current Apps Script URL
`https://script.google.com/macros/s/AKfycbwkipk2SgAGeMwtlAP2gRnB7bJw9YymYVrr5IJ-JkhM4A3PFtRHqrsZf5SYw7JZjLhZPg/exec`

### Remaining: how to get the token stored
Best remaining options to try:
1. **Intuit's own OAuth2 test tool** — https://developer.intuit.com/app/developer/playground — may have a direct token generation flow
2. **Manual copy-paste**: Get the code from the browser URL bar manually, paste it to me, I call `exchangeOAuthCode` directly via a new API endpoint
3. **Ask Intuit community** for the simplest sandbox OAuth2 flow that avoids Google's redirect interception
4. **Service Account**: more complex but avoids user-based OAuth entirely
4. Dashboard's Facturación page may still reference the old `qbo_delayed_charge_id`/per-day labels — check `dashboard/index.html`'s JS against the renamed `QBO Invoice ID` column and new `Period` field next time it's touched.

---

## Session log (2026-06-23/24) — QBO OAuth connected; posting verified end-to-end on 2026-06-20

**Goal:** Get `post_to_qbo` to actually post to the QBO sandbox (it had only ever returned graceful "not connected" results before). Found and fixed four separate bugs, not just a stale token.

**Bugs found and fixed (QBO.js / API.js):**
1. **OAuth callback never fired.** Intuit's redirect lands on our exec URL with `?code=&realmId=` but no `action=` param (our registered redirect_uri has no query string), so `doGet` silently defaulted to `ping` every time — this was the real cause of the 2026-06-22 "OAuth blocked" entry above, not Google's redirect page. Fixed: `doGet` now treats a request carrying `code` with no `action` as `oauth_callback`.
2. **Wrong API host.** `qboApiRequest()` (and a hardcoded debug endpoint) called `https://quickbooks.api.intuit.com` (production) for a sandbox realm → every call failed with `ApplicationAuthorizationFailed` (403/3100) even with a fresh, correctly-scoped token. Fixed both to `https://sandbox-quickbooks.api.intuit.com`.
3. **Invalid query field.** `findOrCreateQBOInvoice()` filtered Invoice by `CustomerMemo`, which QBO's query API doesn't allow ("property 'CustomerMemo' is not queryable") → always failed, masked as "not connected". Fixed: query by `CustomerRef` only, filter memo/period client-side.
4. **Missing sandbox reference data.** `CFG.QBO.INCOME_ACCOUNT_REF`/`TAX_CODE_IVA`/`TAX_CODE_EXEMPT`/`PAYMENT_METHOD_REFS` were all blank placeholders, and the `Commission` item didn't exist yet in the sandbox. Filled with real sandbox IDs (Account "Services"=1, TaxCode TAX/NON, PaymentMethod Cash=1/Visa=3) and created the Commission item (id 19) — **these are sandbox-only values, must be re-picked with the accountant before connecting production.**
5. **Real bug confirmed live:** `findQBOStagingRow()` matched by Period+Affiliate instead of Date+Affiliate, so when two dates in the same quincena share an affiliate, the QBO Posted/Invoice ID write-back updated the wrong row. Fixed to match Date+Affiliate (the row's natural key); also fixed the same Period-only matching bug in the `qbo_approve_chain` test endpoint.

**Verified live on 2026-06-20** (6 affiliates, MXN 24,443 total net): rebuilt staging, force-approved Ops/Commercial/Admin for sandbox testing, called `post_to_qbo` — all 6 posted with correct QBO Invoice IDs (157, 155, 146, 159, 147, 161) and Customer IDs, payments recorded. Note: posting one date also flushes any other already-approved-but-unposted dates in the same quincena period, by design — QBO invoices accumulate per affiliate per period, not per day.

**Also fixed (pre-existing, found while testing):** 2026-06-22 had 8 stale Admin-Approved-but-unposted rows from earlier testing sessions that got swept into this run's post (their invoices were already correct since each row's own booking lines were used) — only the *sheet write-back* for 3 of 2026-06-20's rows needed manual correction, done via a one-time endpoint (removed after use).

**Cleanup:** removed all one-time/sensitive debug endpoints added during this session (`qbo_set_tokens` token-injection, `qbo_correct_staging_row`). Kept general-purpose debug tools (`qbo_raw_query`, `qbo_test_customer`, `qbo_check_token`, `qbo_refresh`) since they have ongoing diagnostic value.

**Still pending:** access token refresh is still manual only (`qbo_refresh` endpoint) — no time-driven trigger auto-refreshes it, so it'll go stale again after ~60 min of inactivity. Production migration will need separate Production keys (Client ID/Secret) from Intuit and a fresh OAuth handshake against the real company — the sandbox keys hardcoded in `exchangeOAuthCode()`/`refreshOAuthToken()` won't work there.

### Next steps
1. Add a time-driven trigger to auto-refresh the QBO access token (e.g. every 45 min) so it doesn't go stale between sessions.
2. When ready for production: request Production keys from Intuit, swap `CLIENT_ID`/`CLIENT_SECRET` and the API host back to `quickbooks.api.intuit.com`, re-pick `INCOME_ACCOUNT_REF`/tax codes/payment method refs with the accountant for the real company, redo OAuth against the real company.
3. Commercial corrections UI (editing gross/net/commission per booking in Facturación).
 4. Coordinator limited view for Edder & Jesus.

---

## Session log (2026-06-23) — TaxCode auto-create, sandbox fully verified

**Goal:** Fix the `IVA 16%` tax code creation (was failing silently), then verify the full post flow works end-to-end with the new booking-line fields.

### What was built / changed

**QBO.js — `postDayToQBO()` line fields (new description + DocNumber + ServiceDate):**
- `Description`: `'Booking #' + l.booking_pk + ' — ' + l.pax + 'px — ' + affiliate + ' (' + date + ')'` (removed tour name, added pax + affiliate)
- `ServiceDate: date` on every booking line AND commission line
- `DocNumber`: `period.replace(' ','-') + '-' + affiliate.substring(0,12)` on invoice create + sparse update
- `qboApiRequest()` error logging improved: now logs real QBO error code + message instead of silently returning null

**QBO.js — `resolveOrCreateIVATaxCode()` (new function):**
- Looks up `IVA 16%` by exact name — returns cached ID if already resolved
- If not found: queries `TaxAgency`, then calls `POST /v3/company/{realmId}/taxservice/taxcode` (the correct Intuit-documented endpoint — NOT `/taxrate` + `/taxcode`)
- Body: `{ TaxCode: 'IVA 16%', TaxRateDetails: [{ TaxRateName: 'IVA 16%', RateValue: 16, TaxApplicableOn: 'Sales', TaxAgencyId: <agency_id> }] }`
- Caches resolved ID in `QBO_IVA_TAX_CODE_ID` Script Property
- IVA booking lines now call `resolveOrCreateIVATaxCode()` instead of using `CFG.QBO.TAX_CODE_IVA`; exempt lines unchanged

**QBO.js — `findQBOFirstTaxAgency()` (new helper):**
- Queries `SELECT * FROM TaxAgency MAXRESULTS 1` — needed because TaxService requires a TaxAgencyId

**API.js — new endpoints:**
- `qbo_list_tax_codes` — `SELECT * FROM TaxCode`, returns full list with IDs
- `qbo_list_tax_agencies` — `SELECT * FROM TaxAgency`, returns list
- `qbo_resolve_iva_tax_code` — calls resolver; `&force=true` clears cache first

### What was verified live

- Tax agencies in sandbox: Arizona Dept. of Revenue (id 1), Board of Equalization (id 2) — used id 1
- `qbo_resolve_iva_tax_code&force=true` → created `IVA 16%` in sandbox, TaxCode id=4, TaxRate id=4, 16%, active
- `IVA 16%` confirmed in `qbo_list_tax_codes` (count went from 5 → 6)
- QBO token cache: still valid from earlier session — refreshed once (`qbo_refresh` returned ok)
- All 6 staging rows for 2026-06-20 already posted in prior session (QBO Invoice IDs 157/155/146/159/147/161 confirmed) — no re-post needed

### Production note (important)
- Sandbox TaxAgency is "Arizona Dept. of Revenue" — wrong for MX. Production QBO company will need SAT (Servicio de Administración Tributaria) as the agency. Either pre-create `IVA 16%` in production QBO (so resolver finds it), or update `findQBOFirstTaxAgency()` to use the correct MX agency ID.
- Tax rates and codes are company-specific — re-confirm with accountant before production.
- Token still expires every 60 min — auto-refresh trigger still pending.

### Current Apps Script URL
`https://script.google.com/macros/s/AKfycbxnEEtwt_cysahiCnd1PQvXJdmaq19obsvs5EDJIm8DvJv9PjLFmCFwnXLhoT_qcB2yHA/exec` (updated 2026-06-24 — the previous URL's deployment was deleted; this is also what `dashboard/index.html`'s `API` constant now points to)

### Next steps
1. Auto-refresh token trigger (time-driven, every 45 min)
2. Commercial corrections UI (Facturación gross/net/commission edits)
3. Coordinator limited view for Edder & Jesus
4. When moving to production: swap sandbox TaxAgency → SAT; pre-create or auto-create `IVA 16%` in production QBO; fresh OAuth handshake against real company

---

## Session log (2026-06-24) — DocNumber bug fix, real end-to-end QBO post verified, dashboard URL repair

**Context:** A separate debugging session (via the Apps Script editor directly, not clasp) had already deployed several new diagnostic endpoints and fixes (`@76`–`@86`) chasing a "postDayToQBO always returns null/pending" symptom, with a working hypothesis that `findOrCreateQBOInvoice` wasn't finding invoices due to a missing `CustomerMemo`. That deployment chain replaced the old documented live deployment ID entirely (it no longer exists — `clasp deploy -i` against it now fails with "Requested entity was not found").

**Investigation — disproved the CustomerMemo hypothesis:** Added a temporary `qbo_trace_post` endpoint. Trace showed `findOrCreateQBOInvoice` correctly found the existing invoice with `CustomerMemo: "ABIERTO — Q2 Jun 2026"` already set — not the problem.

**Real root cause found:** `postDayToQBO`'s `DocNumber` field — `period.replace(' ', '-') + '-' + affiliate.substring(0, 12)` — produced 22–24 character strings, but QBO's Invoice `DocNumber` field has a **21-character limit**. Every single create/sparse-update call was silently rejected by QBO's validation, and `qboApiRequest()` swallows faults into a bare `null`, which is why it always looked like a generic "not connected" failure. Confirmed by replicating the exact payload via hand-rolled fetches (bypassing `DocNumber`) — those succeeded every time; only the real `DocNumber`-bearing payload failed.

**Fix (QBO.js):**
- `formatPeriodDocNumber(periodString)` — builds `"Q2/JUN/26"` from `"Q2 Jun 2026"` (9 chars, well under the limit).
- `resolveUniqueDocNumber(base)` — queries existing invoices by `DocNumber LIKE` and appends `A`, `B`, `C`... for additional invoices needed in the same period (DocNumber must be unique per company; one period now needs one DocNumber per affiliate's invoice).
- Invoice creation uses `resolveUniqueDocNumber(formatPeriodDocNumber(period))`; sparse updates reuse the invoice's **existing** `DocNumber` (`fresh.Invoice.DocNumber`) rather than recomputing — the letter suffix is only resolved once, at creation.

**Verified live, real approval chain, real bookings (not sandbox bypass):** Built staging for 2026-06-23 (7 affiliates, 14 real bookings), ran `authorize_day` → `commercial_approve_day` → `admin_approve_day` → `post_to_qbo`. Result: **7/7 posted, 0 errors.** Two brand-new invoices created with the new format confirmed live: `Q2/JUN/26` (Agam Hotel) and `Q2/JUN/26A` (Carolina Bacalar) — confirms `lines_added > 0` works for genuinely new invoices, not just the "already posted" case.

**Cleanup:**
- Fixed `dashboard/index.html`'s `API` constant — was pointing to the now-deleted deployment ID, updated to the current live one, committed and pushed.
- Removed all temporary diagnostic functions/endpoints added during this investigation (`qbo_trace_post`'s endpoint, `traceSparseUpdateRaw`, `traceWithInternalRefresh`, `traceViaSharedRequest` and their API.js cases) — QBO.js and API.js are back to production-only code.

**Still pending:** auto-refresh trigger for the QBO token (still manual via `qbo_refresh`, ~60 min lifespan); production migration steps (separate Production keys, real TaxAgency, fresh OAuth handshake) noted above.

---

## Session log (2026-06-24) — QBO line/description format changes, deployed + verified

**Goal:** Three formatting changes to `postDayToQBO()` in QBO.js, requested by Omri:
1. Booking line `ServiceDate` now uses the tour's own date (`Tour Date` from Bookings, carried through staging as `tour_date`) instead of the staging/posting date — stays inside `SalesItemLineDetail` since QBO's schema has no top-level `Line.ServiceDate`.
2. Booking line `Description` changed to `'Booking #' + booking_pk + ' — ' + contact_name + ' — ' + pax + 'px'` (added client name via new `contact_name` field, dropped tour name/affiliate/date from the string).
3. Commission line `Description` simplified to just `'Commission'`, with its date now living only in `ServiceDate`. Dedup logic updated accordingly: since Description is no longer unique per day, re-run safety now checks Description **and** ServiceDate together on existing invoice lines (previously matched on Description alone, which embedded the date).

**Booking Lines JSON** (staging) now also carries `tour_date` and `contact_name` per line — added in `buildQBOStagingForDate()`.

**Deployed:** pushed via `clasp push`, then `clasp deploy -i <live deployment id> -V 97` (clasp push alone only updates `@HEAD`, not the live exec URL — must explicitly redeploy the pinned deployment ID each time).

**Verified live** on a fresh random date (2026-06-19, not previously posted): ran `authorize_day` → `commercial_approve_day` → `qbo_build` → `qbo_approve_chain` (test-only force-approve, same pattern as prior sessions) → `post_to_qbo`. Result: 6/8 affiliate invoices posted successfully. Confirmed via `qbo_raw_query` directly against the QBO sandbox Invoice object that the new line shows `"Description":"Booking #356003482 — Luis A Garcia — 1px"` and `"ServiceDate":"2026-06-19"` inside `SalesItemLineDetail` — exactly as intended.

**New (pre-existing, unrelated) bug found while testing:** 2/8 invoices (Mia, Intrepid Travel) failed with `"Invalid Line TaxCode in the request" / "Valid line TaxCodes for US should be TAX or NON. Supplied value: 4"`. Root cause: `resolveOrCreateIVATaxCode()` returns the custom `IVA 16%` TaxCode id (created in the 2026-06-23 session) for IVA-taxed lines, but this sandbox company is provisioned as a **US-template** company, which only accepts `TAX`/`NON` on `SalesItemLineDetail.TaxCodeRef` — custom tax codes aren't valid there even though the TaxCode record itself exists. Diagnosed via a temporary `qbo_trace_create` debug endpoint (added, used, then fully removed — QBO.js/API.js are back to production-only code, deployed @97).
- Not fixed yet — affects only IVA-taxed lines, and is a sandbox-company-template limitation, not a code bug. Will resolve itself once on a real MX production company (which uses SAT tax codes, not the US TAX/NON pair) — worth re-checking once production OAuth is connected.

**Live Apps Script URL unchanged:** `https://script.google.com/macros/s/AKfycbxnEEtwt_cysahiCnd1PQvXJdmaq19obsvs5EDJIm8DvJv9PjLFmCFwnXLhoT_qcB2yHA/exec` (same deployment ID, now @97).

---

## Session log (2026-06-24, continued) — Sandbox auto-post pipeline (SANDBOX_MODE)

**Goal:** automate the full daily QBO posting pipeline for sandbox testing — no manual approval clicks needed, while keeping a documented one-flag path back to the real approval gate for production.

**Config.js:**
- Added `CFG.SANDBOX_MODE = true`. Flip to `false` for production — restores the Admin Approved gate immediately, no other code changes needed.

**QBO.js:**
- `postQBODate()`'s row filter changed from `Admin Approved === 'Y'` to `(CFG.SANDBOX_MODE || Admin Approved === 'Y')` — when sandbox mode is on, every staged-but-unposted row posts regardless of approval state.

**Helpers.js:**
- `morningRun()` now also calls `postQBODate(yesterdayStr())` after `buildQBOStagingForYesterday()` — the daily cron now builds AND posts automatically.
- Added `resetMorningRunTrigger()` — deletes only existing `morningRun` triggers (leaves `wednesdayPaymentRun`/`archiveCheck` alone) and installs a fresh one at **4am America/Cancun**. Exposed via `?action=qbo_setup_morning_trigger` for one-off re-runs.

**appsscript.json:**
- Added the `script.scriptapp` OAuth scope (required for `ScriptApp.getProjectTriggers()`/`deleteTrigger()`). New scopes require a one-time manual re-authorization — Omri ran `resetMorningRunTrigger` once from the Apps Script editor to grant it; confirmed working afterward via the API endpoint (idempotent — re-running always reports `removed: 1`, never accumulates duplicate triggers).

**Verified:** `morningRun` trigger confirmed installed, single instance, 4am Cancun time.

**Production checklist reminder:** before connecting a real QBO company, set `CFG.SANDBOX_MODE = false`. Until then, every staged invoice posts automatically every morning with zero human review — acceptable for sandbox, not for real money.

---

## Session log (2026-06-28) — 5 reliability fixes for QBO posting pipeline, all verified live

**Goal:** harden the daily QBO posting pipeline (token freshness, failure visibility, production-cutover safety, race conditions, row-matching correctness). Worked through 5 requested fixes one at a time, each deployed and verified live before moving to the next.

**Fix 1 — Token refresh before posting (QBO.js):** `refreshOAuthToken()` already existed; added a call to it as the first line of `postQBODate()` so the access token is refreshed before every post run, not just manually via `qbo_refresh`. Verified: token refresh confirmed live, `postQBODate` still runs normally with the call in place.

**Fix 2 — Email alert on posting failure (QBO.js, appsscript.json):** Added an alert block at the end of `postQBODate()` — emails `ceo.gmiexperience@gmail.com` whenever `errors.length > 0 || posted === 0` (after the staged-rows loop runs; the early "nothing staged" return path does not trigger it, by design — that's a no-op day, not a failure). Added the `https://www.googleapis.com/auth/script.send_mail` OAuth scope (note: the originally-specified `gmail.send` scope would NOT have worked — that's for the separate Gmail API, not `MailApp`). Required a one-time manual re-authorization in the Apps Script editor (same as any new scope). Verified live with a temporary test endpoint, then removed it.

**Fix 3 — Sandbox guard for production cutover (Config.js, QBO.js):** Added `CFG.SANDBOX_REALM_ID = '9341457328235620'` (the sandbox company's Realm ID). `postQBODate()` now blocks with a clear error if `CFG.SANDBOX_MODE` and the connected QBO company's realm ID disagree — prevents accidentally posting to the sandbox in "production mode" or to a real company while still flagged as sandbox. Verified both directions live (blocked when flag/realm mismatched, normal when matched), then confirmed the flag was left back at `true`.

**Fix 4 — Flush + delay between staging and posting in morningRun (Helpers.js):** `morningRun()` now does `buildQBOStagingForYesterday()` → `SpreadsheetApp.flush()` → `Utilities.sleep(2000)` → `postQBODate()`, closing a race window where posting could read stale/partial staging data. Verified by running `morningRun` live — staging rows were built and flushed before posting read them, in the correct order.

**Fix 5 — `findQBOStagingRow` Date+Affiliate matching:** This was already fixed correctly in an earlier session (2026-06-23/24, see above) — it already matches by Date+Affiliate, not Period+Affiliate. No code change made. Re-verified live on 2026-06-19 (8 affiliates staged same date): each affiliate's own `QBO Invoice ID`/`QBO Customer ID`/`QBO Posted` was written to its own row only — no cross-row mixing.

**Current live deployment:** `https://script.google.com/macros/s/AKfycbxnEEtwt_cysahiCnd1PQvXJdmaq19obsvs5EDJIm8DvJv9PjLFmCFwnXLhoT_qcB2yHA/exec` (same deployment ID throughout this session, now @108).

**Reminders:**
- `CFG.SANDBOX_MODE = true` is intentional for ongoing sandbox testing — flip to `false` only once a real production QBO company is connected (Fix 3's guard will now block accidental cross-wiring of the flag and the connected realm).
- Token refresh now runs automatically at the start of every `postQBODate()` call — no more manual `qbo_refresh` needed before posting.
- The pre-existing IVA TaxCode sandbox bug (documented 2026-06-24 above) is still unresolved and unrelated to these fixes — affects only IVA-taxed affiliates (e.g. Mia, Intrepid Travel) on this sandbox company.

### Session log (2026-06-28) — Reliability rollout: 4 fixes for unattended QBO posting
**Goal:** harden the daily QBO posting pipeline so it survives token expiry, missed posting days, Apps Script timeouts, and silent morningRun failures — without human intervention. All four fixes built, deployed, and verified live.

**Fix 1 — Token expiry detection (QBO.js):**
- `refreshOAuthToken()` now detects both `No refresh token found` and `json.error` from Intuit (refresh token expired/revoked) and sends an emergency `🚨 QBO URGENT — Re-authorization Required` email — without this, the daily cron would silently post nothing for days/weeks.
- Every successful refresh now writes `QBO_TOKEN_REFRESHED_AT` (ISO timestamp) to Script Properties.
- New `checkQBOTokenAgeWarning()` runs at the top of `postQBODate()` (after a successful refresh) and sends a ONE-SHOT `⚠️ QBO Warning — Token Expiring Soon` email once when the refresh token is older than 90 days. Tracks the latch via `QBO_90D_WARNING_SENT_AT` so it doesn't spam every morning for the remaining ~10 days before expiry.
- `postQBODate()` now checks the refresh return value — if the refresh fails (dead token), it skips the row loop entirely instead of also firing the pre-existing "QBO not connected" alert on top of the re-auth email.

**Fix 2 — `auditUnpostedDays()` (QBO.js + Helpers.js):**
- New function scans the QBO Staging sheet for admin-approved (or sandbox-mode) rows that are still `QBO Posted != Y`, dated between yesterday and 7 days back, and auto-retries each missed date through the normal `postQBODate()` path (which respects the existing re-run safety in `postDayToQBO`).
- Capped at 7 days back per run (`AUDIT_MAX_DAYS_BACK = 7`) to avoid blowing the 6-min Apps Script execution limit if a 30-day backlog existed. Anything older gets a separate email for manual handling.
- Wired into `morningRun()` BEFORE `buildQBOStagingForYesterday()` — catches missed days first, then processes yesterday normally.
- Verified live: caught real stale data on first run — 2 out-of-scope rows from 2026-06-19 (out-of-scope email fired), 3 still-failing rows from 2026-06-25 (still-failing email fired; the failures are the pre-existing US-template IVA TaxCode sandbox bug, not a new issue).

**Fix 3 — Apps Script execution time guard (Helpers.js):**
- `morningRun()` now records `startTime` and logs elapsed seconds after every major phase (`buildDailySummary`, `updateBoatDocStatuses`, post-block total) for observability.
- 300-second guard wraps the QBO block: if more than 5 minutes have already elapsed by the time QBO would start, it sends `⚠️ QBO Skipped — morningRun too slow` and returns early instead of crashing silently into the 6-minute limit.

**Fix 4 — `morningHealthCheck()` safety net (QBO.js + Helpers.js + API.js):**
- New function reads the QBO Staging sheet for yesterday's rows, counts any with `QBO Posted != Y`, and if any exist: emails `🚨 QBO Health Check — Unposted invoices for <date>` then calls `postQBODate(yesterday)` to auto-retry. On a healthy day, runs silently with no email.
- New `resetMorningHealthCheckTrigger()` (Helpers.js) installs a separate 4:30am Cancun trigger (30 min after `morningRun`) — only touches `morningHealthCheck` triggers (other triggers untouched, mirrors the existing `resetMorningRunTrigger` pattern).
- Installed live 2026-06-28 via `?action=qbo_setup_health_check_trigger` (response: `"morningHealthCheck now runs daily at 4:30am"`).
- Verified end-to-end: blanked `QBO Posted` on yesterday's already-posted `Cliente Directo` row → ran `morningHealthCheck()` → alert email sent + row reposted successfully (re-run safety verified — booking PK already on invoice, no duplicate lines).

**New endpoints added to API.js (kept as permanent debug helpers, by Omri's call):**
- `?action=qbo_audit_unposted` — manual trigger for `auditUnpostedDays()`
- `?action=qbo_health_check` — manual trigger for `morningHealthCheck()`
- `?action=qbo_setup_health_check_trigger` — installs/reinstalls the 4:30am trigger
- `?action=qbo_simulate_old_token&days=N&reset_warning=1` — verification helper for the 90-day warning path; sets `QBO_TOKEN_REFRESHED_AT` to N days back. Used to verify Fix 1.
- `?action=qbo_restore_token_timestamp` — companion to the above, resets the timestamp to now and clears the warning latch.
- `?action=qbo_test_health_check_blank` — verification helper that blanks a posted row, runs `morningHealthCheck`, and restores. Used to verify Fix 4.

**Helper added:** `sendQBOAlertEmail(subject, body)` in QBO.js — single shared failure-safe email path so notification errors never crash the pipeline.

**Still pending:** the pre-existing IVA TaxCode sandbox bug still triggers the pre-existing "QBO Posting Alert" email daily for Mia/Intrepid Travel/Eco Experience Mexico (alerts whenever `posted === 0`). Omri's call 2026-06-28: live with the emails until production QBO is connected — root cause will resolve itself against a real MX QBO company.

---

## Session log (2026-06-28, continued) — Removed 8 dead QBO Staging columns

**Goal:** Omri noticed the QBO Staging sheet had a lot of columns and asked to check for unnecessary ones.

**Found:** the live sheet still had the original Phase-1 column layout (Date, Affiliate, QBO Customer ID, Invoice Lines, Total Net, Total Commission, Total IVA, Total Gross, Booking Count, QBO Invoice ID, Status, Staged At, Approved By, Approved At, Sent At, Error) with all the newer approval-chain columns appended after it by `addQBOStagingColumns()` — order never matched `Config.js`'s `COLS.QBO_STAGING` array. Confirmed this was harmless (not a bug): `appendObjectAsRow`/`getColMap`/`readSheetAsObjects` all map by header **name**, never by position.

**8 dead columns identified and removed** (zero code references, or superseded by a newer column):
- `Invoice Lines` (superseded by `Booking Lines JSON`)
- `Total Net`, `Total Gross` (superseded by the `(MXN)` versions; legacy ones were never written to)
- `Status` (frozen at `"staged"` forever — set once at creation for cell coloring only, never updated even after Admin Approved/QBO Posted; actively misleading, not just unused)
- `Approved By`, `Approved At`, `Sent At`, `Error` (zero references anywhere in the codebase)

**Done via:** `removeDeadQBOStagingColumns()` (Helpers.js) — deletes by header name (re-reads `getColMap` after each delete since indices shift), kept in the codebase as a one-time-migration-style function (same convention as `addQBOStagingColumns`/`addMissingBookingsColumns`). Temporary `qbo_cleanup_dead_columns` API endpoint added, run once, then removed.

**Verified live:** sheet now has 28 columns (was 36), matches `Config.js`'s schema exactly. Re-checked `qbo_staging_for_date` on existing 2026-06-23 data — all 7 rows still read correctly (7/7 ops/comm/admin approved, 7/7 posted, totals intact) after the column deletion.
