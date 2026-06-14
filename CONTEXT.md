# Adventure Lab — Claude Code Workspace Context

Read this file at the start of every session before doing anything.

---

## Who I Am

**Owner:** Omri Mohr  
**Business:** Adventure Lab — tour operation based in Bacalar, Mexico  
**Google account:** ceo.gmiexperience@gmail.com

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
- [ ] Owner commercial dashboard — in progress
- [ ] QBO Phase 2 (admin approval screen) — pending
- [ ] QBO Phase 3 (send to QBO) — pending
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
