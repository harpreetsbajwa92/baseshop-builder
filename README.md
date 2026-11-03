# Base Shop Builder

**Live app:** https://harpreetsbajwa92.github.io/baseshop-builder/

Phone-first tracker for the goal **EMD + $250K by June 30, 2027**. It is one HTML file with no build step, published with GitHub Pages. It uses the same setup as the Daily Discipline app: localStorage is the working copy, and every device syncs through one public npoint.io JSON bin.

**No personal or recruit data lives in this repo.** Names, meetings, certifications, licences and the MD list are only in the sync store.

## Tabs
- **Header:** "GOAL: EMD + $250K by June 30, 2027", days left, and **results-only** goals (goal text only; the "so far" notes were removed in -3). A $100K business means 15 base recruits and 100,000 base points a month. That works out to 4 recruits and 23K points a week (15 ÷ 4.33 rounded up, 100,000 ÷ 4.33), and 1 recruit and 4K points a day, Mon–Sat (26 working days).
1. **Meetings:** (base shop only, tagged "BASE SHOP") a date picker (Toronto time, so he can backfill). It has two sets of rings, each for the viewed day, its week and its month: **Recruiting** (recruits vs 1 / 4 / 15) and **Production** (base points vs 4K / 23K / 100K). Recruits = Personal recruiting + Base recruiting results. Base points = Personal closing + Base closing results; those boxes are labelled "points". Rows: Personal recruiting, Personal closing, Base recruiting, Base closing (meetings + Results), plus New contacts. Sunday has no daily goal.
2. **Certs:** (base shop only, tagged "BASE SHOP") "Results vs goal" rings for the cert goal (1 a day, 7 a week, 30 a month: Today/Week/Month with "of X" and %). day by day, using the same date picker with arrows as Meetings (defaults to today). It shows who cleared which certification on that day, the count for that day, and small week and month counts. Filled by the bot (`bs-v1:certs`).
3. **Licences:** a date picker. On top, compact new-licence rings: Today count (no daily goal), Week x/1 and Month x/3 for the **base shop** goal, plus a super base row of counts (no goal), with the new names for the chosen Day, Week or Month in one small card. Below, **All licensed agents** from `bs-v1:licensed`: Base shop and Super base panels with Name, WFG code, current month cash flow and rolling 12-month cash flow, a total row, sorted by rolling-12 cash flow. Long lists scroll inside the panel. Shows "Waiting for MyWFG data" until the bot fills it.
4. **MDs:** two read-only panels from `bs-v1:mdstats`, **My base MDs** and **Sunchandan's team** (Sunchandan Singh, SMD, plus his MDs). Each row shows Name (+ title), current month recruits and current month cash flow, with "—" until filled. The bot fills it from MyWFG. A small direct SMD legs ring (x of 3) sits below.
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
| `bs-v1:licensed` | `{"month":"YYYY-MM","updated":"YYYY-MM-DD" or "","source":"MyWFG","base":[{"name","code","cf_month":number\|null,"cf_12":number\|null}],"superbase":[...]}`. Roster of ALL licensed agents for the whole team (base shop + super base = Sunchandan Singh's team). `code` = WFG agent code, `cf_month` = current month cash flow, `cf_12` = rolling 12-month cash flow (dollars). The app sorts by `cf_12` (highest first) and adds a total row per panel. Empty lists show "Waiting for MyWFG data". Seeded empty on Nov 2, 2026 (rev 20); the bot fills it from MyWFG with a write-merge. |
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
