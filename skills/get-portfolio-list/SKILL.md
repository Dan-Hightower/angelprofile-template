---
name: get-portfolio-list
description: "Scrape the user's AngelList Invest portfolio and update their Google Sheet automatically. Use when the user asks to pull their portfolio data, sync investments, update their sheet, or export their AngelList positions. Triggers include: 'get my portfolio', 'pull my AngelList data', 'sync my investments', 'update the sheet with my portfolio', or 'export my positions from AngelList'."
---

# Get Portfolio List

Scrapes the user's AngelList Invest portfolio and writes the data directly into their Google Sheet using browser automation. No APIs or scripts needed — just browser use.

## Setup

Read `config.json` for:

```json
{
  "angellistSlug": "your-angellist-slug",
  "googleSheetUrl": "https://docs.google.com/spreadsheets/d/SHEET_ID/edit#gid=0"
}
```

If `googleSheetUrl` is not in config, ask the user for the editable URL of their Google Sheet.

## Steps

### Part 1: Scrape AngelList

1. **Navigate** to `https://invest.angellist.com/v2/@{slug}/portfolio`
2. **Wait** for the positions table to finish loading
3. **Extract the Apollo cache** by running in the browser console:

```javascript
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
```

4. **Paginate if needed** — scroll to load additional pages if there are more than 50 positions, then re-extract

### Part 2: Write to Google Sheet

5. **Navigate** to the user's Google Sheet URL (the Portfolio tab, gid=0)
6. **Clear existing data** — select all data rows below the header (row 2 to last row), right-click → Delete rows. Keep the header row intact.
7. **Click cell A2** to start entering data
8. **For each position**, enter the values across columns A through H:
   - A: Company name
   - B: Status (Live / Realized / Closing / Transferred)
   - C: Invested amount (number only, no $ sign)
   - D: Current value (number only)
   - E: TVPI formula: `=D{row}/C{row}`
   - F: AngelList slug
   - G: *(leave blank — fill with generate-portco-descriptions)*
   - H: Investment date (YYYY-MM-DD)
   - I: IRR formula: `=IF(C{row}>0, POWER(D{row}/C{row}, 365/MAX(1, TODAY()-H{row})) - 1, "")`

**Efficiency tip:** Rather than typing cell by cell, paste all data at once:
- Click cell A2
- Paste a tab-separated block of all rows (columns A–H, with formulas for E and I)
- Google Sheets will auto-fill the grid

9. **Verify** — scroll through the sheet to confirm the data looks correct and formulas are calculating

### Part 3: Update Profile tab

10. **Switch to the Profile tab** (gid=1382849094)
11. **Update the summary stats:**
    - `total_investments` — count of all rows in Portfolio tab
    - `total_startups` — count of unique company names
    - `total_invested` — sum of column C
    - `total_value` — sum of column D
    - `multiple` — total_value / total_invested
    - `last_updated` — today's date

## Output

Report:
1. How many positions were found on AngelList
2. Confirmation that the Google Sheet was updated
3. Any positions that couldn't be parsed (flag for manual review)
4. Remind the user to run `generate-portco-descriptions` to fill in company descriptions

## Notes

- The user must be logged into both AngelList and Google in the browser
- If the Apollo cache method fails (`window.__APOLLO_CLIENT__` is undefined), wait a few seconds and retry
- Never delete or modify the header row (row 1)
- For realized investments, `currentValue` may be the exit value at the time of exit
