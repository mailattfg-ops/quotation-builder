# Think Forge Global — Quotation Builder

A single-file web app that generates branded PDF quotations and business proposals for Think Forge Global. No build step, no server, no dependencies to install — open the HTML file and it works.

---

## Quick Start

**Local server (recommended)**

```bash
npm start
# opens at http://localhost:3000
```

Or open `quotation-generator.html` directly in a browser (`file://`) — everything is self-contained.

**Deploy to Vercel**

```bash
npx vercel
```

Routes are configured in `vercel.json`:
- `/` → `quotation-generator.html`

---

## Project Structure

```
quotation-builder/
├── quotation-generator.html   # Entire app — HTML + CSS + JS in one file
├── package.json               # npm start script (npx serve)
├── vercel.json                # Vercel routing config
├── tfg-logo.png               # TFG wordmark (source asset — not loaded at runtime)
├── reference-design.pdf       # Original design reference
└── README.md
```

> The logo PNG is base64-embedded inside `quotation-generator.html` (`const LOGO_B64`). The reference PDF is kept for design reference only and is not used by the app.

---

## How It Works

The app is entirely self-contained in `quotation-generator.html`. On load, a **landing screen** presents two paths:

### Create New
Fill out the continuous scrollable form, toggle the live preview on/off, then **Save as PDF** via the browser print dialog (choose *Save as PDF* as the destination, margins set to *None*).

### Edit Existing
Upload any PDF previously exported from this tool. The app uses **PDF.js** (loaded lazily from CDN) to extract a hidden state JSON embedded in the PDF at generation time, restores every form field automatically, and drops you into the editing flow with an **EDITING** badge in the header.

---

## Document Types

| Type | Pages | Description |
|------|-------|-------------|
| **Initial Quotation** | 1–2 | Short plain-language brief for first contact. No cover page. Shows: overview, what's included (grid), price, timeline, next steps. |
| **Quotation** | 6 | Full document with styled cover, overview, objectives, feature table or service sections, pricing, scope & assumptions, signature page, back cover. |
| **Business Proposal** | 6 | Same as Quotation but with the Business Proposal cover style, closing note, and investment summary table. |

---

## Form Structure (continuous flow)

All sections scroll in a single sidebar — no tabs. Sections shown/hidden automatically based on document type:

| Section | Initial | Quotation | Proposal |
|---------|---------|-----------|----------|
| Document Type | ✓ | ✓ | ✓ |
| Client Details | ✓ | ✓ | ✓ |
| Cover Images | — | ✓ | ✓ |
| Project Overview | ✓ | ✓ | ✓ |
| Project Objectives | — | ✓ | ✓ |
| Key Deliverables | ✓ | — | — |
| Features / Services | — | ✓ | ✓ |
| Pricing & Investment | ✓ | ✓ | ✓ |
| Investment Summary | — | — | ✓ |
| Scope & Assumptions | — | ✓ | ✓ |
| Closing / Next Steps | ✓ | ✓ | ✓ |
| Closing Note | — | — | ✓ |
| Expected Outcomes | — | ✓ | ✓ |

---

## Service Content Modes (Quotation & Proposal)

| Mode | Description |
|------|-------------|
| **Feature Table** | Single or dual-plan comparison table |
| **Service Sections** | Free-form sections with bullet points, timeline, and cost per section |

---

## Preview Toggle

The preview pane is **hidden by default**. The sidebar centers on screen. Click **◧ Preview** in the action bar to open the live preview alongside the form. Click **✕ Hide Preview** to close it and return to centered form mode.

---

## Edit Existing Document (Round-Trip)

Every PDF exported from this tool contains an invisible embedded state snapshot. When a PDF is uploaded via the Edit flow:

1. PDF.js extracts the full text layer from the PDF
2. The app locates `__TFG_STATE_START__` / `__TFG_STATE_END__` markers
3. The base64-encoded JSON between the markers is decoded
4. All form fields, dynamic lists, toggles, and uploaded images are restored exactly

**Only PDFs exported from this tool are supported.** Uploading any other PDF will show a clear error message.

---

## PDF Export

- **Page size**: exactly A4 (`210mm × 297mm`) — enforced via `@media print` CSS
- **Engine**: native browser print (`window.print()`) — vector quality, fully selectable text, no quality loss
- **How to save**: click *Save as PDF* → browser print dialog → set destination to *Save as PDF*, margins to *None*

---

## Key Constants & State (inside `quotation-generator.html`)

| Name | Purpose |
|------|---------|
| `LOGO_B64` | Base64-encoded TFG logo PNG |
| `IMG_PLACEHOLDER` | Inline SVG shown when no cover photo is uploaded |
| `docType` | `'initial'` / `'quotation'` / `'proposal'` |
| `serviceMode` | `'table'` / `'sections'` |
| `planMode` | `'single'` / `'dual'` |
| `frontCoverImg` | User-uploaded front cover photo (base64) |
| `backCoverImg` | User-uploaded back cover photo (base64) |
| `previewVisible` | Whether the preview pane is shown |

---

## Updating the Logo

1. Convert your new PNG to base64:
   ```bash
   python3 -c "import base64; print('data:image/png;base64,' + base64.b64encode(open('new-logo.png','rb').read()).decode())"
   ```
2. In `quotation-generator.html`, find `const LOGO_B64 = "data:image/png;base64,..."`
3. Replace the value with the new base64 string.

---

## Updating Company Contact Details

Search `quotation-generator.html` for and replace all occurrences of:
- `www.thinkforgeglobal.com`
- `mail@thinkforgeglobal.com`
- `+91 97450 04161`

There are approximately 6–8 occurrences across the page renderers.

---

## Design System

| Token | Value | Used for |
|-------|-------|---------|
| Primary red | `#c0392b` | Accents, headings, buttons |
| Dark | `#1a1a1a` | Body text, dark backgrounds |
| Light grey | `#f8f8f8` | Page backgrounds, pricing boxes |
| Font | Inter (Google Fonts) | All text |

Cover photo areas use `clip-path: polygon(...)` for the diagonal cut effect and `filter: grayscale(100%)` on the image.

---

## Browser Compatibility

Tested in Chrome and Safari (macOS). Requires:
- CSS `clip-path` — Chrome 55+, Safari 13.1+, Firefox 54+
- `FileReader` API — all modern browsers
- `fetch` / `Promise` / `async-await` — all modern browsers
- PDF.js (CDN) — loaded only when Edit flow is used; requires internet connection on first use
