# Kaizen Tracker — Apps Script upgrade

I'll return updated `Code.gs`, `Index.html`, `Styles.html`, `Scripts.html` for you to paste into the Apps Script editor. Everything stays in Google Sheets + Drive — no new services.

## 1. Three roles with a two-stage approval

New role set stored in the `Users` tab:

| Role | Can do |
|---|---|
| `engineer` | Submit Kaizens for their own factory, upload before/after + evidence photos, edit their own drafts |
| `manager` | Everything an engineer can, plus approve/reject their factory's submissions |
| `admin` (Group IE Head) | All factories, group-level approval, user management, config, import/export |

Status flow on each record:

```text
Submitted  ->  Factory Approved  ->  Group Approved
       \-> Rejected (with a reason, sent back to the engineer)
```

Scorecards count **Group Approved** records. Factory dashboards show both stages so a manager can see what is still waiting on you.

Two extra columns are added to each factory tab: `ApprovedByFactory`, `ApprovedByGroup` (timestamp + username), plus `RejectReason`. Existing rows are migrated automatically on setup — nothing is deleted.

## 2. Per-factory monthly targets

The `Config` tab gets a second block:

```text
Factory | TargetPerCategory
AAL     | 2
ZAL     | 2
DTX     | 3
DML     | 2
SSL     | 2
```

You can edit these in-app from the Settings screen (no need to touch the sheet). Category weights stay as they are today. A factory with no row falls back to the global default, so nothing breaks.

## 3. Security fixes (the important part)

Right now every server function is callable by anyone who can open the web app URL — the role is only checked in the browser. Someone could add users or approve their own Kaizens. I'll fix this:

- Login issues a signed session token stored in `CacheService`; every server call validates it and re-reads the caller's real role from the sheet.
- `addUser`, `resetUserPassword`, `setUserActive`, config edits, and CSV import become admin-only, enforced on the server.
- Factory users can only read and write their own factory's data — enforced server-side, not by hiding a dropdown.
- Passwords get a per-user salt instead of a bare SHA-256 hash, and the default `admin123` forces a password change on first login.

## 4. Dashboard rework — Apple-style light theme

Your existing palette (paper `#f5f3ee`, ink, orange accent) is already close, so I'll keep it and tighten it: more whitespace, softer card elevation, larger numerals, restrained motion.

**Factory dashboard** (auto-loads for the logged-in user's factory):
- Top row: Kaizen score gauge, submissions this month vs target, approval-pending count, average impact score
- Category breakdown bar (Safety / Quality / Productivity / Cost / 5S-Visual) against that factory's target
- 6-month trend line
- Recent submissions with before/after thumbnails, click to open full-size

**Group dashboard** (admin):
- Five-factory ranking table with score, submissions, target attainment, pending approvals
- Side-by-side category comparison across factories
- Group trend line and an approvals inbox you can act on directly

Every widget recomputes from the sheet on login and after each submission — no manual refresh.

## 5. Landing page and branding

Logo and brand colours pulled from armanagroup.com and used on the login page, sidebar, and exported reports. If the site's logo file can't be fetched cleanly at a usable resolution, I'll flag it and ask you to upload a PNG/SVG rather than shipping a blurry scrape.

## 6. Reliability

- One batched `getBootstrapData` call on login instead of several round-trips, so the loading overlay clears fast
- Every server call wrapped with a clear on-screen error instead of a silent hang
- Image uploads compressed in the browser before upload (Drive quota + speed)
- CSV import validates rows and reports which lines failed instead of half-importing

## What you'll do after I hand the files over

1. Paste the four files into Apps Script, save
2. Run `setup()` once — it migrates the new columns and config rows in place
3. Deploy → Manage deployments → Edit → New version
