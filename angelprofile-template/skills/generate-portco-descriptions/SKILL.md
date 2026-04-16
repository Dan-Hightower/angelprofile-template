---
name: generate-portco-descriptions
description: "Generate short 3-5 word descriptions for portfolio companies and update the Google Sheet. Use this skill whenever the user asks to add descriptions, summarize companies, label investments, or generate taglines for their portfolio. Also trigger when the user says things like 'describe my investments', 'add company summaries to the sheet', 'what do my portfolio companies do', or 'fill in the description column'. This skill is designed to run after get-portfolio-list has populated the sheet with company data."
---

# Generate Portfolio Company Descriptions

Generates concise 3-5 word descriptions for each company in the portfolio and writes them to the Description column (col G) of the Google Sheet.

## When to run

After `get-portfolio-list` has populated the sheet, many companies will have empty Description cells. Run this skill to fill them in.

Only writes to **empty** Description cells — never overwrites existing descriptions.

## Source priority

For each company missing a description, try in order:

1. **AngelList portco page** — the company detail page usually has a one-line pitch and market tags. Condense to 3-5 words.
2. **Company website** — if AngelList has no description, visit the website URL from the portco page and derive a 3-5 word summary from the homepage headline.

## Description examples

Good descriptions are concrete and specific:
- "Supersonic passenger aircraft"
- "AI-powered legal contract review"
- "Construction procurement software"
- "Carbon removal via ocean chemistry"
- "Fully reusable rocket engines"

Avoid vague phrases like "enterprise SaaS platform" or "B2B software company."

## Steps

1. **Read config** — load `config.json` to get the user's AngelList slug
2. **Read the sheet** — fetch the Portfolio tab CSV and identify rows where col G (Description) is empty
3. **For each empty row:**
   - Build the AngelList portco URL: `https://angellist.com/company/{slug}` using the value from col F
   - Navigate to the page and extract the one-line description
   - If no description found, visit the company website and derive one from the homepage
   - Condense to 3-5 words
4. **Write to sheet** — update each empty Description cell via Google Sheets Apps Script (see below)
5. **Report** — summarize how many descriptions were added and flag any companies where no description could be found

## Writing to Google Sheets via Apps Script

The site reads the sheet as a published CSV (read-only from the browser). To write descriptions back, use a Google Apps Script bound to the sheet:

```javascript
// In Google Sheets: Extensions → Apps Script → paste this function
function updateDescriptions(updates) {
  // updates: array of { row: number, description: string }
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Portfolio');
  updates.forEach(({ row, description }) => {
    sheet.getRange(row, 7).setValue(description); // col G = 7
  });
}
```

If the user doesn't have Apps Script set up, output the descriptions as a formatted list they can paste manually into the sheet.

## Output format (fallback)

If writing via Apps Script is not available, output a table the user can paste:

```
Row | Company        | Description
----|----------------|---------------------------
2   | Stoke Space    | Fully reusable rocket engines
3   | Aether Bio     | mRNA delivery for rare disease
```
