# Westgate Supply: quote matcher (team version)

The user works at Westgate Supply, a steel, pipe and fittings distributor, and sends vendors requests for quotes (RFQs). Claude's job: when a vendor quote email comes into **the user's own inbox**, find the matching line in the **Westgate Open Lines** sheet and write the quote in.

The user may be new to the terminal. Explain things plainly and keep reports short.

## Connectors this needs
- **Gmail:** the user's own Westgate account. Search only their inbox.
- **Google Sheets:** needed to write cells. If it isn't connected, ask the user to turn it on (claude.ai → Settings → Connectors) and stop.
- **Claude in Chrome:** Gmail can't read PDF attachments, so open the email in Chrome (logged into the user's Westgate Gmail) and read the PDF preview there.
- Load the `anthropic-skills:google-workspace` skill, if available, before the first sheet write.

## The only sheet this writes to
**Westgate Open Lines Due by Nov 15**, tab **"Line Items"**
- Spreadsheet ID: `1mmK5DCjDx6wppaEXcbM3HfbA1eBt-yLzgpirYraYVT8`
- The header row is row 5, and data starts on row 6.
- **Never write to any other sheet or tab.** If a quote has no matching row here, report it in chat and don't write it anywhere.

Teammates edit and rearrange this sheet. Always re-read the header row and find the columns **by header name**, not by letter:

| Header | Used for |
|---|---|
| Customer PO # | match |
| Description | match |
| Qty, UOM | match |
| Restrictions | AML / country-of-origin check |
| Vendor Quotes Received | **write**: append the quote here |
| Delivery Status | **write**: if it is exactly `No vendor PO yet`, change it to `Quoted, no vendor PO yet` |

Why the Delivery Status change: the sheet colors a row orange while its status starts with "No vendor". Changing it to "Quoted, ..." clears the orange for that row only.

## Write format
- Vendor Quotes Received: `Vendor Name: $price/UOM` (e.g. `Elite Supply: $231.38/EA`).
- If something in the quote needs a heads-up but is still a match, add a short note in parentheses, e.g. `Elite Supply: $272.73/EA (SA-516-70 not stated)`. The caveat stays with that price wherever it goes.
- If the same vendor and price are already in the cell or its note, skip it. A teammate may have entered the same quote.

## More than one quote on a line
- **The cell shows only the cheapest quote.**
- **Every other quote goes into a note on that cell** (a Google Sheets note, shown when you hover). One quote per line:
  ```
  Other quotes:
  Elite Supply $231.38/EA (Q20291819, 11/5)
  ```
- **A new quote that's cheaper** replaces the cell text, and the old cell text moves into the note.
- **A new quote that's more expensive** is only added to the note.
- **A vendor revising their own price** follows the same rule.
- If a cell already holds several quotes separated by `; ` (older entries), keep the cheapest in the cell and move the rest into the note.
- **Only compare the same unit.** If the units differ (per FT vs per EA vs per 100 LB), ask the user instead of guessing.
- On a tie, keep the existing quote.
- Never delete a price. Every quote ends up either in the cell or in the note.
- Read the cell **and its note** before writing, and re-read both after.
- To write a note, use the Sheets `update_spreadsheet` tool with an `updateCells` request on that cell and `fields: "note"`. The tab's numeric sheetId comes from `get_spreadsheet`. Write the full note text: existing lines plus the new line.

## Matching rules
- Match on Customer PO + item + quantity + UOM. Many descriptions repeat with different quantities, so the PO and quantity decide the row.
- Treat equivalent terms as the same: `RFSO` / `slip-on`, `150#` / `150 lb`, `Sch 40` / `S/40` / `STD`, `Sch 80` / `XH` / `XS`, `SA-105` / `A105`, and size spacing (`10x10x 5/16` = `10x10x5/16`).
- Check grade (e.g. A500 B vs C, A106B vs A53B), length, quantity, and AML / approved-mill / country-of-origin restrictions.
- **Ambiguous, or the specs don't line up → don't write.** Show the user the candidate rows and ask.
- A vendor quoting one combined quantity for two identical lines (e.g. 72 pcs covering a 34 and a 38) is fine: write the unit price on both rows.
- Partial quantities, substitutions (e.g. 5/16" wall instead of 1/4"), "no stock" and "can't quote" are not matches. Report them, don't write them.

## Safety checks for every write
1. Re-read the target cells right before writing. Teammates edit this sheet too.
2. Write.
3. Re-read to confirm the text landed on the right row (PO and description still match).
4. Log the email thread in `processed.json` in this folder: thread id, vendor, rows written, timestamp, last message id. Skip threads already logged unless they have a newer vendor message.

## Approval
- By default, show the user a table of the proposed changes and **wait for their OK** before writing.
- If `CLAUDE.local.md` (the user's personal file, not shared) says `quote-writes: pre-approved`, write clear matches without asking, then report what was written. Still ask about anything ambiguous.
