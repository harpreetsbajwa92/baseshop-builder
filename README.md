# Base Shop Builder

**Live app:** https://harpreetsbajwa92.github.io/baseshop-builder/

Phone-first tracker for the goal **EMD + $250K by June 30, 2027**. It is one HTML file with no build step, published with GitHub Pages. It uses the same setup as the Daily Discipline app: localStorage is the working copy, and every device syncs through one public npoint.io JSON bin.

**No personal or recruit data lives in this repo.** Names, meetings, certifications, licences and the MD list are only in the sync store.

## Tabs
- **Header:** "GOAL: EMD + $250K by June 30, 2027", days left, and **results-only** goals (goal text only; the "so far" notes were removed in -3). A $100K business means 15 base recruits and 100,000 base points a month. That works out to 4 recruits and 23K points a week (15 ÷ 4.33 rounded up, 100,000 ÷ 4.33), and 1 recruit and 4K points a day, Mon–Sat (26 working days).
1. **Meetings:** a date picker (Toronto time, so he can backfill) and rings for today, this week and this month: meetings against target. The rows are Personal recruiting, Personal closing, Base recruiting and Base closing, each with a meetings box and a Results box. There is also a New contacts box. (The Headline results inputs were removed in v2026.11.02-3.)
2. **Certs:** the daily certification log (filled by the bot).
3. **Licences:** the new-licence log (filled by the bot), with rings against 2 a week and 8 a month.
4. **MDs:** an editable MD candidate list, plus a direct SMD legs ring.
5. **Trends:** a dashboard row for every metric (this week against its weekly goal, and this month). Tapping a row opens the trend view (`#trends/<slug>`), which has a metric picker and Week / Month / All. Week and Month show daily bars with a dashed daily goal line. All shows weekly bars with a dashed weekly goal line. Each view shows the total against its goal, a comparison with the prior period, and ‹ › to move between periods.

## Targets (whole numbers)
Meeting targets per week, in the order personal recruiting / personal closing / base recruiting / base closing:
- Nov 2026: 3/5/3/4 (15 a week)
- Dec 2026: 4/5/5/6 (20 a week)
- Jan 2027 onward: 4/5/6/8 (23 a week). The monthly targets are 17/22/26/34.

The ramp months' monthly target is round(week × 4.33). The daily target is ceil(week / 6). New contacts are 3 a day, 18 a week and 75 a month. The trend goals with no daily target are personal recruiting results (1 a week, 3 a month), personal closing results (1 a week, 4 a month) and new licences (2 a week, 8 a month). Base recruiting results, base closing results and certifications have no target.

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
| `bs-v1:licences` | array of `{"id","date":"YYYY-MM-DD","name","source"?:"DBI Hub"\|"MyWFG","note"?,"u":ms}` | **bot** |
| `bs-v1:mds` | array of `{"id","name","title","stage":"Identified"\|"Working"\|"On track"\|"Promoted","notes","next","u":ms}` | app (the bot may also write) |
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
- **v2026.11.02-3:** Removed the Headline results card (Base recruits / Base points inputs) from the Meetings tab. The header goals stay, without the "so far" notes. Removed the Base recruits / Base points trend charts, since they no longer have an input. Store data is unchanged, and any existing `h` values are kept.
- **v2026.11.02-2:** results-only header goals, 4 meeting rows, headline results inputs, Trends tab.
