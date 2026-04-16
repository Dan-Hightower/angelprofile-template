---
name: get-portfolio-list
description: "Scrape the user's AngelList Invest portfolio and return a structured list of all investments. Use when the user asks to pull their portfolio data, sync investments, update their sheet, or export their AngelList positions. Triggers include: 'get my portfolio', 'pull my AngelList data', 'sync my investments', 'update the sheet with my portfolio', or 'export my positions from AngelList'."
---

# Get Portfolio List

Navigates to the user's AngelList Invest portfolio dashboard and fetches all investment positions using the Apollo Client cache. Returns structured data suitable for pasting into the Google Sheet.

## Setup

Read the AngelList slug from `config.json`:

```json
{
  "angellistSlug": "your-angellist-slug"
}
```

The portfolio dashboard URL is: `https://invest.angellist.com/v2/@{angellistSlug}/portfolio`

## How it works

AngelList's portfolio dashboard loads positions via GraphQL and stores them in the Apollo Client cache on the page. Rather than clicking through pagination, we read the cache directly after the page loads.

## Steps

1. **Navigate** to `https://invest.angellist.com/v2/@{slug}/portfolio`
2. **Wait** for the positions table to finish loading (the network requests to settle)
3. **Extract the Apollo cache** by running in the browser console:

```javascript
// Run this in the browser console after the portfolio page loads
const cache = window.__APOLLO_CLIENT__?.cache?.data?.data;
const positions = Object.values(cache || {})
  .filter(v => v.__typename === 'PortfolioInvestment')
  .map(v => ({
    company:   v.companyName,
    status:    v.status,
    invested:  v.invested?.amount,
    value:     v.currentValue?.amount,
    tvpi:      v.tvpi,
    date:      v.investmentDate,
    slug:      v.companySlug,
  }));
console.table(positions);
```

4. **Paginate if needed** — if there are more than 50 positions, scroll to load additional pages before extracting
5. **Format output** as CSV matching the sheet column order:

```
Company,Status,Invested ($),Current Value ($),TVPI,AngelList Slug,Description,Investment Date
Stoke Space Technologies,Live,5000,4500,0.90,stoke-space,,2021-03-15
```

Leave Description blank — run `generate-portco-descriptions` afterwards to fill it in.

## Column mapping

| Sheet Column | Source field |
|---|---|
| A — Company | `companyName` |
| B — Status | `status` (Live/Realized/Closing/Transferred) |
| C — Invested ($) | `invested.amount` |
| D — Current Value ($) | `currentValue.amount` |
| E — TVPI | `tvpi` |
| F — AngelList Slug | `companySlug` |
| G — Description | *(leave blank)* |
| H — Investment Date | `investmentDate` (YYYY-MM-DD) |

## Output

Return:
1. A summary count (e.g. "Found 133 positions across 118 companies")
2. The full CSV data ready to paste into the Portfolio tab
3. Any positions that couldn't be parsed (flag for manual review)

## Notes

- The Apollo cache method is faster and more complete than scraping the visible table — it captures positions that may not be visible on screen
- If `window.__APOLLO_CLIENT__` is undefined, the page may still be loading — wait a few seconds and retry
- For realised investments, `currentValue` may be the exit value at the time of exit
