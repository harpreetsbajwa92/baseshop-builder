# Base Shop Builder

**Live app:** https://harpreetsbajwa92.github.io/baseshop-builder/

Phone-first tracker for the goal **EMD + $250K by June 30, 2027**. It is one HTML file with no build step, published with GitHub Pages. It uses the same setup as the Daily Discipline app: localStorage is the working copy, and every device syncs through one public npoint.io JSON bin.

**No personal or recruit data lives in this repo.** Names, meetings, certifications, licences and the MD list are only in the sync store.

## Tabs
- **Header:** "GOAL: EMD + $250K by June 30, 2027", days left, and **results-only** goals (goal text only; the "so far" notes were removed in -3). A $100K business means 15 base recruits and 100,000 base points a month. That works out to 4 recruits and 23K points a week (15 ÷ 4.33 rounded up, 100,000 ÷ 4.33), and 1 recruit and 4K points a day, Mon–Sat (26 working days).
1. **Main** (was Meetings; v13 adds Self improvement and per-row "+ names" chips): (base shop only, tagged "BASE SHOP") a date picker (Toronto time, so he can backfill). It has two sets of rings, each for the viewed day, its week and its month: **Recruiting** (recruits vs 1 / 4 / 15) and **Production** (base points vs 4K / 23K / 100K). Recruits = Personal recruiting + Base recruiting results. Base points = Personal closing + Base closing results; those boxes are labelled "points". Rows: Personal recruiting, Personal closing, Base recruiting, Base closing (meetings + Results), plus New contacts. Sunday has no daily goal.
2. **Certs:** (base shop only, tagged "BASE SHOP") "Results vs goal" rings for the cert goal (1 a day, 7 a week, 30 a month: Today/Week/Month with "of X" and %). day by day, using the same date picker with arrows as Meetings (defaults to today). It shows who cleared which certification on that day, the count for that day, and small week and month counts. Filled by the bot (`bs-v1:certs`).
3. **Licences:** a "100 LICENSED TEAM" goal header (roster total of 100, with a progress bar) replaces the EMD header. Monthly only (current month, no date picker). The stats card has Base shop and Super base rings, each "X of 3" with % and the days left. Below are this month's new names in Base shop and Super base (name + date), then **All licensed agents** from `bs-v1:licensed`: Base shop and Super base panels with Name, WFG code, current month cash flow and rolling 12-month cash flow, and a total row, sorted by rolling-12 cash flow, at full length. This tab scrolls as a normal page. The others stay one screen.
- `bs-v1:llqpreport`: daily LLQP report shown as the "Daily LLQP report" card on the Certs tab. Inner JSON (in the usual `{"v":"<JSON string>","t":ms}` wrapper): `{"date":"YYYY-MM-DD","updated":"ISO time with offset","headline":"one short line","fresh":[{"name","cert","score","date":"YYYY-MM-DD HH:MM"}],"studying":[{"name","last_login":"YYYY-MM-DD HH:MM","days_left":N,"note":"short"}],"needs_call":[{"name","reason":"short","days_left":N,"days_since_login":N}],"exams_soon":[{"name","exam","date"}],"dbi":"one line on DBI Hub ticks","counts":{"active":N,"studied_24h":N,"ran_out":N}}`. Times are ET. `studying` and `needs_call` are in priority order, max ~6 each (the app shows the first 6). Fresh = cert passes dated within 48 h of the snapshot with score >= 60%. Studying = active recruits who logged in within 7 days, most recent first. `active` = In Progress / Pre-requisite required with days left > 0; `ran_out` = still listed but 0 days left; `studied_24h` = logins in the 24 h before the snapshot. Empty lists show "No fresh passes" / "No calls needed" / "No exams booked"; leave anything unknown empty, never invent. Source: `/workspace/oliver/snapshots/<date>*.csv` (compare with the previous day) plus `/workspace/dbihub/ticks_to_add_<date>.csv` for the DBI line. Written by the **bot** each morning after the LLQP fetch, with a write-merge (GET the whole doc, set only this key with `t`=now, `_rev`+1, POST with `User-Agent: curl/8`, re-GET to verify). Seeded Nov 2, 2026 at rev 22 → 23 from the Nov 2 10:48 AM snapshot.
4. **MD Factory:** a gold "MD FACTORY" title replaces the goal header on this tab. Below are two read-only panels from `bs-v1:mdstats`, **My base MDs** and **Sunchandan's team** (Sunchandan Singh, SMD, plus his MDs), at full height. Each row shows Name (+ title), recruits and cash flow, with "—" until filled. The bot fills it from MyWFG.
5. **Trends:** a dashboard row for every metric (this week against its weekly goal, and this month). Tapping a row opens the trend view (`#trends/<slug>`), which has a metric picker and Week / Month / All. Week and Month show daily bars with a dashed daily goal line. All shows weekly bars with a dashed weekly goal line. Each view shows the total against its goal, a comparison with the prior period, and ‹ › to move between periods.

## Targets (whole numbers)
Meeting targets per week, in the order personal recruiting / personal closing / base recruiting / base closing:
- Nov 2026: 3/5/3/4 (15 a week)
- Dec 2026: 4/5/5/6 (20 a week)
- Jan 2027 onward: 4/5/6/8 (23 a week). The monthly targets are 17/22/26/34.

The ramp months' monthly target is round(week × 4.33). The daily target is ceil(week / 6). New contacts are 3 a day, 18 a week and 75 a month. The trend goals with no daily target are personal recruiting results (1 a week, 3 a month), personal closing results (1 a week, 4 a month) and new licences (1 a week, 3 a month). Base recruiting results, base closing results and certifications have no target.

## Sync store
`https://api.npoint.io/5b132b3a6768f39258a9` is public and needs no auth. GET reads it, and POST replaces the **whole** document.

```
{
  "_rev": <int, +1 on every write>,
  "bs-v1:<KEY>": { "v": "<value as a JSON *string*>", "t": <edit time, ms since epoch> },
  "bs-v1:<KEY>": { "t": <ms>, "d": 1 }          // deleted (tombstone)
}
```
The prefix is `bs-v1:`, not `nn-v1:`. This app shares the `harpreetsbajwa92.github.io` origin, and so the same localStorage, with the fitness app, so its keys must never use the fitness prefix.

### Keys
| Key | Value (inside `v`, JSON-encoded) | Written by |
|---|---|---|
| `bs-v1:day:YYYY-MM-DD` | `{"m":{"pr":n,"pc":n,"br":n,"bp":n},"r":{"pr":n,"pc":n,"br":n,"bp":n},"nc":new contacts,"h":{"rec":base recruits,"pts":base points},"u":ms}`. All values are whole numbers, and a missing value means it was left blank. `bp` = **Base closing** (it was called "Base production"). `h` was added in v2026.11.02-2. Since v2026.11.02-3 it is no longer entered or shown, but the merge still keeps it and it is never deleted. Older records may still carry `m.lc`, `m.leg` and `r.lc`, `r.leg`, `r.cash`, which are kept but no longer shown. | app |
| `bs-v1:certs` | array of `{"id","date":"YYYY-MM-DD","name","cert","score":int?,"note"?,"u":ms}` | **bot** |
| `bs-v1:licences` | array of `{"id","date":"YYYY-MM-DD","name","team"?:"base"\|"superbase","source"?:"DBI Hub"\|"MyWFG","note"?,"u":ms}`. `team` was added in v2026.11.02-6. A missing `team` counts as `"base"` (Harpreet's own base shop). `"superbase"` = Sunchandan Singh's team. | **bot** |
| `bs-v1:mds` | (legacy, no longer shown since v2026.11.02-4. Kept and never deleted.) array of `{"id","name","title","stage","notes","next","u":ms}` | none |
| `bs-v1:mdstats` | `{"month":"YYYY-MM","updated":"YYYY-MM-DD" or "","base":[{"name","title":"MD"\|"SMD"\|"","recruits":int\|null,"cashflow":number\|null}],"superbase":[{...same row shape}]}`. `base` = MDs in Harpreet's base. `superbase` = Sunchandan Singh (SMD) plus his MDs. All numbers are for the current month, and `null` shows "—". The older `{"rows":[...]}` shape is still read as `base`. Migrated at rev 18 on Nov 2, 2026: base = Gokul Ravichandran, Saurabh Nair (MD), Ishdeep Singh, Dhrumil Vashi; superbase = Sunchandan Singh (SMD). | **bot** (from MyWFG) |
| `bs-v1:licensed` | `{"month","updated","source","base":[{"name","code","title","rec_month","rec_12","cf_month","cf_12"}],"superbase":[...]}`. Roster = MyWFG Downline Status (Super Team, Life and/or Securities Registered) + Harpreet. rec = Personal Recruits (month / rolling 12), cf = R09 / R08. Not listed = 0. Split by Upline SMD (Sunchandan = superbase). |
| `bs-v1:legs` | `{"target":3,"smd":["Sunchandan Singh"]}` | bot/app |

`cert` must be exactly one of: `Ethics Compliance`, `Ethics`, `Life Insurance`, `A&S`, `Seg Funds`.

`id` must be unique and stable. Use `c_YYYYMMDD_name-slug_cert` for certifications and `l_YYYYMMDD_name-slug` for licences. Before adding an entry, check whether that id already exists so nothing is logged twice.

### How the bot should write (read → modify → write → verify)
```python
import json, time, urllib.request
B = "https://api.npoint.io/5b132b3a6768f39258a9"; H = {"Content-Type": "application/json", "User-Agent": "curl/8"}
get = lambda: json.load(urllib.request.urlopen(urllib.request.Request(B + "?_=%d" % time.time_ns(), headers=H)))
D = get(); now = int(time.time() * 1000)
certs = json.loads(D.get("bs-v1:certs", {}).get("v") or "[]")
new = {"id": "c_20261103_jane-doe_ethics", "date": "2026-11-03", "name": "Jane Doe", "cert": "Ethics", "score": 90, "u": now}
if new["id"] not in {c["id"] for c in certs}:
    certs.append(new)
    D["bs-v1:certs"] = {"v": json.dumps(certs, separators=(",", ":")), "t": now}
    D["_rev"] = int(D.get("_rev", 0)) + 1
    urllib.request.urlopen(urllib.request.Request(B, data=json.dumps(D).encode(), headers=H, method="POST"))
    time.sleep(2); assert any(c["id"] == new["id"] for c in json.loads(get()["bs-v1:certs"]["v"]))
```
Always POST the full document you just read, with your change applied. Never post only one key, because that would wipe every other key. Keep `t` set to the current ms time so the devices take the new value. npoint needs a browser-like or curl `User-Agent`; it returns 403 to Python's default one.

## Live updates and cache busting
These are the same as the fitness app:
- `Cache-Control: no-cache` meta tags.
- `APP_VERSION` in `index.html` must equal `"v"` in `version.json`. On open, when the app comes back to the foreground, and every 60 s, the app fetches `version.json?t=<now>` with `cache:"no-store"`. If the version differs, it reloads once to `?v=<new>`.
- Sync runs on open, on returning to the foreground, when the device comes back online, every 60 s, and 2 s after any edit.

**On every deploy, bump both `APP_VERSION` and `version.json`.**

On iPhone, open the link in Safari, tap Share, then **Add to Home Screen**.

## Changelog
- **v2026.11.02-18 (Nov 2, 2026):** Certs tab upgrade. Below the goal rings, Who cleared and the daily LLQP card (headline, count chips, Updated, fresh passes, exams in 7 days, DBI) there are now full-width **Practice tests** and **Certification exams** tables for all 19 active Oliver's recruits: a sticky name column with days left and last login, every attempt score in date order, Ethics Compliance status, x/4 passed and a coaching comment. They scroll sideways on a phone. **Highlights & calls to action** follows, and "Needs a call" and "Studying" were removed from the daily card. Store: `bs-v1:llqpreport` gained practice/exams/highlights/actions (rev 27 → 28).
- **v2026.11.02-17 (Nov 2, 2026):** Licences → New licences: the two goal cards (Base shop gold, Super base blue) are now compact cards the same size and style as the New names cards. They sit side by side on phone and laptop, with a 48px ring (count, "of 3"), "n of 3 · %", "x to go · d days left", and the ✓/green state when the goal is hit. Nothing else changed.
- **v2026.11.02-16 (Nov 2, 2026):**
  - Licences roster fixed. The licensed list was built from the cash flow reports, so licensed agents with $0 cash flow were missing. The roster source is now the MyWFG **Downline Status** report: Team = Super Team (Base - 6th), Type = Life and/or Securities Registered, all title levels, as of 11/2/2026. That gives 29 people; Harpreet (1298FC, SMD, report owner, not listed in his own downline) is added from the cash flow / recruits reports, so 30 in total: base 22, super base 8.
  - Removed versus before (not Life/Securities registered in Downline Status): Pardeep Singh (262SPC, MD), Amit Hooda, Satnam Dass Jangra, Jasmine Malhotra. Added: Varsha Gunda, Athul Babu Kochath Babu, Tisha Patel, Rajveer Randhawa, Arpit Shah, Sumit Sumit, Simran Kaur Bhatia.
  - Each person gets `rec_month` (Custom Report Personal Recruits, Oct 2026), `rec_12` (Personal Recruits, Nov 2025–Oct 2026), `cf_month` (R09) and `cf_12` (R08). These reports list only non-zero people, so anyone not listed gets 0.
  - Base vs super base comes from the Upline SMD column (recruits 12 mo, then R08/R09). If there is none, the Platform Fee report "Team: MD Base" (1298FC) puts them in base (Varsha, Athul, Rajveer, Sumit). Unconfirmed (no upline in any file, put in base): Tisha Patel 32BSZC and Arpit Shah 369BDC. Both are in the Super Team platform fee list but not the MD Base list, so they may be Sunchandan's.
  - Licences tab "All licensed agents" tables: Name, Code, Rec Oct, Rec 12, CF Oct, CF 12 (sorted ▾), with totals. Compact on phone: names wrap, $10K+ shows as $37.3K. The 100 LICENSED TEAM header counts the new roster (30).
  - Store: `bs-v1:licensed` replaced (rev 24 → 25). `bs-v1:mdstats` got `recruits_12` from the Personal Recruits Nov 2025–Oct 2026 report, 0 if not listed (rev 25 → 26). Every MD's Oct `recruits` matches the Oct 2026 Personal Recruits report.
- **v2026.11.02-15 (Nov 2, 2026):**
  - MD Factory: both tables (My base MDs, Sunchandan's team) now show 4 numbers per MD with totals: Rec <month> (`recruits`), Rec 12 mo (`recruits_12`), CF <month> (`cashflow`), CF 12 mo (`cf_12`). Blank/null shows "—". Cash flow of $10K or more shows as e.g. $37.3K. Still one screen on phone and laptop, no inner scroll.
  - Store: `bs-v1:mdstats` rows got `cf_12` (copied from `bs-v1:licensed` by WFG code, = R08 Custom Report "Top 100 Total CashFlow in Associate's Entire Hierarchy" Nov 2025–Oct 2026) and `recruits_12: null` (no per-MD rolling-12 recruit count exists in the local exports). Write-merge rev 23 → 24, only this key changed.
- **v2026.11.02-14 (Nov 2, 2026):**
  - Certs: new compact gold "Daily LLQP report" card from the new key `bs-v1:llqpreport`: "Updated <date, time>" label, headline, count chips (Active, Studied 24h, Ran out), 🎉 Fresh passes (one line per recruit), 📖 Studying (last login, days left, note), 📞 Needs a call (amber, with reason, days left, days since login), 🗓 Exams in 7 days and the DBI Hub line. Empty sections show one short line. Laptop: beside the rings (wider right column, report split into two inner columns). Phone: below the "Who cleared" list. The Certs page now scrolls normally (fitBoxes removed for Certs); no inner scroll boxes.
  - The key is read through the normal sync (45 s GET, hedged mirror read, 3 retries, quick retry after errors); the card shows "Loading…" / "Can't reach the store yet. Retrying…" until data arrives and rerenders after sync.
  - Store: `bs-v1:llqpreport` seeded with a write-merge (rev 22 → 23) from the Nov 2 snapshots. No other key changed.
- **v2026.11.02-13 (Nov 2, 2026):**
  - The Meetings tab is renamed **Main** in the bottom nav. The tab id (`#meet`) and all data keys are unchanged.
  - Main: new **Self improvement** row after New contacts. It is a count (`si` in the day record) with a goal of 1/d · 7/wk · 30/mo, plus a "+ note" chip that opens a text line for what he did (`sin`). It does not feed the recruiting or production rings. Trends has a new "Self improv." tile and trend view in the Meetings group.
  - Main: each row (Personal recruiting, Personal closing, Base recruiting, Base closing, New contacts) has a "+ names" chip. Tapping it opens a text line for the people met, comma-separated; the chip then shows "👤 N". The names are saved in the same day record as `nm:{pr,pc,br,bp,nc}`. Only one line is open at a time, so the tab stays one screen (rows tightened to fit 390×844 and 1280×800).
  - Day record now: `{"m":{..},"r":{..},"nc":n,"si":n,"sin":"text","nm":{"pr":"A, B",...},"h":{..},"u":ms}`. The sync merge for day records now keeps EVERY field from both sides (the newer side wins per field), and merges `m`, `r`, `h` and `nm` key by key. Older code dropped unknown fields on a two-sided merge.
  - Sync: hedged read. If api.npoint.io hasn't answered in 6 s, the app also asks the www mirror, and the first good answer wins (npoint's api was taking 26–41 s on some requests during testing).
  - Tested with uploads blocked: a test day (2026-10-01, only on the test browser) saved and reloaded a count, names and a note. The real day records and the store were untouched (store still at _rev 22).
- **v2026.11.02-12 (Nov 2, 2026):**
  - Licences tab: the EMD + $250K header is replaced (on this tab only) by a gold "100 LICENSED TEAM" goal header. It shows the current total from `bs-v1:licensed` (base + super base, 19 + 8 = 27 now), "of 100", "N to go", a gold progress bar with %, and a split line (Base shop N in gold · Super base N in blue). Meetings, Certs and Trends keep the EMD header; MD Factory has its own title.
- **v2026.11.02-11 (Nov 2, 2026):**
  - Licences: Super base now has a monthly goal of 3 as well. Its ring matches Base shop ("X of 3", a % pill, "N to go · D days left", a ✓ and a green ring when hit), and it stays blue. The header now reads "goal 3 / month each". The Trends "Super base licences" tile and trend view use the same goal as base (1/wk, 3/mo).
  - MD tab renamed **MD Factory** (bottom nav and title). On this tab only, the big goal header is replaced by a gold "MD FACTORY" title card with the subtitle "My base MDs + Sunchandan's team · <month>". Other tabs keep the goal header.
  - MD Factory: the tables are full height with no inner scroll box or sticky header/total (the page scrolls if needed). The "Direct SMD legs: X of 3" card is removed. `bs-v1:legs` is still in the store, just not shown.
- **v2026.11.02-10 (Nov 2, 2026):**
  - Licences is monthly only. The date navigator, the Day/Week/Month toggle and the Today/Week ring columns are gone. On top is one stats card for the current month: a big Base shop ring "X of 3" with a % pill and "N to go · D days left" (green ✓ when hit), plus a blue Super base count (Sunchandan's team, no goal). Below it, "New names" for the month in Base shop and Super base panels (name + date licensed, "None yet" when empty). Then the full "All licensed agents" roster: both panels always shown (stacked on a phone, side by side on a laptop), at FULL length with no inner scroll box. **This tab scrolls like a normal page.** All other tabs still fit one screen.
  - Sync reliability fix (the "Waiting for MyWFG data" / "offline, saved on this device" bug): every npoint request used to abort after 15 s and a single failure marked the device offline until the next 60 s tick, so a slow npoint response (it can stall for a minute or more) left a device with no `bs-v1:licensed` at all. Now GET waits 45 s and POST 90 s. Reads retry 3 times, alternating with npoint's mirror `https://www.npoint.io/documents/<id>` (same doc under `contents`, needs `Accept: application/json`). Pulled data is applied and shown immediately, before any upload. A failure retries in 10–60 s, not just on the 60 s tick. The version.json check now has an 8 s timeout so it can never block the sync. Footer states: "synced", "loaded, upload retrying", or "offline, showing saved data · retrying". The roster shows "Loading…" or "Can't reach the store yet. Retrying…" instead of "Waiting for MyWFG data" while nothing has loaded.
- **v2026.11.02-9:**
  - Certs: the count card is replaced by a "Results vs goal" ring grid (BASE SHOP tag), styled like Meetings. It has Today/Week/Month rings, each with "of X" and a %, against the cert goal of 1 a day, 7 a week and 30 a month. The rings count `bs-v1:certs` entries (excluding `team:"superbase"`) for the selected day, its week and its month. The date picker and the day's "Who cleared" list stay; a long list scrolls inside its card.
  - Trends: the Certifications tile and trend view now use the cert goal (1/day, 7/wk, 30/mo).
- **v2026.11.02-8:**
  - `bs-v1:licensed` filled (interim) from the MyWFG Oct 2026 custom cash-flow reports (R08 rolling 12 Nov 2025–Oct 2026, R09 Oct 2026; MASTER_v2 sheets 8–9). Split by the "Upline SMD" column: Sunchandan Singh (27RJEC) and everyone with Upline SMD Sunchandan = super base; everyone else under 1298FC (incl. Harpreet) = base. People in the 12-month report with no Oct row get `cf_month` 0.
  - `bs-v1:mdstats` updated from the report titles: month 2026-10, base = all 8 MDs with Upline SMD Harpreet (Oct cash flow), superbase = Sunchandan (SMD) + his 3 MDs. Recruits left blank.
  - The app labels the cash-flow month from the key (`month`), e.g. "CF Oct 26" in the roster and "Cash flow Oct 2026" on the MD tab, instead of assuming the current month.
  - MD tables scroll inside their panels (sticky header and total) so the MD tab stays one screen; the recruits total shows "—" when no recruits are filled.
- **v2026.11.02-7:**
  - Meetings and Certs are base-shop-only reports and carry a gold "BASE SHOP" tag. Certs ignores any entry tagged `"team":"superbase"`.
  - Licences: compact new-licence rings (Base shop with the 1/wk, 3/mo goal; Super base counts) and a one-card list of new names for the Day/Week/Month.
  - Licences: new "All licensed agents" roster from the new key `bs-v1:licensed`, in Base shop and Super base panels (Name, WFG code, CF month, CF 12 mo, total row), sorted by rolling-12 cash flow. The list scrolls inside its panel so the page stays one screen. On a phone a Base / Super base toggle shows one panel at a time; on a laptop both show side by side.
  - Store: `bs-v1:licensed` seeded empty with a write-merge (rev 19 → 20). No other key changed.
- **v2026.11.02-6:**
  - Licences: goal rings like Meetings (Today count, Week x/1, Month x/3 for the base goal, plus super base counts with no goal). Two panels, Base and Super base, list licence names for the Day, Week or Month.
  - `bs-v1:licences` entries gain `team` ("base" or "superbase"; missing = base).
  - MDs: two panels, "My base MDs" and "Sunchandan's team" (Name + title, recruits, cash flow, totals). `bs-v1:mdstats` now has `base` and `superbase` lists (migrated with a write-merge at rev 18). The SMD legs ring is kept.
  - Trends: "Base licences" (goal 1 a week, 3 a month) and "Super base licences" (no goal).
  - Still one screen at 390×844 and 1280×800.
- **v2026.11.02-5:**
  - Compact one-screen layout. Every tab fits with no scrolling at 390×844 (phone) and 1280×800 (laptop).
  - The header is now the gold goal line, then days left, then one centred row of Month / Week / Today goals.
  - Meetings: the results rings are a 2×3 grid of small rings (recruiting and production × day/week/month), next to tight meeting and result rows with new contacts inline. On laptop the rings and rows sit side by side.
  - Trends is a compact tile grid (2 columns on phone, 4 on laptop). Certs, Licences and MDs are tighter.
  - Points under 10K now show one decimal (3.5K).
  - No features, keys or store data changed.
- **v2026.11.02-4:**
  - Premium look: a gold GOAL headline and centred goal tiles.
  - Meetings rings now show results (recruits and base points) instead of meetings.
  - Certs and Licences are day by day with a date picker.
  - The licence goal is now 1 a week and 3 a month.
  - MDs is a table from the new key `bs-v1:mdstats`, which was seeded with a write-merge so no store data was lost.
  - The Trends tab gets a Results group (Recruits and Base points, with goals). The closing-results charts are now in points.
- **v2026.11.02-3:** Removed the Headline results card (Base recruits / Base points inputs) from the Meetings tab. The header goals stay, without the "so far" notes. Removed the Base recruits / Base points trend charts, since they no longer have an input. Store data is unchanged, and any existing `h` values are kept.
- **v2026.11.02-2:** results-only header goals, 4 meeting rows, headline results inputs, Trends tab.
