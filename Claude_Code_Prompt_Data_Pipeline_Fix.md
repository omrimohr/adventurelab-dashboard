# Claude Code Prompt — Adventure Lab Data Pipeline Fix

Read CONTEXT.md first. Then read this full plan before touching anything.
Do NOT start coding yet. Show me a step-by-step plan of what files you will change and what you will do in each. Wait for my approval.

Also read the file `FareHarbor_JSON_Fields_Reference.md` in the dashboard folder — it contains the complete field mapping rules, all tested and confirmed.

---

## PART 1 — FIX REVENUE COLUMNS ON BOOKINGS TAB

The current webhook processing script is populating Revenue, Commission, Tax, and Gross columns incorrectly. Replace the money calculation logic with these confirmed rules:

### Step 1 — Status check
```
IF status = "cancelled" or "rebooked" → exclude from revenue calculations
```

### Step 2 — Flags
```
IF payments = [] AND invoice_price = 0
→ Flag column = "NEEDS REVIEW"
→ Gross = 0, Net = 0, Commission = 0, IVA = 0
→ Store booking.note in Notes column

IF payments = [] AND invoice_price > 0
→ Flag column = "UNPAID"
→ Proceed to money calculation
```

### Step 3 — Money calculation (all active bookings)
```
Gross Revenue = booking.receipt_total / 100
                IF receipt_total = 0 → use booking.invoice_price / 100

Net Revenue   = booking.invoice_price / 100
                IF invoice_price = 0 (direct booking) → use booking.receipt_total / 100

IVA           = sum of booking.customers[*].invoice_cost.tax / 100

Commission    = Gross - Net - IVA

Discount      = sum of booking.custom_field_values[*].total_cost.total / 100
                WHERE total_cost.total < 0
```

### Step 4 — Historical data fix (bookings created before June 8, 2026 14:00 local = 2026-06-08T19:00:00Z)

For bookings created BEFORE the cutoff, some affiliates were configured as "Referral" type in FareHarbor. For those affiliates, invoice_price = Commission (not Net). The rule is INVERTED:

```
REFERRAL_AFFILIATES = [
  "Adventure Lab - Affiliate",
  "Angel Martinez",
  "Bacalar Top Experiences",
  "Bacalar's Xperience",
  "Bakance",
  "Casa Hormiga Hotel",
  "Circulo Bacalar",
  "Destino Bacalar",
  "Kolibri Belize",
  "Makaabá Eco-Boutique",
  "Mayan Playa",
  "Pura Vida",
  "Tregua Bacalar"
]

IF affiliate_company.name IN REFERRAL_AFFILIATES
AND booking.created_at < "2026-06-08T19:00:00Z"
→ invoice_price = Commission owed TO affiliate (not Net)
→ Net = Gross - Commission - IVA
→ Commission = invoice_price / 100
```

Store the REFERRAL_AFFILIATES list in Config.js so it can be updated easily.

### Step 5 — OTA exception (GetYourGuide, Viator)
```
IF invoice_price = 0 AND affiliate exists AND affiliate_collected > 0
→ Treat receipt_total as Net (OTA already deducted their commission)
→ Commission = unknown, set to 0
→ Flag column = "OTA - No Invoice"
```

### Column mapping on Bookings tab
Make sure these columns exist and are populated correctly:
- `Gross (MXN)` — Gross Revenue
- `Net (MXN)` — Net Revenue (was previously called "Revenue (MXN)" — rename if needed)
- `Commission (MXN)` — Commission
- `Tax (MXN)` — IVA
- `Discount (MXN)` — NEW column, add if not exists
- `Flag` — NEW column: "NEEDS REVIEW", "UNPAID", "OTA - No Invoice", or blank

---

## PART 2 — FIX PAX IN BOOKINGS TAB

Replace current pax calculation with:
```
Pax = booking.customer_count
```
That's it. Do not calculate from custom fields or checkin status. Just use customer_count directly.

---

## PART 3 — FIX TOURS AGGREGATION SCRIPT

The Tours tab aggregates bookings into tour slots. Fix the aggregation:

### Pax
```
Tour Pax = sum of customer_count from all bookings in that tour
```
Remove any logic that tries to calculate pax from custom fields.

### Revenue
```
Tour Gross = sum of Gross from all bookings in that tour
Tour Net = sum of Net from all bookings in that tour
Tour Commission = sum of Commission from all bookings in that tour
Tour IVA = sum of IVA from all bookings in that tour
```

### Custom Fields
Add a new column to Tours tab: `Custom Fields`
```
Custom Fields = concatenated raw custom field text from all bookings in that tour
Format: "FieldName: Value | FieldName: Value"
Separated by " || " between different bookings
```

---

## PART 4 — ADD CORRECTION COLUMNS TO BOOKINGS TAB

Add these columns at the END of the Bookings tab. Do not touch any existing columns.

- `Corrected Pax` — number, blank if no correction
- `Last Corrected By` — text, overwritten each time
- `Last Corrected At` — timestamp, overwritten each time
- `Authorized` — Y or blank
- `Authorized By` — text
- `Authorized At` — timestamp

---

## PART 5 — CREATE CORRECTIONS LOG TAB

Create a new tab called "Corrections Log" with these columns:

| Booking ID | Tour ID | Field Changed | Original Value | Corrected Value | Reason | Changed By | Team | Timestamp |

- Team values: "Coordinator" or "Commercial"
- This tab is append-only — never delete rows
- When a correction is saved from the dashboard, write one row per field changed

---

## PART 6 — ADD API ENDPOINTS FOR CORRECTIONS

Add endpoints to API.js so the dashboard can:
1. Read bookings for a specific date (with corrected values if they exist)
2. Save a correction (write to Corrected columns + append to Corrections Log)
3. Authorize a day (write Authorized=Y to all bookings for that date)

The dashboard reads: `Corrected Pax` if not blank, otherwise raw `Pax`. Same pattern for any correctable field.

---

## WHAT NOT TO CHANGE
- Do not touch raw FareHarbor columns that already exist (only ADD new columns)
- Do not change the webhook receiving logic (how data arrives) — only change how it's PROCESSED
- Do not build any dashboard UI changes — that comes later

## HOW TO WORK
1. Show me the full plan first — what files you will change and what you will do in each
2. Wait for my approval
3. Work one part at a time (Part 1, then Part 2, etc.)
4. After each part: tell me what you did and what to test
5. At the end: update CONTEXT.md and push everything to GitHub and clasp push
