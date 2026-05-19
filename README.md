# Think Forge Global — Quotation Builder

A single-file web app that generates branded quotation and business proposal PDFs for Think Forge Global. No build step, no server, no dependencies to install — open the HTML file and it works.

---

## Quick Start

**Local preview (recommended)**

```bash
npx serve -p 3000 .
# open http://localhost:3000/quotation-generator.html
```

Or simply open `quotation-generator.html` directly in a browser (file://) — everything is self-contained.

**Deploy to Vercel**

```bash
npx vercel
```

Routes are already configured in `vercel.json`:
- `/` → `ai_video_strategy_analysis.html` (company homepage — separate project)
- `/quotation` → `quotation-generator.html` (this app)

---

## Project Structure

```
quotation-builder/
├── quotation-generator.html   # The entire app — HTML + CSS + JS in one file
├── vercel.json                # Vercel routing config
├── tfg-logo.png               # Think Forge Global wordmark (source asset)
├── reference-design.pdf       # Original PDF the design is based on
└── README.md
```

> The logo PNG and reference PDF are **not** loaded at runtime. The logo is base64-embedded inside `quotation-generator.html` (search for `const LOGO_B64`). The reference PDF is kept for design reference only.

---

## How It Works

The app is entirely self-contained in `quotation-generator.html`:

- **Sidebar form** (420 px) — 7 tabs: Client, Overview, Objectives, Services, Pricing, Scope, Closing
- **Live preview pane** — rebuilds all pages on every keystroke via `updatePreview()`
- **PDF export** — clones the preview DOM and passes it to [html2pdf.js](https://github.com/eKoopmans/html2pdf.js) (loaded from CDN)

### Document modes

| Mode | Cover style |
|------|-------------|
| **Quotation** | Clean — large client name, city photo diagonal right, TFG logo top-left |
| **Business Proposal** | Full design — "BUSINESS PROPOSAL" headline, photo with diagonal cut, red accent bars, TFG logo bottom-left |

### Service content modes

| Mode | Description |
|------|-------------|
| **Feature Table** | Single or dual-plan comparison table |
| **Service Sections** | Free-form sections with bullet points |

---

## Key Variables (inside `quotation-generator.html`)

| Variable | Line | Purpose |
|----------|------|---------|
| `LOGO_B64` | ~460 | Base64-encoded TFG logo PNG |
| `CITY_B64` | ~461 | Base64-encoded city skyline (default cover photo) |
| `docType` | state | `'quotation'` or `'proposal'` |
| `serviceMode` | state | `'table'` or `'sections'` |
| `planMode` | state | `'single'` or `'dual'` |
| `frontCoverImg` | state | User-uploaded front cover photo (overrides `CITY_B64`) |
| `backCoverImg` | state | User-uploaded back cover photo |

---

## Updating the Logo

1. Convert the new PNG to base64:
   ```bash
   python3 -c "import base64; print('data:image/png;base64,' + base64.b64encode(open('new-logo.png','rb').read()).decode())"
   ```
2. In `quotation-generator.html`, find `const LOGO_B64 = "data:image/png;base64,..."`
3. Replace the value with the new base64 string.

---

## Updating Company Contact Details

Search `quotation-generator.html` for:
- `www.thinkforgeglobal.com`
- `mail@thinkforgeglobal.com`
- `+91 97450 04161`

Replace all occurrences (there are ~6–8).

---

## PDF Output

- **Page size**: 794 × 1123 px (A4 at 96 dpi)
- **Pages**: Cover → Executive Summary → Objectives → Services/Pricing → Scope → Back Cover
- **Engine**: html2pdf.js via jsDelivr CDN (requires internet for first PDF export)

### Offline PDF export

If internet access isn't available, download html2pdf.js and host it locally:

```html
<!-- Replace this CDN line in the <head> -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
<!-- With a local copy -->
<script src="./html2pdf.bundle.min.js"></script>
```

---

## Design Reference

The design is based on `reference-design.pdf` (the original TFG business proposal PDF). Key design decisions:

- **Color**: Crimson `#c0392b`, near-black `#1a1a1a`, light grey `#f8f8f8`
- **Font**: Inter (Google Fonts CDN)
- **Cover photo**: diagonal clip via `clip-path: polygon(110px 0, 100% 0, 100% 100%, 0 100%, 0 130px)`
- **Logo position**: bottom-left of cover (Business Proposal), top-left (Quotation)

---

## Browser Compatibility

Tested in Chrome and Safari (macOS). The app uses:
- CSS `clip-path` — requires a modern browser (Chrome 55+, Safari 13.1+, Firefox 54+)
- `FileReader` API — for cover image uploads (all modern browsers)
- `html2pdf.js` — relies on html2canvas + jsPDF under the hood
