# Google Sheet Setup

Your portfolio site reads live data from a Google Sheet. Follow these steps to set it up.

## Step 1: Copy the template

Open the template sheet and make a copy for yourself:

**[Copy the template →](https://docs.google.com/spreadsheets/d/1vh_65urEOlK5KDTaqjMeS0rkBr5JHkfxH1WspNureuM/copy)**

This gives you a sheet pre-formatted with the right columns and a sample row.

---

## Step 2: Understand the two tabs

The workbook has two tabs:

### Tab 1: `Portfolio` (gid=0)
One row per investment. Columns:

| Column | Header | Description |
|--------|--------|-------------|
| A | Company | Legal company name |
| B | Status | `Live`, `Realized`, `Closing`, or `Transferred` |
| C | Invested ($) | Dollar amount invested |
| D | Current Value ($) | Current mark (use 0 for write-offs) |
| E | TVPI | Formula: `=D2/C2` |
| F | AngelList Slug | The URL slug for the company on AngelList |
| G | Description | One-line description shown in the portfolio table (e.g. "Fully reusable rocket engines") |
| H | Investment Date | Date of investment in `YYYY-MM-DD` format |
| I | IRR | Formula: `=IF(C2>0, POWER(D2/C2, 365/MAX(1, TODAY()-H2)) - 1, "")` |

### Tab 2: `Profile` (gid=1382849094)
Key-value pairs used for the summary stats bar. Keep the keys exactly as shown:

| Key | Description |
|-----|-------------|
| `total_investments` | Total number of checks written (including follow-ons) |
| `total_startups` | Number of unique companies |
| `total_invested` | Total dollars deployed |
| `total_value` | Current portfolio value (sum of column D on Portfolio tab) |
| `multiple` | TVPI: `total_value / total_invested` |
| `last_updated` | Date string shown in footer (e.g. `April 2025`) |

---

## Step 3: Publish the sheet as CSV

The site fetches your data as a public CSV — no API key required.

1. In your Google Sheet, go to **File → Share → Publish to web**
2. In the "Link" tab, set the first dropdown to **Entire document** and the second to **Comma-separated values (.csv)**
3. Click **Publish** and confirm

> **Note:** The sheet must be published to the web for the site to read it. Publishing does not change who can *edit* the sheet — it only makes the data publicly readable.

---

## Step 4: Get your Sheet ID

After publishing, look at the URL of your published CSV. It will look like:

```
https://docs.google.com/spreadsheets/d/e/XXXXXXXXXXXXXXXXXXXXXXXXX/pub?...
```

The long string between `/d/e/` and `/pub` is your **Sheet ID**. Copy it.

> **Alternatively**, you can get it from the publish dialog directly — it appears in the link shown after you publish.

---

## Step 5: Add the Sheet ID to config.json

Open `config.json` in your forked repo and paste your Sheet ID:

```json
{
  "googleSheetId": "YOUR_SHEET_ID_HERE"
}
```

Commit and push. Your site will now load live data from your sheet.

---

## Keeping data fresh

The site reads data on every page load. There's no cache — whatever is in the sheet appears on the site immediately.

For updating your portfolio data, see the [README](../README.md#keeping-data-fresh) for options including a Claude-powered AngelList scraper.
