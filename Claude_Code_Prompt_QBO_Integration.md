# Claude Code Prompt — QBO Integration (Delayed Charges)

Read CONTEXT.md first. Then read `FareHarbor_JSON_Fields_Reference.md` in the dashboard folder.
Then read the existing `QBO.js` and `Config.js` to understand what's already built.
Do NOT start coding yet. Show me a step-by-step plan of what files you will change and what you will do in each. Wait for my approval.

---

## OVERVIEW

We are building the QuickBooks Online integration to:
1. Create Delayed Charges (Cobro Diferido) — one per affiliate per day
2. Auto-create customers and products in QBO if they don't exist
3. Record payments automatically
4. Support a 3-step approval workflow

---

## PART 1 — COMMERCIAL CORRECTION COLUMNS

Add these columns at the END of the Bookings tab (after the existing correction columns):

- `Corrected Gross` — number, blank if no correction
- `Corrected Net` — number, blank if no correction  
- `Corrected Commission` — number, blank if no correction
- `Commercial Corrected By` — text, overwritten each time
- `Commercial Corrected At` — timestamp, overwritten each time
- `Commercial Approved` — Y or blank
- `Commercial Approved By` — text
- `Commercial Approved At` — timestamp

Update the Corrections.js API endpoints to support commercial corrections:
- `save_correction` should accept Team = "Commercial" and write to the commercial correction columns
- Log all commercial corrections in the Corrections Log tab with Team = "Commercial"

Update the `bookings_for_date` API endpoint to return:
- `Effective Gross` = Corrected Gross if not blank, otherwise Gross (MXN)
- `Effective Net` = Corrected Net if not blank, otherwise Net (MXN)
- `Effective Commission` = Corrected Commission if not blank, otherwise Commission (MXN)

---

## PART 2 — APPROVAL FLAGS ON BOOKINGS

The Bookings tab already has `Authorized` (operations approval).
Now add these columns:

- `Admin Approved` — Y or blank
- `Admin Approved By` — text
- `Admin Approved At` — timestamp
- `QBO Posted` — Y or blank
- `QBO Posted At` — timestamp
- `QBO Reference` — stores the QBO delayed charge ID after posting

Rule: Admin can only approve if BOTH Operations (`Authorized` = Y) and Commercial (`Commercial Approved` = Y) are done for ALL bookings on that date.

---

## PART 3 — QBO STAGING TAB UPDATE

Update the existing QBO Staging tab to match the delayed charge structure.

Each row in staging = one delayed charge (one affiliate + one day):

| Column | Description |
|--------|-------------|
| Tour Date | The date of the tours |
| Affiliate | Affiliate name |
| Affiliate Shortname | FareHarbor shortname (stable ID) |
| QBO Customer ID | QBO ID once found/created |
| Booking Lines JSON | JSON array of all bookings: [{bookingPK, tourName, itemPK, gross, hasIVA}, ...] |
| Total Gross | Sum of all booking gross amounts |
| Total Commission | Sum of all commissions (negative) |
| Total IVA | Sum of IVA amounts |
| Total Net | What affiliate owes |
| Ops Approved | Y/blank |
| Commercial Approved | Y/blank |
| Admin Approved | Y/blank |
| QBO Posted | Y/blank |
| QBO Delayed Charge ID | QBO reference after posting |
| Payment Lines JSON | JSON array of payments: [{type, amount, bookingPK}, ...] |
| Payments Posted | Y/blank |

Build a function `buildQBOStaging(date)` that:
1. Reads all bookings for a given date from Bookings tab
2. Checks that both Ops and Commercial approved ALL bookings for that date
3. Groups bookings by affiliate
4. For each affiliate: creates one staging row with all booking lines
5. Uses Effective values (corrected if available, otherwise raw)

---

## PART 4 — QBO CUSTOMER AUTO-CREATE/FIND

Build function `findOrCreateQBOCustomer(affiliateName, affiliateShortname)`:

```
STEP 1: Query QBO for customer where CompanyName = affiliateShortname
  → Found? Return QBO Customer ID

STEP 2: Query QBO for customer where DisplayName = affiliateName
  → Found? Update CompanyName to affiliateShortname, return QBO Customer ID

STEP 3: Not found? Create new QBO customer:
  → DisplayName = affiliateName
  → CompanyName = affiliateShortname
  → Return new QBO Customer ID
```

For direct bookings: use a fixed customer called "Ventas Directas" (create if not exists, CompanyName = "direct-sales").

---

## PART 5 — QBO PRODUCT AUTO-CREATE/FIND

Build function `findOrCreateQBOProduct(tourName, itemPK)`:

```
STEP 1: Query QBO for item where Sku = itemPK (as string)
  → Found? If name differs from tourName, update name. Return QBO Item ID.

STEP 2: Query QBO for item where Name = tourName
  → Found? Update Sku to itemPK. Return QBO Item ID.

STEP 3: Not found? Create new QBO item:
  → Name = tourName
  → Sku = itemPK (as string)
  → Type = "Service"
  → IncomeAccountRef = tour revenue account (use a default, configurable in Config.js)
  → Return new QBO Item ID
```

Also ensure a "Commission" product exists in QBO (search by name, it should already be there).

---

## PART 6 — CREATE DELAYED CHARGES IN QBO

Build function `postDelayedChargeToQBO(stagingRow)`:

```
For each staging row (one affiliate, one day):

1. Find or create the QBO customer (Part 4)
2. For each booking line in Booking Lines JSON:
   a. Find or create the QBO product (Part 5)
   b. Add a line item:
      - ItemRef = QBO product ID
      - Description = "Booking #XXXXXX — Tour Date"
      - Amount = Gross revenue of that booking
      - TaxCodeRef = IVA 16% tax code if booking has IVA, otherwise exempt/none
3. Add commission line:
   - ItemRef = "Commission" product ID
   - Description = "Commission — [date]"
   - Amount = negative total commission
   - TaxCodeRef = None / Exempt
4. Create the Delayed Charge via QBO API
5. Store the returned QBO Delayed Charge ID in staging row
6. Mark QBO Posted = Y
```

Important: Verify the correct QBO API entity name for delayed charges.
It may be "Charge" — check the QBO API docs and confirm before building.

---

## PART 7 — RECORD PAYMENTS IN QBO

For bookings where Adventure Lab collected payment (cash or card):

```
1. Group payments by payment type (cash, card)
2. For each payment group:
   - Create a Payment or Sales Receipt in QBO
   - CustomerRef = the affiliate customer
   - TotalAmt = sum of payment amounts
   - PaymentMethodRef = match to QBO payment method
3. Mark Payments Posted = Y in staging
```

Note: Payments are typically applied against invoices, not delayed charges.
Since we use delayed charges (which become invoices later), check the QBO API
to confirm the best approach for recording payments collected before invoicing.
Present options before building.

---

## PART 8 — API ENDPOINTS FOR DASHBOARD

Add these endpoints to API.js:

1. `qbo_staging_for_date(date)` — returns all staging rows for a date with approval status
2. `admin_approve_day(date)` — checks ops + commercial approved, then marks admin approved
3. `post_to_qbo(date)` — posts all approved staging rows as delayed charges to QBO
4. `qbo_status(date)` — returns posting status (posted/pending/errors)

---

## PART 9 — QBO AUTH SETUP

Check if QBO OAuth2 is already configured in the existing QBO.js.
If not, set up:

1. Store QBO credentials in Script Properties (NOT in code or sheet)
2. OAuth2 token refresh logic using Google Apps Script OAuth2 library
3. Realm ID in Config.js

Do NOT hardcode any secrets. Use PropertiesService.getScriptProperties().

---

## IVA HANDLING — CRITICAL

IVA must use QBO's native tax system:
- Apply TaxCodeRef to each line item (IVA 16% or Exempt)
- Do NOT create a separate line item for IVA
- QBO calculates the tax automatically
- The IVA 16% tax code already exists in the QBO account
- Find it by querying TaxCode entities and store the ID in Config.js

---

## IDENTIFIER MATCHING — CRITICAL

Do NOT match by name. Use stable identifiers:

- Affiliates: FareHarbor `shortname` → stored in QBO `CompanyName` field
- Tours: FareHarbor `item.pk` → stored in QBO `Sku` field
- If name changed in FareHarbor → update name in QBO, don't create duplicate

---

## WHAT NOT TO CHANGE
- Do not change the webhook processing (Webhook.js)
- Do not change the existing revenue calculation logic
- Do not change the operations correction/authorization system (only extend it)
- Do not build any dashboard UI — that comes later

## HOW TO WORK
1. Show me the full plan first — what files you will change and what you will do in each
2. Wait for my approval
3. Work one part at a time (Part 1, then Part 2, etc.)
4. After each part: tell me what you did and what to test
5. At the end: update CONTEXT.md and push everything to GitHub and clasp push
