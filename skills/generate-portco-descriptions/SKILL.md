---
name: generate-portco-descriptions
description: "Generate short 3-5 word descriptions for portfolio companies and write them directly into the Google Sheet. Use this skill whenever the user asks to add descriptions, summarize companies, label investments, or generate taglines for their portfolio. Also trigger when the user says things like 'describe my investments', 'add company summaries to the sheet', 'what do my portfolio companies do', or 'fill in the description column'. This skill is designed to run after get-portfolio-list has populated the sheet with company data."
---

# Generate Portfolio Company Descriptions

Generates concise 3-5 word descriptions for each company in the portfolio and writes them directly into the Description column (col G) of the Google Sheet using browser automation.

## Setup

Read `config.json` for:

```json
{
  "googleSheetUrl": "https://docs.google.com/spreadsheets/d/SHEET_ID/edit#gid=0"
}
```

If `googleSheetUrl` is not in config, ask the user for the editable URL of their Google Sheet.

## When to run

After `get-portfolio-list` has populated the sheet, many companies will have empty Description cells. Run this skill to fill them in.

Only writes to **empty** Description cells — never overwrites existing descriptions.

## Description examples

Good descriptions are concrete and specific:
- "Supersonic passenger aircraft"
- "AI-powered legal contract review"
- "Construction procurement software"
- "Carbon removal via ocean chemistry"
- "Fully reusable rocket engines"

Avoid vague phrases like "enterprise SaaS platform" or "B2B software company."

## Steps

1. **Navigate to the Google Sheet** — open the Portfolio tab
2. **Read the sheet** — identify rows where column G (Description) is empty, and note the company name (col A) and AngelList slug (col F) for each
3. **For each company missing a description:**
   - Open a new tab and navigate to `https://angellist.com/company/{slug}` using the slug from col F
   - Extract the one-line description or pitch from the company page
   - If no description found, visit the company's website and derive a 3-5 word summary from the homepage headline
   - Condense to 3-5 words
4. **Write to the sheet** — navigate back to the Google Sheet tab, click the empty cell in column G for that row, and type the description
5. **Repeat** for all companies missing descriptions

## Efficiency tip

Batch the work: research all descriptions first, then go back to the sheet and fill them all in at once rather than switching tabs for each company.

## Output

Report:
1. How many descriptions were added
2. Any companies where no description could be found (flag for manual review)

## Notes

- The user must be logged into Google in the browser
- Never overwrite existing descriptions — only fill empty cells
- Keep descriptions to 3-5 words maximum
