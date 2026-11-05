---
name: quote-check
description: Check the user's Gmail for new vendor quote emails and write clear matches into the Westgate Open Lines sheet. Use for /quote-check, "check my email for quotes", or when the quote-watch loop fires.
---

# Quote check

Run one check of the user's inbox for vendor quotes. Follow the rules in this folder's `CLAUDE.md` (sheet ID, columns, write format, matching rules). The only sheet this may write to is **Westgate Open Lines → "Line Items"**.

## Steps

1. **Find new emails.** Search Gmail for `newer_than:1d in:inbox -from:westgatesupply.com`. This also skips quotes a teammate forwarded internally. Read `processed.json` in this folder (if it's missing, treat it as empty `[]`). Skip any thread already logged, unless it has a vendor message newer than its `last_message_id`.
2. **Read each new thread.** Pull vendor, quote number, date, each line (item, size, grade, qty, UOM, unit price), lead time, and exceptions (no stock, substitutions, partial qty). If the prices are only in a PDF, open the email in Chrome and read the PDF preview (zoom in on the line table). Never guess at numbers you can't read.
3. **Read the sheet.** Read the header row (row 5) of "Line Items" and find the columns by name. Then read Customer PO #, Description, Qty, UOM, Restrictions, Vendor Quotes Received and Delivery Status for the data rows.
4. **Match.** Use the matching rules in `CLAUDE.md`. Sort each quote line into one of these:
   - **Clear match:** one row fits on PO, item, qty, UOM and specs.
   - **Ask:** more than one row could fit, or the specs don't line up.
   - **Not in sheet:** no row fits. Report it, don't write it.
   - **Not a quote:** no stock, can't quote, substitution or partial. Report it.
5. **Approval.** If `CLAUDE.local.md` says `quote-writes: pre-approved`, go to step 6. Otherwise show a table (row, item, current cell, new cell, status change) and stop until the user says OK.
6. **Write each clear match:**
   - Re-read the row's Vendor Quotes Received cell **and its note**, and its Delivery Status cell.
   - Skip if the same vendor and price are already in the cell or the note.
   - **Empty cell:** write `Vendor: $price/UOM`.
   - **Cell already has a quote:** follow "More than one quote on a line" in `CLAUDE.md`.
     - The cheapest quote goes in the cell.
     - Every other quote goes in the cell note, as `Vendor $price/UOM (Q#, m/d)` under `Other quotes:`.
     - If the units differ, don't write. Ask the user instead.
   - If Delivery Status is exactly `No vendor PO yet`, set it to `Quoted, no vendor PO yet`.
7. **Verify.** Re-read every written row, including the note, and confirm all of these:
   - The PO and description still match.
   - The cheapest quote is in the cell.
   - No price was lost.
8. **Log.** Add each handled thread to `processed.json`: `thread_id`, `vendor`, `result` (rows written or why not), `processed` (timestamp), `last_message_id`.
9. **Report.**
   - Nothing new: reply with one short line.
   - Otherwise:
     - A short table of what was written: row, item, price, and whether it's now the lowest or went into the note.
     - Then anything that needs the user's decision.
     - Then quotes that weren't in the sheet.
