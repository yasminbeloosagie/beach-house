# Belo-Osagie Beach House — App Handover Document

## What this document is for
This document contains everything needed to continue building or editing the beach house inventory app in a new conversation. Share this at the start of any new chat and Claude will have full context.

---

## The App in One Sentence
A mobile-first web app for managing inventory at the Belo-Osagie beach house in Ibeshe, Lagos — used by the caretaker (Usman) to submit weekly check-ins, and by the Lagos team to monitor stock levels and log dispatches.

---

## Live URLs
- **App:** `https://yasminbeloosagie.github.io/beach-house`
- **GitHub repo:** `https://github.com/yasminbeloosagie/beach-house`
- **Google Sheet:** `https://docs.google.com/spreadsheets/d/1JEhhxycy0AYqyu04IEm8mx-5Mm882xCMLF8x-80d0_s`

---

## Tech Stack
| Layer | Tool | Purpose |
|---|---|---|
| Hosting | GitHub Pages | Hosts the app at a permanent URL for free |
| Database | Google Sheets | Stores all inventory, dispatches, check-ins, issues |
| Backend | Google Apps Script | Handles all reads and writes between the app and the Sheet |
| Frontend | Single HTML file (index.html) | The entire app — one file |

---

## Google Sheets Setup
**Sheet ID:** `1JEhhxycy0AYqyu04IEm8mx-5Mm882xCMLF8x-80d0_s`
**Sheet sharing:** Anyone with link → Viewer

### Apps Script
The backend is a Google Apps Script deployed as a Web App. It handles all reads (GET) and writes (POST) between the app and the Sheet.

**Script URL:** `https://script.google.com/macros/s/AKfycbwZxrNuK9A23bjD_bqyLTLhLuzcb11w0z5orF6ZPsiOY2yzqwYPAUKiY6KLtjsaBt0G/exec`

**To find or redeploy the script:**
1. Open the Google Sheet
2. Click Extensions → Apps Script
3. The script is in Code.gs
4. To redeploy: Deploy → Manage deployments → create new version
5. Update `SCRIPT_URL` in `index.html` if the URL changes

**Script supports three operations:**
- `GET ?tab=TabName` — reads all rows from a tab and returns them as JSON
- `POST` with `{ action: "append", tab, row }` — appends a new row
- `POST` with `{ action: "update", tab, id, fields }` — updates specific fields on a row by id

### Tab structure
The Google Sheet has 4 tabs:

**Inventory** — columns: `id, name, type, unit, quantity, reorder_at, active`
- `type` is either `consumable` or `durable`
- `active` is `TRUE` or `FALSE` — deleting sets to FALSE, never removes the row
- `reorder_at` is the threshold — dashboard alerts when quantity ≤ this number

**Checkins** — columns: `id, timestamp, item_id, item_name, quantity, notes`
- One row per item per check-in submission
- All rows from same submission share the same timestamp

**Dispatches** — columns: `id, date_sent, item_id, item_name, quantity, sent_by, notes`
- Logged by Lagos team when items are sent to beach house
- `date_sent` is manual — can be backdated if forgotten

**Issues** — columns: `id, timestamp, type, message, urgency, resolved`
- `type` values: `urgent-report`, `missing-delivery`, `delivery-confirmed`, `checkin`
- `urgency` values: `urgent`, `can-wait`, `info`
- `resolved` is `TRUE` or `FALSE` — Lagos team marks as resolved on dashboard

---

## User Roles & PINs
| Role | PIN | Access |
|---|---|---|
| Caretaker | None | Weekly check-in form only |
| Team | 1234 *(change this)* | Dashboard, inventory view, dispatch log |
| Admin | 5678 *(change this)* | Everything + add/edit/delete items, change reorder thresholds |

**To change PINs:** Edit `index.html` in GitHub, find these lines and update:
```javascript
const TEAM_PIN  = '1234';
const ADMIN_PIN = '5678';
```

---

## App Structure — All Screens
The app is a single HTML file with multiple screens shown/hidden via JavaScript.

| Screen ID | Who sees it | What it does |
|---|---|---|
| `screen-role` | Everyone | Role selector — Caretaker / Team / Admin |
| `screen-pin` | Team + Admin | PIN entry keypad |
| `screen-checkin` | Caretaker | Weekly inventory check-in (exact counts with + / − buttons) |
| `screen-delivery` | Caretaker | Confirm a pending delivery has arrived |
| `screen-missing` | Caretaker | Tick items missing from a delivery |
| `screen-urgent` | Caretaker | Report an urgent mid-week issue |
| `screen-dashboard` | Team + Admin | Main dashboard with 3 tabs: Home, Inventory, Dispatch |
| `screen-add-item` | Admin only | Add a new item to inventory |
| `screen-log-dispatch` | Team + Admin | Log items sent to beach house |
| `modal-delete` | Admin only | Confirmation before removing an item |

### Dashboard tabs
- **Home tab** — stat cards (out/low/ok/total), alert banners for open issues, needs-restocking list, reconciliation view
- **Inventory tab** — all items with expandable edit rows (qty + reorder threshold for admin, qty only for team)
- **Dispatch tab** — log new dispatch, full dispatch history

---

## Key Logic

### Stock status
- `out` = quantity is 0
- `low` = quantity > 0 but ≤ reorder_at
- `ok` = quantity > reorder_at

### Reconciliation (anti-theft)
Formula: `had at last check-in + dispatched since = should have`
Compare to `currently reported quantity`. Gap > 2 triggers a warning flag.

### Weekly rhythm
1. Sunday — Lagos team sends reminder to Usman
2. Usman opens app → Caretaker → fills in exact counts → submits
3. Dashboard updates automatically
4. Lagos team checks dashboard → sees what needs restocking
5. Logs dispatch when items sent (can backdate if forgotten)
6. Usman confirms delivery when items arrive
7. Lagos team marks any issues as resolved

### Delivery flow
- Lagos logs a dispatch → Usman sees "delivery expected" banner when he opens app
- Usman taps "Confirm received" → issue logged as confirmed
- If something is missing → Usman taps "Something is missing" → ticks missing items → Lagos sees red alert on dashboard → Lagos taps "Mark as resolved" after sorting replacement

---

## How to Make Changes

### Change a PIN
Edit `index.html` in GitHub. Find `const TEAM_PIN` or `const ADMIN_PIN` and update the value.

### Add a new screen or feature
1. Start a new Claude chat
2. Share this document
3. Describe what you want to add
4. Claude will provide updated code
5. In GitHub: click `index.html` → pencil icon → select all → paste new code → Commit changes
6. Use a descriptive commit message (e.g. "Add check-in history screen")

### Update the Apps Script URL
If the script is redeployed and gets a new URL, find `const SCRIPT_URL = '...'` in `index.html` and replace it.

### Update the Sheet ID
The Sheet ID is only referenced in the Apps Script (Code.gs), not in `index.html`. Apps Script automatically connects to the sheet it is attached to.

---

## Known Issues / Things Not Yet Built
- **Resolved confirmation screen** — "Mark as resolved" works but just refreshes the dashboard. No dedicated confirmation screen.
- **Check-in history view** — ability to see past check-ins for a specific item over time. Not yet built.
- **Edit dispatch entry** — admin can't yet correct a wrong dispatch entry. Would need a new screen.
- **Rename item** — admin can change quantity and reorder threshold but not the item name. Small addition needed to the edit screen.
- **PINs are hardcoded** — they're in the code, not in the Sheet. Fine for now but could be moved to Sheet later.

---

## Users
| Person | Role | Device | Location |
|---|---|---|---|
| Yasmin | Admin + Team | iPhone | Lagos |
| Assistants | Team | Android | Lagos |
| Operations Manager | Team | Android | Lagos |
| Usman | Caretaker | Android | Ibeshe beach house |

---

## How to Deploy Changes
1. Make code changes (or have Claude make them)
2. Go to `https://github.com/yasminbeloosagie/beach-house`
3. Click `index.html`
4. Click the pencil ✏️ icon to edit
5. Select all (Cmd + A) and delete
6. Paste the new code
7. Write a descriptive commit message
8. Click **Commit changes**
9. Wait ~2 minutes for GitHub Pages to redeploy
10. Hard refresh the app link to see changes

---

## Security Notes
- Apps Script URL is visible in the public GitHub code — acceptable because the script only accesses this one Sheet and runs as the Sheet owner
- PINs are basic 4-digit codes — sufficient for a small family team tool
- Sheet is set to "Anyone with link can view" — write access goes through Apps Script only
- Do not share PINs publicly

---

## Commit History Reference
| Date | Commit message | What changed |
|---|---|---|
| Apr 2026 | Initial commit | README added |
| Apr 2026 | Add files via upload | First version of app (index.html) — read-only via Google Sheets API |
| Apr 2026 | Switch backend to Apps Script for read/write support | Replaced Google Sheets API with Apps Script — all reads and writes now work properly |

---

*Document last updated: April 2026*
*App version: v1.1 — Apps Script backend, full read/write working*
