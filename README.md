# Angel Investor Portfolio Site

A clean, forkable portfolio site for angel investors. Shows your investments live from a Google Sheet — no backend, no database, just a static HTML file you can deploy in minutes.

**[Live example →](https://angelprofile.vercel.app)**

---

## Quick Start

### 1. Fork this repository for your own profile

Click **Fork** at the top of this page. You'll get your own copy at `github.com/YOUR_USERNAME/angelprofile`. Then clone it locally:

```bash
git clone https://github.com/YOUR_USERNAME/angelprofile.git
cd angelprofile
```

### 2. Set up your Google Sheet

Follow the [Google Sheet setup guide](docs/google-sheet-setup.md) to:
- Copy the template sheet
- Add your investments
- Publish it as CSV
- Get your Sheet ID (the long string between `/d/e/` and `/pub` in the published URL)

### 3. Create your config.json

Inside your cloned repo, copy the example config:

```bash
cp config.example.json config.json
```

Open `config.json` and fill in your details. The most important field is `googleSheetId` — paste the Sheet ID from step 2:

```json
{
  "name": "Your Name",
  "heroEyebrow": "Angel Investor · Founder",
  "headline": "Your one-line headline.",
  "tagline": "What you're working on now.",
  "bio": "Your bio here.\n\nSecond paragraph if you want one.",
  "photo": "assets/photo.jpg",
  "googleSheetId": "PASTE_YOUR_SHEET_ID_HERE",
  "heroLinks": [
    { "label": "@yourhandle", "url": "https://x.com/yourhandle" },
    { "label": "Your website", "url": "https://yoursite.com" }
  ],
  "sectors": ["SaaS", "Fintech", "AI"]
}
```

> **Where do I find my Sheet ID?** After publishing your Google Sheet (step 2), the published URL looks like `https://docs.google.com/spreadsheets/d/e/2PACX-1vXXXXXX.../pub?output=csv`. The Sheet ID is the long string between `/d/e/` and `/pub`. See the [setup guide](docs/google-sheet-setup.md) for details.

### 4. Add your photo

Drop your headshot at `assets/photo.jpg`. Any JPEG works — it's displayed in a circle at roughly 300×300px.

### 5. Deploy

Connect your forked repo to any static hosting provider and deploy. No build step required — it's just a static HTML file.

**Vercel:** Import your repo at [vercel.com/new](https://vercel.com/new), click **Deploy**. Auto-deploys on every push to `main`.

**Netlify, GitHub Pages, or any static host** will also work.

---

## Keeping Data Fresh

The site reads your Google Sheet on every page load, so any edit you make to the sheet appears on the site immediately. The question is how to keep the sheet itself up to date.

### Option A: Claude Code / Cowork (recommended)

The included skills use browser automation to scrape your AngelList portfolio and write the data directly into your Google Sheet — no manual copy-pasting.

**Setup:**
1. Add `googleSheetUrl` to your `config.json` (the editable URL of your sheet)
2. Add `angellistSlug` to your `config.json`
3. Run: `get my portfolio for @yourhandle`
4. The skill opens AngelList, scrapes your positions, then opens your Google Sheet and writes the data in

### Option B: Codex Scheduler

If you have access to [OpenAI Codex](https://openai.com/codex) or a similar code execution environment:

1. Use this prompt on a monthly schedule:
   ```
   Scrape the AngelList portfolio for @YOUR_ANGELLIST_SLUG and return
   a CSV with columns: Company, Status, Invested, Current Value, TVPI,
   AngelList Slug, Description, Investment Date
   ```
2. Paste the output into your Google Sheet's Portfolio tab

### Option C: Manual updates

Just edit the Google Sheet directly whenever you make a new investment or want to update valuations. The site reflects changes immediately.

---

## Skills

This repo includes two Claude skills for keeping your portfolio data fresh. Install them by copying the `skills/` folder into your Claude skills directory (`~/.cursor/skills/` for Cursor, or your Claude Cowork skills path).

### `get-portfolio-list`

Scrapes your AngelList Invest portfolio and writes the data directly into your Google Sheet via browser automation. No manual pasting needed.

**Trigger:** "get my portfolio", "pull my AngelList data", "sync my investments"

### `generate-portco-descriptions`

For every company in your sheet with an empty Description cell, looks up the company on AngelList (and its website if needed) and generates a 3-5 word description (e.g. "Fully reusable rocket engines"). Writes directly into the sheet.

**Run this after `get-portfolio-list`** — the portfolio table on your site shows descriptions instead of company names, so filling this in makes the site much more readable to visitors.

**Trigger:** "describe my investments", "fill in the description column", "add company summaries"

### Recommended monthly workflow

```
1. Run get-portfolio-list              → scrapes AngelList, writes to your sheet
2. Run generate-portco-descriptions    → fills any new Description cells
3. Push any config changes             → auto-deploys
```

---

## config.json Reference

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Your full name — used in the hero, footer, and page title |
| `heroEyebrow` | No | Small text above your name in the hero (default: "Angel Investor") |
| `headline` | Yes | H2 in the About section |
| `tagline` | Yes | Subtitle line in the hero |
| `bio` | Yes | About section body text. Use `\n\n` for paragraph breaks. HTML is supported for bold/links. |
| `photo` | No | Path to your headshot (default: `assets/photo.jpg`) |
| `googleSheetId` | Yes | The published CSV ID from your Google Sheet — see [setup guide](docs/google-sheet-setup.md) |
| `heroLinks` | No | Array of `{ label, url }` links shown in the hero section |
| `sectors` | No | Array of sector tags shown as pills in the About section |
| `showPortfolioHighlights` | No | When `true`, shows only 2x+ TVPI investments instead of the full table (default: `false`) |
| `googleSheetUrl` | No | Editable Google Sheet URL — used by skills to write data directly into the sheet |

---

## Google Sheet Setup

See the full [Google Sheet setup guide](docs/google-sheet-setup.md).

**Sheet columns (Portfolio tab):**

| Col | Header | Notes |
|-----|--------|-------|
| A | Company | Legal name |
| B | Status | Live / Realized / Closing / Transferred |
| C | Invested ($) | Dollar amount |
| D | Current Value ($) | Current mark; 0 for write-offs |
| E | TVPI | Formula: `=D2/C2` |
| F | AngelList Slug | Company slug on AngelList |
| G | Description | One-liner shown in the portfolio table |
| H | Investment Date | YYYY-MM-DD format |
| I | IRR | Formula: `=IF(C2>0, POWER(D2/C2, 365/MAX(1, TODAY()-H2)) - 1, "")` |

---

## What the stats bar shows

The stats bar at the top pulls from the **Profile tab** of your sheet:

- **Total Investments** — number of checks written
- **Startups** — unique companies
- **Deployed** — total dollars invested
- **Portfolio Multiple** — TVPI (total value / total invested)
- **Portfolio IRR** — dollar-weighted annualized return, computed in-browser from your investment dates and values
- **If I Were a VC Fund** — your percentile rank vs. 2020-vintage VC funds, using [Carta Q4 2025 benchmark data](https://carta.com/data/vc-fund-performance-q4-2025/)

---

## Local Development

No build step required. Open `index.html` directly in a browser — but note that `config.json` and the Google Sheet are fetched via HTTP, so you'll need a local server for those requests to work:

```bash
# Python
python3 -m http.server 8080

# Node
npx serve .
```

Then open `http://localhost:8080`.

---

## License

MIT. Fork it, use it, make it yours.
