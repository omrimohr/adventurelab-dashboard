# Adventure Lab — Roadmap & AI Handoff Guide

**Purpose:** This file lets ANY AI model (not just the one that wrote it) continue building the full dashboard — Operational, Commercial, and Admin. Read `CONTEXT.md` first (workspace rules, IDs, push commands), then this file for what to build next and how.

Written 2026-07-08. Update the status marks as work ships.

---

## 1. Where the project stands (2026-07-08)

DONE and live:
- FareHarbor → Webhook.js → Google Sheets pipeline, token-enforced (WEBHOOK_ENFORCE=true since 2026-07-07)
- Raw tab = append-only event log; rebooking guard; ~10 extra booking fields; nightly rebuildTours
- Ops day-view `ops.html`: approvals, freshness badge, Cumplimiento Capitán, insumos check-out/check-in per tour, ⛽ Gastos (gasolina + compras)
- Owner dashboard `index.html` incl. Facturación page; QBO staging→approval→post chain working end-to-end in SANDBOX
- Correction report backend (Reportes tab + Drive folder)
- Security: DASHBOARD_KEY on doGet, WEBHOOK_TOKEN on doPost, secrets in Script Properties only

---

## 2. Big next steps (in recommended order)

### Step A — Data hygiene (small, do first)
- Run `?action=fix_duplicates`. Known duplicates since 2026-06-22: booking PK 359014237; tour AvPKs 2012411443, 2013277641, 2013277847, 2014354094.
- Verify counts after; then optionally add a weekly duplicate-audit to morningRun.

### Step B — Phase 3: Commercial approval (`com.html`)
- New page `dashboard/com.html` for Angel (commercial manager): review each day's staged QBO lines after Ops approval, approve/reject per affiliate, comments.
- Backend already has the approval-chain schema (Ops → Commercial → Admin); Phase 3 = the UI + the `com_approve_*` doGet/doPost actions in API.js/Webhook.js, following the exact same patterns as ops.html (apiUrl/getAccessKey/handleUnauthorized/text-plain POST contract — see §4).
- Admin trigger: an Admin page/section (could live in index.html) with "Aprobar día → post to QBO" replacing today's sandbox auto-post.

### Step C — QBO production cutover (needs the accountant, not just code)
- Flip `CFG.SANDBOX_MODE` to `false` ONLY after: real QBO company connected via OAuth, real Mexican IVA TaxCode mapped (sandbox currently errors with US TaxCode), customer/product names matched to the real company, accountant sign-off.
- Keep the quincena-accumulation invoice model (redesigned 2026-06-22) — do not revert to daily invoices.

### Step D — Real login (replaces device-name identity)
- Today `getApprover()` = device name. Replace with a simple username + PIN: a Users tab (usuario, pin_hash, rol: ops/com/admin), a `login` doPost action returning a session token stored in localStorage, and role checks on approve actions.
- Keep it simple — no OAuth, no external auth service. This is a small internal team.

### Step E — Ops corrections UI
- Ops needs to fix things webhooks don't carry: crew/captain reassignment, walk-ons (pax without booking), no-show adjustments. Add to ops.html per tour, writing through existing correction endpoints (Corrections.js) — remember `clearOpsDayCache_(dateStr)` after any direct sheet write.

### Step F — Captain checklists (deferred by design)
- Captains do NOT get a dashboard (decision 2026-07-06 — they only check in on FareHarbor). Future: simple per-tour checklist forms feeding the ops dashboard, incl. incidencias.

### Step G — Code health (do opportunistically, not as a project)
- `index.html` (~90KB) and `QBO.js` (~101KB) are oversized. When touching them, split by responsibility (e.g. QBO into QBO_Auth / QBO_Staging / QBO_Post). Never restructure in one big pass.
- `?action=data` caps at 500 rows — will need pagination or date filtering as data grows.

### Step H — Nice-to-haves (only if asked)
- Daily digest (WhatsApp/email summary of yesterday), PWA wrapper for ops.html, shopping-list view from insumos stock minimums, admin UI for editing Recetas/Catálogo (today edited directly in the Sheet).

---

## 3. Working rules for ANY model (critical — the owner relies on these)

1. **Omri is not a developer.** Plan first, present it, then execute one step at a time. Be concise. Show progress as visual ✅/⬜ checklists.
2. When asking him to run anything in the terminal, say **which folder**; when asking him to run an Apps Script function, say **which FILE it's in**.
3. `dashboard/` repo is **PUBLIC** — never hardcode keys/secrets; they live in Script Properties. Confirm with Omri before every `git push`.
4. Apps Script deploys: `clasp push` from `apps-script/adv-lab-dashboard/`, then redeploy ONLY the pinned deployment: `clasp deploy -i AKfycbxnEEtwt_cysahiCnd1PQvXJdmaq19obsvs5EDJIm8DvJv9PjLFmCFwnXLhoT_qcB2yHA -d "description"`. Never create new deployments; only 2 should exist (pinned + @HEAD).
5. **Sheet-creation times out** on this large spreadsheet — create tabs one per execution with flush (see `setupInventorySheetsStepwise()` in Inventory.js), never in bulk.
6. Frontend↔backend contract: reads = GET `apiUrl('?action=X')` (auto-appends `key=`); writes = POST with `Content-Type: text/plain;charset=utf-8` and JSON body, routed in Webhook.js doPost via `isAuthorizedRequest`. Exception: gas_boat/gas_station are doGet actions with query params.
7. FareHarbor fires 2+ near-duplicate webhooks per change (LockService serializes them); Raw tab is append-only — readers must last-write-win by UUID. Activity/availability notes and slot crew changes do NOT fire webhooks.
8. Run `graphify query "<question>"` before reading source files (token savings); `graphify . --update` after code changes (code-only extraction works without an LLM key).
9. Update `CONTEXT.md` session log at the end of every session. Two AI tools share this codebase (Claude Code + MiniMax) — never push from both at once; routing rules are in CONTEXT.md.
10. Test on live data carefully: use the test functions pattern (testXxx + cleanupXxx in the same file) and ask Omri to run them from the editor, pasting back the log.

---

## 4. Skills / capabilities the AI tool needs

Installed and used today (Claude Code plugins — reinstall if missing):
- **superpowers** plugin — `brainstorming` → `writing-plans` → `subagent-driven-development` workflow. This is HOW features get built here: spec in `docs/superpowers/specs/`, plan in `docs/superpowers/plans/`, then execute task-by-task with reviews.
- **graphify** (`~/.claude/skills/graphify/`) — codebase knowledge graph, query before reading files.

If those aren't available (different model/tool), replicate manually: write a short design doc → get Omri's approval → write a step-by-step plan → implement one task at a time with a review after each.

Domain knowledge needed (no special tooling — docs in this repo):
- Google Apps Script (V8), clasp CLI, SpreadsheetApp/CacheService/LockService quirks
- FareHarbor webhook payloads — see `FareHarbor_JSON_Fields_Reference.md` and `docs/superpowers/specs/2026-07-05-webhook-event-log-review.md`
- QuickBooks Online API (OAuth2, Invoices, TaxCodes, sandbox vs production)
- Plain HTML/CSS/JS (no frameworks) — mobile-first for ops.html

Connectors that would help but are NOT set up (optional):
- QuickBooks connector/MCP — only for inspecting QBO data; posting already goes through QBO.js. Requires OAuth authorization by Omri.
- Gmail/WhatsApp sending — only needed for the daily-digest idea (Step H).
- Browser automation (claude-in-chrome / Playwright) — useful for driving the Apps Script editor; note the Chrome MCP allowlist only permits script.google.com, and FareHarbor cannot be automated — Omri does FareHarbor actions himself.

---

## 5. Where everything is documented

| Topic | File |
|---|---|
| Workspace rules, IDs, push commands, security model, session logs | `CONTEXT.md` |
| Webhook architecture + empirical firing rules | `docs/superpowers/specs/2026-07-05-webhook-event-log-review.md` |
| Insumos/gasolina design | `docs/superpowers/specs/2026-07-06-insumos-gasolina-inputs-design.md` |
| Past implementation plans | `docs/superpowers/plans/` |
| FareHarbor payload fields | `dashboard/FareHarbor_JSON_Fields_Reference.md` |
