# AWS Estimate Converter

> Convert AWS Pricing Calculator exports (`.csv` or `.json`) into clean, formatted `.xlsx` spreadsheets — entirely in the browser. No backend, no uploads, no dependencies to install.

---

## What It Does

The AWS Pricing Calculator lets you export estimates as CSV or JSON files. These raw exports are useful for data but aren't presentation-ready. This tool converts them into a polished Excel workbook with:

- A styled **Excel Table** with alternating row stripes
- Proper **column alignment** — numeric values right-aligned, text left-aligned, currency centered
- An **orange-highlighted bold Total row** with `SUM` formulas
- **Autofit column widths and row heights** based on actual cell content
- **Upfront column** that is automatically shown or hidden depending on whether any service in the estimate has a one-time upfront cost
- **Configuration summary** for each service, formatted as readable key-value pairs, capped at 80 characters wide with text wrapping

---

## Supported Input Formats

### CSV (`.csv`)
The standard export from the AWS Pricing Calculator web UI.

**How to export:**
1. Open your estimate at [calculator.aws](https://calculator.aws)
2. Click **Export** → **CSV**

### JSON (`.json`)
The JSON export from the AWS Pricing Calculator. Two structural variants are supported:

| Variant | Structure | Example |
|---|---|---|
| **Format A** | `Groups.Services` → flat array | Single-group estimates |
| **Format B** | `Groups.<GroupName>.Services` → nested by group | Multi-group estimates (VPC 1, VPC 2, Landing Zone, etc.) |

**How to export:**
1. Open your estimate at [calculator.aws](https://calculator.aws)
2. Click **Export** → **JSON**

---

## Output Format

Each converted file produces a single `.xlsx` workbook with the following columns:

| Column | Description | Alignment |
|---|---|---|
| Group hierarchy | Group or estimate name | Left |
| Region | AWS region | Left |
| Description | Service description (JSON only) | Left |
| Service | AWS service name | Left |
| Upfront *(if applicable)* | One-time upfront cost | Right |
| Monthly | Recurring monthly cost | Right |
| First 12 months total | `Monthly × 12` | Right |
| Currency | Always `USD` | Center |
| Configuration summary | Key-value pairs of service config | Left, wraps |

> **Note:** The Upfront column is automatically **removed** if the total upfront across all services is `$0.00`. It only appears when at least one service has a non-zero upfront cost.

### Row Filtering

Rows are included or excluded based on the following logic:

- ✅ **Kept** — upfront > 0 (regardless of monthly)
- ✅ **Kept** — monthly > 0 (regardless of upfront)  
- ❌ **Removed** — both upfront and monthly are zero or empty

---

## Usage

This is a **single HTML file** — no build step, no server, no installation required.

### Option A — Open directly in browser
```
Double-click aws_estimate_converter.html
```

### Option B — Host on GitHub Pages
1. Push `aws_estimate_converter.html` to a GitHub repository
2. Go to **Settings → Pages**
3. Set source to your branch and `/ (root)`
4. Access via `https://<your-username>.github.io/<repo-name>/aws_estimate_converter.html`

### Option B — Serve locally
```bash
# Python 3
python -m http.server 8080

# Node.js (npx)
npx serve .
```
Then open `http://localhost:8080/aws_estimate_converter.html`

---

## How to Use

1. **Drop or click** to select one or more `.csv` or `.json` files
2. Click **⚙ Convert** — the progress bar shows 4 steps:
   - `1. Clean` — parse and filter rows
   - `2. Format` — apply cell styles and alignment
   - `3. Formulas` — write SUM formulas and inject Excel Table XML
   - `4. Finalise + Autofit` — calculate column widths and row heights
3. Click **⬇ Download** to save the `.xlsx` file(s)
4. Use **Clear all** to reset and start over

Multiple files can be queued and converted in one batch.

---

## Formatting Details

### Excel Table
The output uses Excel's native Table format (`TableStyleLight8`) injected via JSZip post-processing. This enables:
- Filter dropdowns on every header
- Alternating row stripe styling
- Compatible with Excel, LibreOffice, and Google Sheets

### Total Row
- Background fill: `FCE4D6` (Orange Accent 2, Lighter 80%)
- All values **bold**
- Uses `SUM(E3:Ex)` range formulas — recalculates automatically if you edit values in Excel
- Upfront, Monthly, and First 12 months each have their own independent SUM

### Autofit
Column widths are calculated by scanning the actual content of every cell in the table area (header row through total row). The acknowledgement rows below are excluded from the autofit calculation.

- **Configuration summary column** — capped at a maximum width of **80 characters**; content wraps and row height adjusts accordingly
- **Numeric columns** — minimum width of 16 characters
- **All other columns** — minimum width of 10 characters

### Configuration Summary
Built from the service's properties/configuration fields:
- **CSV**: parsed from the raw configuration column, leading commas stripped
- **JSON**: built by joining all `Properties` key-value pairs as `key: value, key: value, ...`

---

## Dependencies

All loaded from CDN — no local installation needed.

| Library | Version | Purpose |
|---|---|---|
| [xlsx-js-style](https://github.com/gitbrent/xlsx-js-style) | `1.2.0` | Excel file generation with cell styling |
| [JSZip](https://stuk.github.io/jszip/) | `3.10.1` | Post-processing the `.xlsx` zip to inject the Excel Table XML |

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx-js-style@1.2.0/dist/xlsx.bundle.js"></script>
```

> Both scripts are loaded from public CDNs. An internet connection is required when opening the HTML file.

---

## Browser Compatibility

| Browser | Support |
|---|---|
| Chrome 90+ | ✅ |
| Edge 90+ | ✅ |
| Firefox 88+ | ✅ |
| Safari 14+ | ✅ |

All file processing happens **client-side**. Your data never leaves your browser.

---

## Known Limitations

- **Formulas use static ranges** — if you insert rows above the total row in Excel, the SUM range won't expand automatically (use Excel's Table Total Row feature instead if needed)
- **CDN dependency** — requires internet access to load `xlsx-js-style` and `JSZip`; for fully offline use, download and bundle those scripts locally
- **Single sheet per file** — each input file produces one worksheet; multi-group JSON estimates are flattened into a single sheet with the group name in the "Group hierarchy" column

---

## File Structure

```
aws_estimate_converter.html   ← the entire tool, self-contained
README.md                     ← this file
```

---

## License

MIT — free to use, modify, and distribute.
