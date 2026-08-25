# CBE Vantage

A single-file, static, multi-page BI application for the CBE research team, comparing **CBE vs Industry** (or any set of entities) across digital financial service channels — ATM, POS, Debit Cards, Mobile Banking, Internet Banking, Mobile Money, and Agent Banking.

No backend, no build step — it's one HTML file. Runs entirely in the browser and works on GitHub Pages. The app starts empty — a researcher uploads their own CSV or Excel file each session; nothing is pre-loaded or bundled in.

**Refreshing the page doesn't lose your work.** Once data is uploaded, it — along with the page you're on, your filters, and any chart-type changes — is kept in the browser and restored automatically after a refresh. The admin console also remembers whether you were on "Manage Researchers" or "View Dashboard" and returns you there. Logging out clears this (so the next person to log in in the same browser doesn't see the previous researcher's uploaded data).

## Login & accounts

The app opens on a branded sign-in screen (CBE's actual official emblem, embedded directly in the file — no external dependency or network request needed for it) with two roles:

- **Admin** — signs in to an admin console where they can create, edit, and remove researcher accounts, change their own password, and generate the code needed to make accounts (including the admin login itself) work for everyone (see below). No default credentials are shown anywhere on the login page — contact whoever administers your deployment for the current admin login, or see "Changing the admin password" below if you're setting this up yourself.
- **Researcher** — signs in with the username/password an admin gave them, and lands directly in the BI dashboard.

**Passwords are hashed (SHA-256), never stored in plain text — and never shown in the UI except once, right when a password is created or changed.** When an admin creates a researcher or resets any password, the plain-text value is displayed once immediately after, and can't be retrieved again after that — copy it right away.

### Changing the admin password

1. Sign in as admin → **Manage Researchers** → **Change admin password**. This updates the login for your browser immediately.
2. Click **"Get permanent account code"** to get a snippet containing the new admin password's hash (and every researcher account).
3. Paste it over the existing `const BUILT_IN_ADMIN = ...` / `const BUILT_IN_RESEARCHERS = [...]` lines near the top of `index.html`'s script, then commit and push. The new password now works for every visitor to the deployed site, not just your browser.

The current baked-in admin password was set during setup — ask whoever deployed this instance, or check the commit history / `BUILT_IN_ADMIN` line in `index.html` if you have repo access (you'll only find the hash there, not the plain-text password).

**If a login stops working right after the code's admin password changes:** this shouldn't happen — a browser's locally-saved password change is tagged against the specific baked-in password it was made against, so if `index.html` gets redeployed with a *different* `BUILT_IN_ADMIN`, any old local override is automatically recognized as stale and discarded, and the new baked-in password takes over immediately. You never need to manually clear browser storage for this.

### Making accounts work for anyone who visits the site

This is a static site — there's no server or database, so an account an admin creates (or a password change) only works **on the admin's own browser** at first (stored in `localStorage`). To make it work for anyone visiting the deployed GitHub Pages site, on any device:

1. In the admin console, click **"Get permanent account code"** — this generates `BUILT_IN_ADMIN` and `BUILT_IN_RESEARCHERS` code, covering the admin login and every researcher account (hashed passwords, never plain text).
2. Copy it, open `index.html`, and paste it over the existing `const BUILT_IN_ADMIN = ...` / `const BUILT_IN_RESEARCHERS = [...]` declarations near the top of the `<script>` section.
3. Commit and push to GitHub. Once the Pages deployment updates, those accounts (and any password changes) work for every visitor, not just the browser that made the change.

Until that bake-in step is done, a newly created account or password change only works locally — the admin console clearly labels each researcher account as **"In code (works everywhere)"** or **"This browser only"** so it's obvious which ones still need to be baked in.

**Security note (read this):** hashing prevents the password itself from sitting in storage in readable form, and access is still gated by a real login screen — but this remains client-side JavaScript with no server to enforce anything. Someone with the browser's developer console can still see the stored hashes and session state, or read the `BUILT_IN_ADMIN`/`BUILT_IN_RESEARCHERS` hashes directly from the page source (though not reverse a hash back into a password). Don't use this to gate anything genuinely confidential. If real access control is ever needed, this would need a proper backend (e.g. GitHub OAuth, Firebase Auth, or a small API) in front of it.

## Pages (bottom tabs)

1. **Overview** — KPI cards for every channel, an all-channel leaderboard table, a scale-comparison chart, a footprint-share chart, **plus an individual trend chart for every single channel** (not just the combined totals).
2. **One page per channel** (ATM, POS, Mobile Banking, …) — metric picker, KPI cards, a large trend/snapshot/share view for the selected metric, a full metric comparison table, **plus an individual chart for every metric tracked in that channel** — so nothing is hidden behind the dropdown.
3. **Trends & Growth** — pick any channel+metric as the focus: full trend, year-over-year % growth, and a CAGR-by-channel leaderboard.
4. **Explorer** — free-form "build your own visual": choose channel, metric, and chart type independently.
5. **Data Table** — the full parsed dataset as a sortable, filterable grid.

Every individual chart — whether it's the big featured one at the top of a page or one of the small-multiples further down — has its own full type switcher and PNG export, so a researcher can change any single graph to any of the 8 types without affecting the others.

## Chart types

Every chart panel — on every page, not just Explorer — has a type switcher in its header offering **16 chart types**:

- **Bar**, **Bar (Pill)** — fully rounded pill-shaped bars, **Bar (Values)** — the number printed above each bar, and **3D Bar** — a shaded pseudo-3D look (Chart.js is 2D-only, so this is drawn with a custom depth-shading effect, not a true 3D engine)
- **Line**, **Line (Values)** — same as Line but with the numeric value printed above every point, and **Area**
- **Bar + Line** — a combo chart that overlays bars and a line in the same view (useful for putting one entity's scale in bars and another's trend as a line)
- **Pie**, **Pie (Values)** — the number centered inside each slice with automatic white/black text so it stays readable on any color, **Pie (Exploded)** — slices pulled slightly apart, and **3D Pie** — a shaded pseudo-3D cylinder look, plus **Donut**
- **Radar**, **Histogram**, **Scatter**

For any Line, Area, Line (Values), Bar + Line, or Scatter chart, a second row of buttons lets you change the point marker shape — **Circle, Diamond, Square, Triangle, Star, Cross** — independent of the chart type itself.

A researcher can change any individual chart to any type (and any line-family chart to any point shape) at any time — nothing is locked to a fixed visualization, and every combination exports the same as any other chart (PNG, and via the Word/PDF page exports).

Long category labels (e.g. "Value of transaction using ATM (Bill)") wrap onto multiple lines instead of being cut off, and axis ticks never auto-skip, so nothing gets silently hidden when a chart has a lot of categories.

## Colors

**CBE is always rendered in its brand purple** (`#5C2159`). **Other named Ethiopian banks get their own recognizable color** rather than a generic assigned one — the app recognizes a bank by name *or* common abbreviation, however it's spelled in the uploaded file:

| Bank | Color | Recognized as |
|---|---|---|
| Commercial Bank of Ethiopia | Purple `#5C2159` | CBE, Commercial Bank of Ethiopia |
| Awash Bank | Orange `#E8842C` | Awash, Awash Bank, Awash International Bank, AIB |
| Dashen Bank | Blue `#1F4E9C` | Dashen, Dashen Bank |
| Bank of Abyssinia | Yellow `#F2C230` | Abyssinia, Bank of Abyssinia, BOA |
| Cooperative Bank of Oromia | Light blue `#5BC2E7` | Coop, CBO, Cooperative Bank of Oromia |
| Oromia Bank (formerly Oromia International Bank) | Rust `#B5651D` | Oromia, OIB, Oromia Bank |
| Hibret Bank | Red `#C0392B` | Hibret, United Bank |
| Wegagen Bank | Green `#2E9E4F` | Wegagen |
| Nib International Bank | Teal `#00897B` | Nib, NIB |
| Zemen Bank | Deep maroon `#7A1F3D` | Zemen |
| Bunna Bank | Coffee brown `#6F4423` | Bunna |
| Abay Bank | Slate green `#5C7A6E` | Abay |
| Enat Bank | Pink `#D6408F` | Enat |
| Berhan Bank | Olive `#7D8B3F` | Berhan |

Note on accuracy: Awash's orange, Dashen's blue, Bank of Abyssinia's yellow, and Cooperative Bank of Oromia's light blue were given exactly as specified. The rest use each bank's well-known public branding where reasonably confident, chosen above all to stay clearly distinct from every other bank in the table — none of these were verified against an official brand guideline document, so if any don't match a bank's actual current colors, they're a one-line fix in `BANK_COLOR_TABLE` near the top of the script.

**Cooperative Bank of Oromia and Oromia Bank are recognized as the two different banks they actually are** — a dataset with both "Oromia" and "Cooperative Bank of Oromia"/"COOP" as separate rows gets two different colors, not the same one.

Any entity that isn't a recognized bank (e.g. "Industry", an aggregate total, or a bank not yet in the table) falls back to gold for the first one, then a rotation of distinct colors (slate violet, steel blue, terracotta, forest green, brick red, navy) for any more — chosen to stay clearly different from every named bank color above.

**CBE always leads wherever entities are listed** — KPI cards, comparison tables, chart legends, the sidebar's entity list — since this is CBE's own reporting tool. Everyone else keeps their original relative order behind CBE.

## Branding

Colors and type follow CBE's public identity as closely as could be confirmed without access to an official brand-guidelines document: the deep purple field and golden emblem that CBE is known for (and that the bank's 2025 attempt to drop in favor of a plain black wordmark was publicly rejected over). Headings use Poppins, body text uses Inter — reasonable, professional stand-ins in the absence of CBE's actual specified typeface. **If CBE's design/brand team has an official style guide (exact hex values, approved fonts, logo files), send it over and this can be tightened to match exactly.**

The login page displays CBE's actual official emblem, embedded directly in `index.html` as a base64 image (as `CBE_LOGO_BASE64` near the top of the script) — it was provided directly rather than sourced externally, so there's no external URL, no network dependency, and nothing that can go stale or break. If you ever need to swap it for a different version, replace that constant with the new image's base64 data (or point the `logoUrl` variable inside `buildLoginScreen()` at an external/local file path instead). A simple drawn emblem is kept as an `onerror` fallback purely as defensive insurance, though it should never actually be needed.

## Exporting

- **Individual chart images** — every chart panel has a small **PNG** button that downloads just that chart, rendered at 2x resolution for crisp printing.
- **Export page → Word** — bundles every chart currently on the page into a real `.docx` file with a CBE-branded header, each chart under its own (fully visible, non-truncated) heading, at full resolution. This builds a genuine Office Open XML package (a real zip of XML parts with the images embedded as proper media relationships) rather than the older "HTML file renamed to .doc" trick — that trick silently dropped inline images in Word's HTML import filter on many Word versions, which is why earlier exports opened with blank space where the charts should have been. Verified by actually opening the generated file structure and rendering it, not just by generating it.
- **Export page → PDF** — same idea via a proper PDF: one chart per page, CBE-branded header band, heading text wrapped so it's never clipped, chart image scaled to fill the page at full resolution.
- **Export data → Excel** — exports the currently filtered dataset (respecting the sidebar's entity/period filters) as a `.xlsx` file with Year / Entity / Channel / Metric / Value columns.

Chart titles are baked into the chart image itself (not just the surrounding page), so headers stay attached and fully visible in every exported PNG, Word doc, and PDF page.

## Multiple datasets

Uploading a file **adds** it as a new dataset rather than replacing what's already loaded — upload as many CSV/Excel files as you want in one session. A **Datasets** section at the top of the left sidebar lists every one by name; click a name to switch which one you're viewing (filters, chart-type choices, and the active page reset to defaults on switch, same as a fresh upload). Each dataset has a small **×** to remove it — removing the active one switches you to another loaded dataset, or back to the empty upload screen if none are left. Entity colors (e.g. CBE's purple) stay consistent across every dataset you switch between. The full set of loaded datasets, and which one was active, survives a page refresh the same way a single dataset did before.

## Filters

A persistent left-hand filter pane controls **Entities** (checkbox list — untick down to just one to see a single entity's own summary and graphs, or keep several ticked to compare) and **Periods** (years), and applies across every page. At least one entity must always stay ticked.

**Entities can be drag-reordered.** Each entity row has a drag handle — dragging one to a new position changes the order it appears in *everywhere*: the sidebar itself, KPI cards, comparison tables, and every chart's legend and data order. Colors stay tied to the entity's name, not its position — CBE is always purple regardless of where it sits in the list. This order is remembered per dataset and survives a page refresh; switching to a different loaded dataset resets to the default CBE-first order for that dataset.

The page tabs (Overview, each channel, Trends & Growth, Explorer, Data Table) sit directly below the toolbar's upload/export controls, above the sidebar and charts — so navigation and page-level actions are both up top, out of the way of the content itself.

## Deploying to GitHub Pages

1. Add `index.html` to your repo root (the filename matters — GitHub Pages serves `index.html` at the root URL).
2. Commit and push.
3. **Settings → Pages** → Source: `Deploy from a branch` → pick your branch and `/ (root)` → Save.
4. Your dashboard is live at `https://<username>.github.io/<repo>/` within a minute or two.

No other configuration is required. Because accounts are per-browser (see Security note above), there's nothing to configure server-side either.

## Data format

Accepts CSV or Excel (`.xlsx`/`.xls`) in any of three layouts — the app detects which one it's looking at automatically, in this order:

**Long/tidy format**: a flat table with headers exactly named `Year, Entity, Section, Metric, Value` (case-insensitive), one row per data point — the easiest format to produce from other systems.

**Simple entity-by-period matrix**: a plain cross-tab — one column of entity names (e.g. "Name of Bank" with CBE, Awash, Abyssinia, …) followed by one column per period (years like `2019/20`, `2020/21`, or fiscal-quarter labels like `Q1 2023`), with a single metric implied for the whole table. If there's a title row above the header (a merged cell like "Total Deposits in Millions (Birr)"), that title becomes the metric/channel name and is *not* treated as a data category — only the actual entity names in the first column are. If there's no title row, the file name is used instead. This is the right format for a simple "one metric, compared across entities, over time" report — most single-table exports from other systems will already look like this.

**Wide "banking" layout** (matches the original Digital.xlsx): row 1 = channel names spanning their metric columns, row 2 = metric names, column A = year (carried down across blank cells), column B = entity name, remaining columns = numeric values. This is for the richer case of multiple channels *and* multiple metrics per channel in one file. Works for any number of entities and any number of channels/metrics.

Whichever layout is used, **any entity named "CBE" (case-insensitive) is always recognized and always rendered in the bank's brand purple** — this is what makes the app know "this dataset is a CBE comparison" regardless of which format the file arrives in.

## Libraries used (verified working versions)

Loaded from cdnjs: Chart.js 4.5.0, PapaParse 5.4.1, SheetJS (xlsx) 0.18.5, jsPDF 2.5.1, JSZip 3.10.1. These exact versions were checked against the real npm packages to confirm the minified UMD build actually exists at that version before pinning — some Chart.js patch releases (e.g. 4.4.4) don't ship a `chart.umd.min.js`, which silently breaks every chart with no visible error. If upgrading any of these, verify the file actually exists at the new version first.

## Files

- `index.html` — the entire application. This is the only file needed to deploy.
