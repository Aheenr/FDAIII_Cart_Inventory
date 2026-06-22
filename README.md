# Cart Inventory — NFC device lookup

Scan a laptop's NFC tag with a phone → a webpage opens showing the **room the laptop belongs to**, plus its asset tag, serial, and type. To track a device, you only ever write the tag **once**; the room it's assigned to lives in the spreadsheet, so moves are handled by editing the spreadsheet, not re-writing tags.

## What's in this folder
- **index.html** — the lookup page (open it on a phone after a scan).
- **Cart_Inventory_FLAT.xlsx** — your spreadsheet. The page reads this as the live source of truth.
- **tag-urls.csv** — every asset tag and the exact URL to write to its NFC tag.
- **README.md** — this file.

---

## How it works (the important idea)

Each NFC tag holds a **URL** that ends in the laptop's asset tag, e.g.

```
https://YOUR-SITE/?id=DOE-AC646348
```

When scanned, the page reads `DOE-AC646348`, looks it up in the spreadsheet, and shows its room. **The room is never stored on the tag** — so when a laptop moves carts, you just change its row in the spreadsheet and the next scan shows the new room. No re-writing tags. That's the future-proofing.

---

## Step 1 — Put it online (free, ~10 min)

The page needs to live at a web address so phones can open it. Easiest free option is **GitHub Pages**:

1. Make a free account at github.com and create a new **public** repository (e.g. `cart-inventory`).
2. Upload **index.html** and **Cart_Inventory.xlsx** to it (drag-and-drop in the browser works).
3. Repo **Settings → Pages → Source: "Deploy from a branch" → main → /(root) → Save**.
4. After a minute your site is live at `https://YOUR-USERNAME.github.io/cart-inventory/`.

That URL (with `?id=...` on the end) is what goes on the tags. Any static host works too (Netlify, Cloudflare Pages, your school web server) — just keep `index.html` and `Cart_Inventory.xlsx` in the same folder.

> The page also has a built-in copy of the data, so it still answers even if the live spreadsheet ever fails to load. When the live file loads, the footer dot turns green and says "live from spreadsheet."

## Step 2 — Write the NFC tags (NFC Tools app, Android)

For each laptop:

1. Open **NFC Tools → Write → Add a record → URL/URI**.
2. Paste that laptop's URL from **tag-urls.csv**, but replace `YOUR-SITE` with your real address, e.g.
   `https://YOUR-USERNAME.github.io/cart-inventory/?id=DOE-AC646348`
3. **OK → Write**, then hold the tag to the phone.
4. Stick the tag on that laptop.

Tip: write one tag, then in NFC Tools just edit the asset-tag part of the URL for the next one — the rest stays the same. A quick way to prep all 292 URLs at once: open `tag-urls.csv`, do a find-and-replace of `YOUR-SITE` with your address, and copy each finished URL as you go.

Optional: NFC Tools can **lock** a tag (read-only) so students can't overwrite it. Locking is **permanent** — only do it once a tag is confirmed working.

## Step 3 — Test
Open `https://YOUR-SITE/?id=DOE-AC646348` in a phone browser. You should see room **426 / Edmonson**. Then scan a real tag.

---

## Keeping it updated - Google Sheets Method

Live updates with Google Sheets (no more re-uploading)

Instead of editing the spreadsheet on your computer and re-uploading it to GitHub every time, you can point the page at a Google Sheet. Then you just edit the sheet in your browser and the change shows up on the next scan. One-time setup:


Go to sheets.google.com → File → Import → Upload and import Cart_Inventory_FLAT.csv (included in this folder). This is your whole inventory as one table: columns Room, Teacher, Asset Tag, Serial Tag, Type. Name the sheet whatever you like.
In that Google Sheet: File → Share → Publish to web. Under "Link", choose the sheet/tab and set the format to Comma-separated values (.csv), then Publish. Copy the link it gives you (it ends in output=csv).
Open index.html, find the line near the top of the script that reads:
const SHEET_CSV_URL = "";
and paste your link between the quotes:
const SHEET_CSV_URL = "https://docs.google.com/.../pub?gid=0&single=true&output=csv";
Upload that one edited index.html to GitHub once. You're done.


From now on, to move a laptop just change its Room cell; to add a device add a row; to add a room type the new room number in the Room column. No downloads, no uploads. Edits usually appear within a minute or two (Google caches the published file briefly).

Why one flat table instead of 16 tabs: a single sheet means one published link covers everything, moving a device is one cell, and a brand-new room needs no changes to the page. The footer dot turns green and reads "live from Google Sheet" when it's pulling from there.


Order of precedence: the page uses the Google Sheet if SHEET_CSV_URL is set and reachable; otherwise the local Cart_Inventory.xlsx; otherwise its built-in snapshot. So it always shows something even if Google is briefly unreachable.

## Keeping it updated (the only routine you need) - Skip Google Sheets Method

**A laptop moved, or you got new devices:**
1. Edit **Cart_Inventory.xlsx** — move the row to the correct room's sheet, or add a new row (Asset Tag, Serial, Type).
2. Re-upload **Cart_Inventory.xlsx** to your host, replacing the old one.

That's it. Existing tags keep working; the next scan reflects the change. You never touch a tag again unless the physical sticker itself changes.

**Adding a brand-new room:** add a new sheet named like the others — room number first, then teacher (e.g. `490Garcia`). Columns: `Asset Tag`, `Serial Tag`, `Type`. The page reads it automatically.

---

## One thing to clean up: 13 duplicate assignments

13 asset tags are currently listed in **two rooms at once** in the spreadsheet (for example `DOE-AC646141` is in both 444 and 446). A laptop can only be in one cart, so these are almost certainly leftovers from when a device was moved and the old row wasn't deleted. When one of these is scanned, the page shows a **"needs review"** flag and lists every room it appears in, instead of guessing.

To fix: search the tag in the spreadsheet, keep the row in its true current room, delete the other. The full list of 13 is visible by searching any of them on the page, and they're the asset tags that appear on more than one sheet.
