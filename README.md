# Vademecum - Italia

A pocket handbook for a 10-day guided tour of Italy — what to buy, what to eat, which shops are worth the detour, and the customs and tax-refund rules that bite.

Mobile-first, works offline, and remembers what you've bought on the device.

*Vademecum* is Italian for a handbook you carry with you — literally "go with me."

The interface and content are in Traditional Chinese; this README is in English.

---

## Files

```
├── index.html      Page layout and logic
├── data.json       ← All content lives here; this is the file you'll usually edit
├── sw.js           Service worker for offline use
├── manifest.json   Add-to-home-screen settings
├── icon.svg        App icon
├── img/            Optional product photos — see img/README.md
└── README.md
```

---

## Deploying to GitHub Pages

1. Create a repo (public, or private on a paid plan)
2. Upload the files to the **root** of the repo — `index.html` must sit at the top level, not inside a folder
3. Go to **Settings → Pages**
4. Set Source to **Deploy from a branch**, pick `main` / `(root)`, and save
5. Wait a minute or two; the URL will be `https://<user>.github.io/<repo>/`

All paths are relative, so the site works from a subdirectory without any changes.

### Add to home screen

Open the URL on a phone:

- **iPhone (Safari):** Share → Add to Home Screen
- **Android (Chrome):** Menu → Add to Home screen

It then opens full-screen like an app and works without a connection.

---

## Using it

### Checking things off

Tap an item to mark it **bought** — it gets a checkmark and a strikethrough. Tap again to undo. The bar at the bottom adds up the total and converts it to TWD as you go.

Prices for list items are estimated from the **midpoint** of the price range, so the total is a rough figure.

### Recording things that aren't on the list

Each day has an "自己加的" (added by me) block at the bottom. Tap **＋ 新增這天買的東西**, enter a name and the amount actually paid in euros, and confirm.

- These are **actual amounts**, not estimates
- They're filed under that day and counted toward the total
- Tap the `×` on the right of an entry to delete it

The receipt sheet lists the two kinds separately, each with its own subtotal: list items are labelled as estimated, custom entries as actually paid.

### Everything else

- Tap **明細** to open a receipt-style breakdown, where you can also **change the exchange rate**
- The **✓ 已買** chip filters down to bought items and custom entries only
- The other chips filter by day
- The progress bar shows list completion (items checked ÷ total list items)

Checkmarks, custom entries, and the exchange rate are stored in the browser's localStorage. **Clearing browser data or switching phones loses them.**

---

## How offline works

On the first load over a network, the service worker caches the page, `data.json`, and the web fonts. After that the site opens fully on a plane or with no signal.

A label at the bottom shows the current state: 已可離線使用 (ready for offline use) / 離線中 · 清單仍可用 (offline, list still available).

The strategy is **network-first**: when there is a connection it always fetches the latest version and only falls back to the cache when the request fails. Updates never get stuck behind a stale cache.

---

## Editing content

### Adding an item

Open `data.json`, find the day's `groups` → `items`, and add an entry:

```json
{
  "id": "ven-mask",
  "n": "威尼斯紙糊面具",
  "p": [20, 40],
  "note": "二月嘉年華季節選擇最多。手工 papier-mâché 為主，攤販塑膠款多為中國製。",
  "map": "https://maps.google.com/?cid=..."
}
```

| Field | Notes |
|---|---|
| `id` | **Stable identifier, required.** Checked state is keyed on this string |
| `n` | Item name, required |
| `p` | Price range as `[low, high]`. Use `null` for no price — displays as `—` |
| `note` | Description. **HTML is allowed**: `<b>bold</b>`, `<span class='warn'>red warning</span>` |
| `map` | Optional Google Maps link |

Totals use the midpoint of the range; `null` contributes nothing.

#### About `id`

Checked state is stored against the `id`, not the position. You can freely **insert, reorder, and delete** items without disturbing anything else.

Prefix convention:

| Prefix | City |
|---|---|
| `ven-` | Venice |
| `flo-` | Florence |
| `tos-` | Tuscany |
| `rom-` | Rome |
| `mil-` | Milan |
| `sm-` | Supermarket |

Two rules:

1. **IDs must be unique across the whole file.** Duplicates log an error to the console and the two items will interfere with each other.
2. **Don't rename an ID once it's in use.** Renaming is equivalent to creating a new item, and the old checked state no longer matches.

Deleting an item leaves its stored state as a harmless orphan in localStorage; it doesn't affect any calculation.

### Product photos

Each item can show a 56 px thumbnail next to its price, so you can match the thing on the shelf against what you meant to buy.

Drop a photo into `img/` named after the item's `id`:

```
img/sm-pocket-coffee.jpg
img/flo-smn-soap.jpg
```

No JSON editing needed — the path is derived from the `id`. Items with no matching file simply don't show a thumbnail. Tap a thumbnail to view it full-size.

`img/README.md` lists every filename against its item.

Notes:

- Only `.jpg` is auto-detected. For another format, add an explicit `"img": "img/xxx.png"` to the item in `data.json` to override the path
- Roughly 400×400 and under 200 KB is plenty; they render at 56 px and are cached for offline use
- Product shots of branded goods are generally someone else's copyright. Photos you take yourself are the safe option, and are more useful anyway — a shelf photo shows the packaging as you'll actually encounter it

---

### Adding a store

Add to the day's `stores` array:

```json
{
  "n": "Store name",
  "r": "★4.5",
  "kind": "pick",
  "addr": "Address",
  "hrs": "每天 8:00–21:00",
  "note": "Description",
  "map": "https://maps.google.com/?cid=..."
}
```

`kind` accepts `"pick"` (green border, recommended) or `"avoid"` (red border, steer clear). Omit it for the default gold border.

### Adding a whole day

Add an object to the `days` array. `id` must be a unique alphanumeric string — it's used for filtering and section anchors.

```json
{
  "id": "d2",
  "chip": "D2 維洛納",
  "tag": "Day 2",
  "title": "貝加莫 · 維洛納",
  "sub": "One line describing the day.",
  "groups": [{ "h": "要買", "items": [] }],
  "stores": [],
  "calls": [{ "red": true, "html": "Red callout box" }]
}
```

Custom entries are filed under `day`, so renaming a day's `id` orphans any entries recorded against it.

### After editing

Bump `VERSION` at the top of `sw.js` (`"v3"` → `"v4"`) and push. Clients pick up the new version the next time they open the site online.

This is belt-and-braces — the network-first strategy already serves fresh content — but bumping the version guarantees the old cache is cleared out.

---

## Design notes

The layout is modelled on an Italian receipt (*scontrino*), on the grounds that a shopping list is a receipt that hasn't happened yet. The name follows the same logic: this is reference material you pull out mid-trip, not an itinerary you read once before leaving.

| Token | Value | Role |
|---|---|---|
| Green | `#0F3D2E` | Darkened Italian flag green; also the signage colour of old pharmacies and cafés |
| Gold | `#9A7B2E` | Progress, store markers, secondary accents |
| Red | `#B5322A` | Warnings and cautions |
| Paper | `#FAF8F1` | Background |

Typefaces:

- **Bodoni Moda** — headings only, used sparingly. Bodoni originated in Parma, Italy
- **IBM Plex Mono** — prices, opening hours, addresses; anything meant to be scanned and compared
- **Noto Sans TC** — Traditional Chinese body text

---

## Privacy

The repo is public, so it deliberately carries no booking identifiers: no tour code, no flight numbers, and no calendar dates. Only arrival clock times and relative day numbers (D1–D10) are kept, since those are what's useful in the moment and neither identifies a booking nor advertises when a home is empty.

If you fork this, keep it that way — a public repo pinning exact travel dates to a named account is worth avoiding.

---

## Known limitations

- Checked state and custom entries live in one browser on one device; **there is no sync**
- Clearing browser data wipes them
- Custom entries stay on the device and are never written back to `data.json`
- Prices are drawn from Taiwanese travel blogs published 2024–2026 and are **estimates only**
- The exchange rate is entered manually and is not fetched automatically
- Missing product photos generate one failed request each on first load; harmless, but it's why `img/` stays optional
- Upgrading from the older three-state version drops any "want" marks and keeps "bought" ones
- Dates are intentionally absent; keep them somewhere private instead

---

## Commit conventions

This repo follows [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

<body explaining why>
```

| Type | Use for |
|---|---|
| `feat` | New functionality |
| `fix` | Bug fixes |
| `docs` | Documentation only |
| `style` | Formatting with no behaviour change |
| `refactor` | Restructuring with no behaviour change |
| `chore` | Housekeeping — versions, config, dependencies |

Subject lines are imperative, under 50 characters, and carry no trailing period. Append `!` to the type for breaking changes and explain them in a `BREAKING CHANGE:` footer.

Content-only updates to `data.json` are `chore(data):`, since they change what the list says rather than how it works.

---

*Last updated: 2026-08-03*
