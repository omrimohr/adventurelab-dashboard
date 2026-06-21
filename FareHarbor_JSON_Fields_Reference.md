# FareHarbor Webhook JSON — Complete Field Reference
Adventure Lab | Generated June 2026

All amounts are in **centavos** (divide by 100 to get MXN). Fields marked ⭐ are the ones we use for the sheet.

---

## TOP LEVEL — Booking Identity

| Field | Interpretation |
|---|---|
| `booking.pk` | FareHarbor booking ID (e.g. 353504173) — our primary key ⭐ |
| `booking.uuid` | FareHarbor UUID — used as Raw Key in sheet ⭐ |
| `booking.display_id` | Human-readable ID shown in FareHarbor dashboard (e.g. "#353504173") |
| `booking.status` | Booking status: "booked", "cancelled", "rebooked" ⭐ |
| `booking.created_at` | When the booking was created (UTC timestamp) |
| `booking.source_type` | How it was booked: "direct", "affiliate", "online" |
| `booking.external_id` | External system ID — usually empty |
| `booking.external_api_url` | URL to fetch this booking via FareHarbor API |
| `booking.dashboard_url` | URL to view booking in FareHarbor dashboard |
| `booking.confirmation_url` | URL sent to customer for confirmation |
| `booking.voucher_number` | Voucher number if used — usually empty |
| `booking.rebooked_from` | UUID of original booking if this is a rebook |
| `booking.rebooked_to` | UUID of new booking if this was rebooked to another |
| `booking.order` | Order grouping — usually null |
| `booking.note` | Internal notes added in FareHarbor ⭐ |
| `booking.note_safe_html` | Same as note but HTML formatted |
| `booking.customer_count` | Number of customers in this booking ⭐ |
| `booking.is_eligible_for_cancellation` | Whether booking can still be cancelled |
| `booking.is_subscribed_for_sms_updates` | Customer SMS preference |
| `booking.is_follow_up_email_disabled` | Whether follow-up email is suppressed |
| `booking.is_reminder_email_disabled` | Whether reminder email is suppressed |
| `booking.pickup` | Pickup location — usually null |
| `booking.arrival` | Arrival info — usually null |
| `booking.lodging` | Hotel/lodging info — usually null |
| `booking.desk` | Desk assignment — usually null |

---

## TOP LEVEL — Money Fields ⭐

| Field | Interpretation |
|---|---|
| `booking.receipt_subtotal` | Gross amount before taxes, in centavos. What client is charged before tax. |
| `booking.receipt_subtotal_display` | Same as above, formatted as string (e.g. "1402.50") |
| `booking.receipt_taxes` | IVA/tax amount charged to client, in centavos ⭐ |
| `booking.receipt_taxes_display` | Same as above, formatted as string |
| `booking.receipt_total` | Total amount client pays = subtotal + taxes, in centavos ⭐ **→ our GROSS** |
| `booking.receipt_total_display` | Same as above, formatted as string |
| `booking.amount_paid` | How much has actually been collected so far, in centavos |
| `booking.amount_paid_display` | Same as above, formatted |
| `booking.invoice_price` | What Adventure Lab invoices to the affiliate (net + IVA for fixed-rate affiliates like Habitas/Amainah), in centavos ⭐ **→ our NET for affiliate bookings** |
| `booking.invoice_price_display` | Same as above, formatted |

---

## TOP LEVEL — Company & Affiliate

| Field | Interpretation |
|---|---|
| `booking.company.name` | Always "Adventure Lab" |
| `booking.company.shortname` | Always "thegaiaexperience" |
| `booking.company.currency` | Always "mxn" |
| `booking.affiliate_company` | Null if direct booking, otherwise object with affiliate info ⭐ |
| `booking.affiliate_company.name` | Affiliate name (e.g. "The Yak Lake House", "Habitas") ⭐ |
| `booking.affiliate_company.shortname` | Affiliate short identifier |
| `booking.affiliate_company.currency` | Affiliate's currency |

---

## TOP LEVEL — Contact (Customer)

| Field | Interpretation |
|---|---|
| `booking.contact.name` | Customer full name ⭐ |
| `booking.contact.email` | Customer email ⭐ |
| `booking.contact.phone` | Customer phone number |
| `booking.contact.normalized_phone` | Phone in international format |
| `booking.contact.phone_country` | Country code of phone |
| `booking.contact.language` | Customer's language preference |
| `booking.contact.is_subscribed_for_email_updates` | Email marketing preference |

---

## TOP LEVEL — Booked By & Agent

| Field | Interpretation |
|---|---|
| `booking.booked_by.name` | Username of who created the booking in FareHarbor (e.g. "theyak", "habitas") ⭐ |
| `booking.agent` | Null if no agent, otherwise object |
| `booking.agent.name` | Agent name (e.g. "Dario Novelo") ⭐ |
| `booking.agent.pk` | Agent ID |

---

## AVAILABILITY — Tour Slot Info

| Field | Interpretation |
|---|---|
| `booking.availability.pk` | FareHarbor availability ID (unique tour slot) ⭐ |
| `booking.availability.start_at` | Tour start datetime with timezone ⭐ |
| `booking.availability.end_at` | Tour end datetime with timezone ⭐ |
| `booking.availability.capacity` | Max capacity of this slot |
| `booking.availability.minimum_party_size` | Minimum people required |
| `booking.availability.maximum_party_size` | Maximum people allowed |
| `booking.availability.online_booking_status` | Whether online booking is open/closed |
| `booking.availability.headline` | Short description of the slot |
| `booking.availability.item.pk` | FareHarbor item (product) ID ⭐ |
| `booking.availability.item.name` | Tour name (e.g. "Sailing Tour", "Private Catamaran") ⭐ |
| `booking.availability.item.headline` | Tour tagline |
| `booking.availability.item.primary_location` | Location object (null for most tours) |
| `booking.availability.item.primary_location.address.city` | City |
| `booking.availability.item.primary_location.address.country` | Country |
| `booking.availability.item.primary_location.latitude` | GPS latitude |
| `booking.availability.item.primary_location.longitude` | GPS longitude |

---

## AVAILABILITY — Customer Type Rates (Pricing Catalog)

These are the price list for the slot — not the actual booking amounts.

| Field | Interpretation |
|---|---|
| `booking.availability.customer_type_rates[*].pk` | Rate ID |
| `booking.availability.customer_type_rates[*].total` | List price per customer type in centavos (before tax) |
| `booking.availability.customer_type_rates[*].total_including_tax` | List price including tax |
| `booking.availability.customer_type_rates[*].capacity` | Capacity for this customer type |
| `booking.availability.customer_type_rates[*].minimum_party_size` | Min for this type |
| `booking.availability.customer_type_rates[*].maximum_party_size` | Max for this type |
| `booking.availability.customer_type_rates[*].customer_type.pk` | Customer type ID |
| `booking.availability.customer_type_rates[*].customer_type.singular` | Name singular (e.g. "Person") |
| `booking.availability.customer_type_rates[*].customer_type.plural` | Name plural (e.g. "People") |
| `booking.availability.customer_type_rates[*].customer_prototype.pk` | Prototype ID |
| `booking.availability.customer_type_rates[*].customer_prototype.display_name` | Display label (e.g. "Person", "Barracuda Catamaran") |
| `booking.availability.customer_type_rates[*].customer_prototype.total` | Prototype price in centavos |
| `booking.availability.customer_type_rates[*].customer_prototype.total_including_tax` | Prototype price with tax |

---

## AVAILABILITY — Custom Field Instances (Add-ons/Fees at Slot Level)

These define what add-ons, fees, and options are available for this slot.

| Field | Interpretation |
|---|---|
| `booking.availability.custom_field_instances[*].pk` | Instance ID |
| `booking.availability.custom_field_instances[*].custom_field.pk` | Field definition ID |
| `booking.availability.custom_field_instances[*].custom_field.name` | Field name (e.g. "Card processing fee (4%)", "Desayuno adventure") |
| `booking.availability.custom_field_instances[*].custom_field.title` | Display title |
| `booking.availability.custom_field_instances[*].custom_field.type` | Input type: "yes-no", "count", "text", "select" |
| `booking.availability.custom_field_instances[*].custom_field.modifier_kind` | How it modifies price: "offset" (fixed amount) or "percentage" |
| `booking.availability.custom_field_instances[*].custom_field.modifier_type` | "adjust" (adds to price) or "none" (informational only) |
| `booking.availability.custom_field_instances[*].custom_field.offset` | Fixed amount modifier in centavos |
| `booking.availability.custom_field_instances[*].custom_field.percentage` | Percentage modifier (e.g. 4.0 = 4%) |
| `booking.availability.custom_field_instances[*].custom_field.is_taxable` | Whether this fee is subject to IVA |
| `booking.availability.custom_field_instances[*].custom_field.is_required` | Whether customer must answer this field |
| `booking.availability.custom_field_instances[*].custom_field.total` | Total amount this field adds at booking level |
| `booking.availability.custom_field_instances[*].custom_field.total_including_tax` | Same including tax |
| `booking.availability.custom_field_instances[*].custom_field.extended_options[*].name` | Option name (e.g. "SI", "NO") |
| `booking.availability.custom_field_instances[*].custom_field.extended_options[*].offset` | Amount this option adds in centavos |
| `booking.availability.custom_field_instances[*].custom_field.extended_options[*].percentage` | Percentage this option adds |
| `booking.availability.custom_field_instances[*].custom_field.extended_options[*].is_taxable` | Whether this option is taxable |

---

## AVAILABILITY — Crew Members (Assigned to the Slot)

| Field | Interpretation |
|---|---|
| `booking.availability.crew_members[*].pk` | Assignment ID |
| `booking.availability.crew_members[*].user.name` | Crew member full name ⭐ |
| `booking.availability.crew_members[*].user.username` | Crew member username |
| `booking.availability.crew_members[*].role.pk` | Role ID |
| `booking.availability.crew_members[*].role.short_name` | Role name (e.g. "Capitan", "Marinero") ⭐ |
| `booking.availability.crew_members[*].note` | Note for this crew assignment |

---

## CUSTOMERS — Per-Person Detail ⭐

One entry per customer in the booking. This is where actual per-person money lives.

| Field | Interpretation |
|---|---|
| `booking.customers[*].pk` | Customer record ID |
| `booking.customers[*].checkin_status` | Null if not checked in, object if checked in ⭐ |
| `booking.customers[*].checkin_status.name` | Status name (e.g. "Checked In") |
| `booking.customers[*].checkin_status.type` | Status type code |
| `booking.customers[*].checkin_url` | URL to check in this customer |
| `booking.customers[*].customer_type_rate.pk` | Which rate this customer is on |
| `booking.customers[*].customer_type_rate.total` | List price for this customer in centavos |
| `booking.customers[*].customer_type_rate.total_including_tax` | List price with tax |
| `booking.customers[*].customer_type_rate.customer_type.singular` | Customer type label (e.g. "Person") ⭐ |
| `booking.customers[*].customer_type_rate.customer_prototype.display_name` | Display name for this rate ⭐ |
| `booking.customers[*].total_cost.price` | Gross price for this customer in centavos (before tax) |
| `booking.customers[*].total_cost.tax` | Tax on this customer's price |
| `booking.customers[*].total_cost.taxable` | Taxable base amount |
| `booking.customers[*].total_cost.feeable` | Amount subject to fees |
| `booking.customers[*].total_cost.total` | Total gross for this customer ⭐ **→ gross per person** |
| `booking.customers[*].invoice_cost.price` | Net price for this customer (what Adventure Lab keeps, before tax) ⭐ **→ net per person** |
| `booking.customers[*].invoice_cost.tax` | IVA on this customer's net price ⭐ **→ tax per person** |
| `booking.customers[*].invoice_cost.taxable` | Taxable base for invoice |
| `booking.customers[*].invoice_cost.feeable` | Feeable base for invoice |
| `booking.customers[*].invoice_cost.total` | Net + IVA for this customer ⭐ |
| `booking.customers[*].invoice_cost.tax_by_type.131449` | IVA breakdown by tax type ID |
| `booking.customers[*].invoice_cost.tax_by_type.21914` | IVA breakdown by tax type ID (16%) |

---

## CUSTOMERS — Per-Person Custom Fields

Same structure as availability custom fields but with actual values chosen.

| Field | Interpretation |
|---|---|
| `booking.customers[*].custom_field_values[*].pk` | Value record ID |
| `booking.customers[*].custom_field_values[*].name` | Field name |
| `booking.customers[*].custom_field_values[*].value` | What the customer selected/entered |
| `booking.customers[*].custom_field_values[*].display_value` | Human-readable value |
| `booking.customers[*].custom_field_values[*].total_cost.price` | Price impact of this field choice in centavos |
| `booking.customers[*].custom_field_values[*].total_cost.tax` | Tax on this field's price |
| `booking.customers[*].custom_field_values[*].total_cost.total` | Total impact of this field |
| `booking.customers[*].custom_field_values[*].custom_field.name` | Field definition name |
| `booking.customers[*].custom_field_values[*].custom_field.modifier_kind` | "offset" or "percentage" |
| `booking.customers[*].custom_field_values[*].custom_field.modifier_type` | "adjust" or "none" |
| `booking.customers[*].custom_field_values[*].custom_field.offset` | Fixed amount modifier |
| `booking.customers[*].custom_field_values[*].custom_field.percentage` | Percentage modifier |
| `booking.customers[*].custom_field_values[*].custom_field.is_taxable` | Whether taxable |

---

## BOOKING-LEVEL CUSTOM FIELD VALUES ⭐ (Add-ons, Discounts, Fees)

Same as customer-level but applied to the whole booking.

| Field | Interpretation |
|---|---|
| `booking.custom_field_values[*].pk` | Value record ID |
| `booking.custom_field_values[*].name` | Field name |
| `booking.custom_field_values[*].value` | Value entered (e.g. "Caseta10" for promo code) ⭐ |
| `booking.custom_field_values[*].display_value` | Human-readable version |
| `booking.custom_field_values[*].total_cost.price` | Price impact in centavos — **NEGATIVE = DISCOUNT** ⭐ |
| `booking.custom_field_values[*].total_cost.tax` | Tax on this field |
| `booking.custom_field_values[*].total_cost.taxable` | Taxable base |
| `booking.custom_field_values[*].total_cost.total` | Total impact including tax — negative if discount ⭐ |
| `booking.custom_field_values[*].custom_field.name` | Field name (e.g. "Promo Last minute", "Card processing fee (4%)") ⭐ |
| `booking.custom_field_values[*].custom_field.modifier_kind` | "offset" or "percentage" |
| `booking.custom_field_values[*].custom_field.modifier_type` | "adjust" = affects price, "none" = informational only |
| `booking.custom_field_values[*].custom_field.offset` | Fixed amount in centavos |
| `booking.custom_field_values[*].custom_field.percentage` | Percentage (e.g. 4.0 = 4%) |
| `booking.custom_field_values[*].custom_field.is_taxable` | Whether this affects taxable base |
| `booking.custom_field_values[*].custom_field.is_always_per_customer` | Whether applied per person or per booking |
| `booking.custom_field_values[*].custom_field.extended_options[*].name` | Option name |
| `booking.custom_field_values[*].custom_field.extended_options[*].offset` | Option amount in centavos |

---

## PAYMENTS ⭐

One entry per payment transaction. A booking can have multiple payments.

| Field | Interpretation |
|---|---|
| `booking.payments[*].pk` | Payment transaction ID |
| `booking.payments[*].created_at` | When payment was made |
| `booking.payments[*].type` | Payment method: "in-store", "affiliate", "card", "online" ⭐ |
| `booking.payments[*].in_store_payment_type` | Null if online, otherwise object |
| `booking.payments[*].in_store_payment_type.name` | e.g. "Cash", "Card terminal" |
| `booking.payments[*].in_store_payment_type.pk` | Payment type ID |
| `booking.payments[*].status` | "succeeded", "pending", "failed" ⭐ |
| `booking.payments[*].currency` | Always "mxn" |
| `booking.payments[*].initial_amount_paid` | Original amount charged in centavos ⭐ |
| `booking.payments[*].initial_amount_paid_display` | Same formatted as string |
| `booking.payments[*].amount_paid` | Current amount (may differ if partially refunded) ⭐ |
| `booking.payments[*].amount_paid_display` | Same formatted |
| `booking.payments[*].refunds[*].pk` | Refund ID |
| `booking.payments[*].refunds[*].amount_refunded` | Amount refunded in centavos ⭐ |
| `booking.payments[*].refunds[*].amount_refunded_display` | Same formatted |
| `booking.payments[*].refunds[*].created_at` | When refund was issued |
| `booking.payments[*].refunds[*].is_cancelled` | Whether refund was cancelled |

---

## RESOURCE USES

Which physical resources (boats, kayaks, SUPs) are assigned to this booking.

| Field | Interpretation |
|---|---|
| `booking.resource_uses[*].pk` | Resource use ID |
| `booking.resource_uses[*].resource.pk` | Resource ID |
| `booking.resource_uses[*].resource.name` | Resource name (e.g. "Espacios en Catamarán Huracan") ⭐ |
| `booking.resource_uses[*].resource.short_name` | Short name |
| `booking.resource_uses[*].customer_pk` | Which customer this resource is assigned to |
| `booking.resource_uses[*].start_at` | Resource use start time |
| `booking.resource_uses[*].end_at` | Resource use end time |
| `booking.resource_uses[*].use_count` | How many units of this resource used |

---

## CANCELLATION POLICY

| Field | Interpretation |
|---|---|
| `booking.effective_cancellation_policy.type` | Policy type (e.g. "hours-before-start") |
| `booking.effective_cancellation_policy.cutoff` | Hours before start that free cancellation ends |

---

## SUMMARY — Fields We Use in the Sheet

| Sheet Column | Source Field | Notes |
|---|---|---|
| Booking PK | `booking.pk` | Primary key |
| UUID | `booking.uuid` | |
| Status | `booking.status` | booked / cancelled / rebooked |
| Webhook Received | timestamp of webhook arrival | |
| Tour Date | `booking.availability.start_at` | date part |
| Tour Time | `booking.availability.start_at` | time part |
| Availability PK | `booking.availability.pk` | used to group bookings into tours |
| Item PK | `booking.availability.item.pk` | |
| Tour Name | `booking.availability.item.name` | |
| Customer Count | `booking.customer_count` | number of customer records |
| Pax | sum of checked-in customers | count of `customers[*].checkin_status` not null |
| Checked In | same as pax | |
| Customer Types | `customers[*].customer_type_rate.customer_prototype.display_name` | concatenated |
| Gross (MXN) | `booking.receipt_total` / 100 | ⭐ what client paid |
| Tax (MXN) | `booking.receipt_taxes` / 100 OR sum of `customers[*].invoice_cost.tax` / 100 | ⭐ IVA |
| Net (MXN) | sum of `customers[*].invoice_cost.price` / 100 | ⭐ Adventure Lab keeps |
| Commission (MXN) | Gross - Net - Tax | ⭐ calculated |
| Discount (MXN) | sum of negative `custom_field_values[*].total_cost.total` / 100 | ⭐ promos/discounts |
| Affiliate | `booking.affiliate_company.name` | null = Direct |
| Agent | `booking.agent.name` | |
| Contact Name | `booking.contact.name` | |
| Contact Email | `booking.contact.email` | |
| Payment Type | `booking.payments[*].type` | concatenated |
| Payment Status | `booking.payments[*].status` | |
| Resources | `resource_uses[*].resource.name` | concatenated |
| Crew | `availability.crew_members[*].user.name + role` | concatenated |
| Notes | `booking.note` | |
| Custom Fields | `custom_field_values[*].name + value` | concatenated — includes pax info for private tours |
| Collected By | derived from payment type + affiliate | |

---

## FIELD MAPPING RULES (Agreed June 2026)

### Money Field Logic

```
IF receipt_total > 0:
    Gross = receipt_total / 100
ELSE IF receipt_total == 0 AND invoice_price > 0:
    Gross = invoice_price / 100   ← affiliate collects from client

Net = invoice_price / 100
    EXCEPTION: IF invoice_price == 0 AND affiliate_company == null:
        Net = receipt_total / 100  ← direct booking, Adventure Lab keeps all

Tax = sum of customers[*].invoice_cost.tax / 100
    (only field that separates IVA correctly for Habitas/fixed-rate affiliates)

Commission = Gross - Net - Tax  ← always calculated, never a raw field

Discount = sum of custom_field_values[*].total_cost.total / 100
    WHERE total_cost.total < 0
    (negative values = promos, last-minute discounts, etc.)
```

### Pax Logic

```
Pax = count of customers[*] WHERE checkin_status IS NOT null
    (checked-in count is the most reliable)

Fallback: customer_count
    (use if tour hasn't happened yet or check-in wasn't recorded)

For private tours: actual group size is in custom_field_values[*].value
    WHERE field name contains "personas" or "people" or "cantidad"
    → stored in Custom Fields column for coordinator to read and correct manually
```

### Known Edge Cases

| Case | Symptom | Solution |
|---|---|---|
| Habitas / fixed-rate affiliate | receipt_total=0, invoice_price=full amount | Use invoice_price as Gross |
| Direct booking | invoice_price=0 | Use receipt_total as Net |
| GetYourGuide / Viator | invoice_price=0 | Use receipt_total as Net (they pay net separately) |
| Card processing fee (4%) | Included in invoice_price but not in customers sum | Always use invoice_price for Net, not customers sum |
| Promo / discount | Negative custom_field_values entry | Capture in Discount column, reflected in receipt_total |
| Cancelled booking | Status=cancelled, amounts may be 0 | Store as-is, flag in dashboard |
| Rebooked | Status=rebooked, original has rebooked_to UUID | Link to new booking, exclude from revenue |

### Zero Income Flag

```
IF booking.payments = [] (empty array)
→ Zero Income = Y
→ Gross = 0, Net = 0, Tax = 0, Commission = 0
→ Show booking.note in dashboard next to the flag
→ Store sum of customers[*].total_cost.total / 100 as "List Price (MXN)" for reference
```

This correctly distinguishes from affiliate-collects bookings (Habitas, Amainah)
which always have payments[0].type = "affiliate" with status "succeeded".

### Unpaid Flag (updated Zero Income logic)

```
IF payments = [] AND receipt_total = 0
→ Zero Income = Y (courtesy/gift — no income expected)
→ Gross = 0, Net = 0, Tax = 0, Commission = 0
→ Show booking.note in dashboard

IF payments = [] AND receipt_total > 0
→ Unpaid = Y (payment due but not yet collected)
→ Gross = receipt_total / 100
→ Net = invoice_price / 100
→ Commission = Gross - Net - Tax
→ Flag in dashboard so coordinator knows to follow up
```

### Rebooked Status

```
IF status = "rebooked"
→ Exclude from all revenue, commission, pax counts
→ Still store in sheet for reference/audit trail
→ Flag as "Rebooked" in dashboard — show rebooked_to UUID if available
→ Gross = 0, Net = 0, Tax = 0, Commission = 0
```

The new booking (rebooked_to) is where the actual revenue lives.

---

## CONFIRMED FIELD MAPPING (Tested & Verified June 2026)

10 bookings tested and confirmed correct across all booking types.

### Status Rules (apply first, in this order)

```
1. IF status = "cancelled"
   → Exclude from revenue. Store for audit only.

2. IF status = "rebooked"
   → Exclude from revenue. Store for audit only.
   → The new booking (rebooked_to UUID) is where revenue lives.

3. IF status = "booked" AND payments = [] AND receipt_total = 0
   → Zero Income = Y (courtesy/gift)
   → Gross = 0, Net = 0, Tax = 0, Commission = 0
   → Show booking.note as reason in dashboard

4. IF status = "booked" AND payments = [] AND receipt_total > 0
   → Unpaid = Y (payment due, not yet collected)
   → Gross = receipt_total / 100
   → Net = invoice_price / 100
   → Commission = Gross - Net - Tax
   → Flag in dashboard for follow-up

5. IF status = "booked" AND payments not empty
   → Normal booking — apply money mapping below
```

### Money Mapping (normal bookings)

```
Gross   = receipt_total / 100
          EXCEPT if receipt_total = 0 and invoice_price > 0:
          Gross = invoice_price / 100  (affiliate collects from client)

Net     = invoice_price / 100
          EXCEPT if invoice_price = 0 (direct booking):
          Net = receipt_total / 100

Tax     = sum of customers[*].invoice_cost.tax / 100

Commission = Gross - Net - Tax  (always calculated)

Discount   = sum of custom_field_values[*].total_cost.total / 100
             WHERE total_cost.total < 0

List Price = sum of customers[*].total_cost.total / 100
             (stored for reference on Zero Income bookings)
```

### Pax Mapping

```
Pax = count of customers[*] WHERE checkin_status IS NOT null
Fallback = customer_count (if tour hasn't happened yet)
Private tour actual pax = read from custom_field_values[*].value
  WHERE field name contains "personas" or "people" or "cantidad"
  → stored in Custom Fields column, coordinator corrects manually
```

### Invoice Direction Rule (who collected determines Net vs Commission)

```
affiliate_collected = sum of payments[*].amount_paid WHERE type = "affiliate"
adv_collected       = sum of payments[*].amount_paid WHERE type IN ["in-store", "card"]

CASE 1: Affiliate collected everything (affiliate_collected > 0, adv_collected = 0)
→ invoice_price = Net to Adventure Lab
→ Commission = Gross - Net - Tax

CASE 2: Adventure Lab collected everything (adv_collected > 0, affiliate_collected = 0)
→ invoice_price = Commission owed TO affiliate
→ Net = Gross - Commission - Tax

CASE 3: Split collection (both > 0)
→ ADV LAB portion = adv_collected
→ Affiliate portion = affiliate_collected
→ Commission = calculated on affiliate rate against full Gross
→ Net = Gross - Commission - Tax
→ Balance = what each party still owes the other

NOTE: A payment with type="affiliate" but amount_paid=0 is ignored —
it means the affiliate payment was recorded but nothing was actually collected.
```

### GetYourGuide & Viator (OTA) — Known Configuration Issue

```
Current state:
- receipt_total = what client paid (already net — GYG/Viator deducted their commission before paying)
- invoice_price = $0 (NOT CONFIGURED in FareHarbor — missing invoice price sheet)
- aff_collected = receipt_total (affiliate records the payment)

What this means:
- The amount in receipt_total IS the net to Adventure Lab (commission already deducted by OTA)
- No invoice is generated because price sheet is not configured

TEMPORARY mapping (until fixed):
- Gross = receipt_total (treat as net, best we can do)
- Net = receipt_total
- Commission = unknown (not in FareHarbor)
- Flag = "OTA - No Invoice" so commercial team knows

ACTION REQUIRED:
→ Valentin and Angel need to configure invoice price sheets for:
  - GetYourGuide - MXN - API
  - TripAdvisor Experiences/Viator - MXN - API
  This will allow FareHarbor to generate proper invoices and populate invoice_price correctly.
```

### FINAL RULE — Zero Income / Needs Review

```
IF payments = [] AND invoice_price = 0
→ Flag = "NEEDS REVIEW"
→ Gross = 0, Net = 0, Commission = 0, IVA = 0
→ Show booking.note prominently in dashboard
→ Coordinator must classify as:
   - Zero Income (courtesy/gift) — confirm $0
   - Unpaid — follow up for payment

All other cases where invoice_price > 0 → treat as real booking with money due.
```

Note: This replaces all previous Zero Income / Unpaid detection rules.

### ALL AFFILIATES NOW BILLING TYPE (Changed June 8, 2026)

All affiliates were switched to Billing type in FareHarbor.
This means:

```
invoice_price = ALWAYS Net to Adventure Lab (regardless of who collected)
Commission = Gross - Net - IVA (always calculated)

No need for referral vs billing detection.
No need for affiliate type list in Config.js.
```

### SIMPLIFIED FINAL MONEY RULES

```
STEP 1 — Status
  IF status = "cancelled" or "rebooked" → exclude from revenue

STEP 2 — Flags
  IF payments = [] AND invoice_price = 0 → NEEDS REVIEW flag
  IF payments = [] AND invoice_price > 0 → UNPAID flag

STEP 3 — Money (all active bookings)
  Gross Revenue    = receipt_total / 100
                     IF receipt_total = 0 → use invoice_price / 100
  Net Revenue      = invoice_price / 100
                     IF invoice_price = 0 (direct) → use receipt_total / 100
  IVA              = sum of customers[*].invoice_cost.tax / 100
  Commission       = Gross - Net - IVA
  Discount         = sum of negative custom_field_values[*].total_cost.total / 100

STEP 4 — Pax
  Pax = customer_count
```

### CUTOFF DATE: Billing change takes effect June 8, 2026 @ 14:00

Bookings CREATED AFTER June 8, 2026 14:00 local time:
→ All affiliates are Billing type
→ invoice_price = always Net to ADV LAB
→ Simplified rules apply

Bookings CREATED BEFORE June 8, 2026 14:00:
→ Some affiliates were Referral type (e.g. Tregua, Hotel Aires)
→ invoice_price may = Commission owed TO affiliate (not Net)
→ For these old bookings, a referral affiliate list is needed:
   REFERRAL_AFFILIATES = ["Tregua Bacalar", "Hotel Aires", ...]
   IF affiliate in REFERRAL_AFFILIATES AND booking created_at < 2026-06-08T19:00:00Z:
       → invoice_price = Commission
       → Net = receipt_total - invoice_price - IVA
   ELSE:
       → invoice_price = Net (normal rule)

ACTION: Build the REFERRAL_AFFILIATES list with Angel to handle historical data correctly.

---

## FINAL CONFIRMED RULES (Approved June 8, 2026)

These rules replace all previous drafts above. This is the definitive version.

### STEP 1 — Status (exclude from revenue)
```
IF status = "cancelled" → exclude from revenue, keep for audit
IF status = "rebooked"  → exclude from revenue, keep for audit
```

### STEP 2 — Flags
```
IF payments = [] AND invoice_price = 0
→ Flag = "NEEDS REVIEW"
→ Gross = 0, Net = 0, Commission = 0, IVA = 0
→ Show booking.note in dashboard
→ Coordinator classifies as Zero Income or Unpaid

IF payments = [] AND invoice_price > 0
→ Flag = "UNPAID"
→ Proceed to Step 3 for money calculation
```

### STEP 3 — Money (all active bookings)
```
Gross Revenue = receipt_total / 100
                IF receipt_total = 0 → use invoice_price / 100

Net Revenue   = invoice_price / 100
                IF invoice_price = 0 (direct booking) → use receipt_total / 100

IVA           = sum of customers[*].invoice_cost.tax / 100

Commission    = Gross - Net - IVA

Discount      = sum of custom_field_values[*].total_cost.total / 100
                WHERE total_cost.total < 0
```

### STEP 4 — Pax
```
Pax = customer_count
```

### STEP 5 — Historical data (bookings before June 8, 2026 14:00 local)
```
Some affiliates were Referral type before the billing change.
For those affiliates: invoice_price = Commission (not Net) — INVERTED.
Requires REFERRAL_AFFILIATES list from Angel.
Rule:
  IF affiliate IN REFERRAL_AFFILIATES AND created_at < 2026-06-08T19:00:00Z
  → invoice_price = Commission owed TO affiliate
  → Net = Gross - Commission - IVA
```

### STEP 6 — OTA exception (GetYourGuide, Viator)
```
invoice_price = 0 (no invoice price sheet configured in FareHarbor)
→ Treat receipt_total as Net (OTA already deducted their commission)
→ Commission = unknown
→ Flag = "OTA - No Invoice"
ACTION: Angel/Valentin to configure invoice price sheets in FareHarbor
```

### Total bookings tested and confirmed: 20+
Affiliate types covered: Direct, The Yak Lake House, Adventure Lab PDV Yak,
Habitas, Tregua Bacalar, Hotel Amainah, Buena Onda, Che Bacalar, Hotel Aires,
Solana, GetYourGuide, Operaciones Adv Lab
