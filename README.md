# Angel Investor Portfolio Site

A clean, forkable portfolio site for angel investors. Shows your investments live from a Google Sheet — no backend, no database, just a static HTML file you can deploy in minutes.

**[Live example →](https://angelprofile.vercel.app)**

---

## Quick Start

### 1. Fork this repository for your own profile

Click **Fork** at the top of this page. You'll get your own copy at `github.com/YOUR_USERNAME/angelprofile`.

### 2. Set up your Google Sheet

Follow the [Google Sheet setup guide](docs/google-sheet-setup.md) to:
- Copy the template sheet
- Add your investments
- Publish it as CSV
- Get your Sheet ID

### 3. Create your config.json

Copy the example config and fill in your details:

```bash
cp config.example.json config.json
```

Open `config.json` and edit:

```json
{
  "name": "Your Name",
  "heroEyebrow": "Angel Investor · Founder",
  "headline": "Your one-line headline.",
  "tagline": "What you're working on now.",
  "bio": "Your bio here.\n\nSecond paragraph if you want one.",
  "photo": "assets/photo.jpg",
  "googleSheetId": "YOUR_SHEET_ID",
  "heroLinks": [
    { "label": "@yourhandle", "url": "https://x.com/yourhandle" },
    { "label": "Your website", "url": "https://yoursite.com" }
  ],
  "sectors": ["SaaS", "Fintech", "AI"]
}
```

### 4. Add your photo

Drop your headshot at `assets/photo.jpg`. Any JPEG works — it's displayed in a circle at roughly 300×300px.

### 5. Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/Dan-Hightower/angelprofile-template)

Or manually: connect your forked repo in [vercel.com](https://vercel.com), click **Deploy**. Done. Vercel auto-deploys on every push to `main`.

---

## Keeping Data Fresh

The site reads your Google Sheet on every page load, so any edit you make to the sheet appears on the site immediately. The question is how to keep the sheet itself up to date.

### Option A: Claude Cowork (recommended for non-developers)

If you use [Claude Cowork](https://claude.ai/cowork), there's a skill that scrapes your AngelList portfolio and updates a CSV you can paste into your sheet.

**Setup:**
1. Install the `get-portfolio-list` skill in Cowork
2. Run it with your AngelList slug: `get my portfolio for @yourhandle`
3. It returns a formatted list you can paste into the Portfolio tab
4. Set a monthly reminder or schedule it as a recurring Cowork task

**To schedule it automatically:**
In Cowork, use the `/schedule` command:
```
/schedule monthly: get my portfolio for @yourhandle and email me the CSV
```

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

Scrapes your AngelList Invest portfolio and returns a structured CSV you can paste into the Portfolio tab. Reads your AngelList slug from `config.json` — nothing to hardcode.

**Trigger:** "get my portfolio", "pull my AngelList data", "sync my investments"

### `generate-portco-descriptions`

For every company in your sheet with an empty Description cell, looks up the company on AngelList (and its website if needed) and generates a 3-5 word description (e.g. "Fully reusable rocket engines"). Writes results back to column G.

**Run this after `get-portfolio-list`** — the portfolio table on your site shows descriptions instead of company names, so filling this in makes the site much more readable to visitors.

**Trigger:** "describe my investments", "fill in the description column", "add company summaries"

### Recommended monthly workflow

```
1. Run get-portfolio-list    → pastes fresh CSV into your sheet
2. Run generate-portco-descriptions → fills any new Description cells
3. Push any config changes   → Vercel auto-deploys
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
