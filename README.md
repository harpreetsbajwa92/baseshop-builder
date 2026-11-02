# Base Shop Builder

**Live app:** https://harpreetsbajwa92.github.io/baseshop-builder/

Phone-first tracker for the goal **EMD + $250K by June 30, 2027**. It is one HTML file with no build step, published with GitHub Pages. It uses the same setup as the Daily Discipline app: localStorage is the working copy, and every device syncs through one public npoint.io JSON bin.

**No personal or recruit data lives in this repo.** Names, meetings, certifications, licences and the MD list are only in the sync store.

## Tabs
1. **Meetings**: a date picker (defaults to today, Toronto time, so you can backfill). Each category has one input for meetings and one optional Results input. The categories are Personal recruiting, Personal closing, Base recruiting, Base production, Licensing check-ins and SMD/MD leg meetings. There are also inputs for New contacts and Cash flow $. Rings show the total against the target for the day, week (Mon–Sun) and month, plus per-category rings and Results rings.
2. **Certs**: a daily certification log (filled by the bot), with counts for today, this week, this month and all time.
3. **Licences**: a new-licence log (filled by the bot), with rings against 2 a week and 8 a month.
4. **MDs**: an editable list of MD candidates, plus a direct SMD legs ring (target 3).

## Targets (whole numbers)
Weekly meeting targets, in the order personal recruiting / personal closing / base recruiting / base production / licensing / leg:
- Nov 2026 (and anything earlier): 3/5/3/4/2/2 (19 a week)
- Dec 2026: 4/5/5/6/3/3 (26 a week)
- Jan 2027 onward: 4/5/6/8/4/3 (30 a week). The monthly targets are 17/22/26/34/17/13.

The ramp months' monthly target is round(week × 4.33). The daily target is ceil(week / 6). New contacts are 3 a day, 18 a week and 75 a month. The Results rings are always at full pace:

| Result | Week | Month |
|---|---|---|
| Personal recruits | 1 | 3 |
| Personal sales | 1 | 4 |
| Base shop recruits | 3 | 14 |
| New licences | 2 | 8 |
| Super base points | 24,000 | 102,000 |
| Cash flow | $6,000 | $28,000 |

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
| `bs-v1:day:YYYY-MM-DD` | `{"m":{"pr":n,"pc":n,"br":n,"bp":n,"lc":n,"leg":n},"r":{"pr":recruits,"pc":sales,"br":base recruits,"bp":points,"lc":licences,"leg":n,"cash":dollars},"nc":new contacts,"u":ms}`. All values are whole numbers, and a missing value means it was left blank. | app |
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
