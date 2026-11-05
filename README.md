# Westgate Quote Matcher

Claude watches **your** Westgate inbox for vendor quotes and writes the prices into the **Westgate Open Lines** sheet ("Line Items" tab) for you.

What it does on each check:
- Finds new quote emails in your inbox, including PDF quotes.
- Matches each quoted line to the right row by Customer PO, item and quantity.
- Adds the price to **Vendor Quotes Received**, e.g. `Elite Supply: $231.38/EA`.
- Changes **Delivery Status** from `No vendor PO yet` to `Quoted, no vendor PO yet`. That turns the row **yellow**: orange means no quote, yellow means quoted but not ordered, and white means ordered.
- Adds **flags** to the cell note, such as a quote above our sell price or a spec the vendor didn't confirm.
- When several vendors quote the same line, it keeps the **cheapest** price in the cell. Every other quote goes into a **note** on that cell (hover over the cell to see it), so no price is ever lost.
- Asks you when a match isn't clear, the specs don't line up, or the units don't match. It never touches any other sheet.

Setup takes about 15 minutes, once.

---

## 1. Get the things it needs

1. **Claude Code desktop app.** Download it from claude.ai/download, install it, and sign in with your Westgate Claude account.
2. **Connectors**, so Claude can read your email and edit the sheet. In a web browser, go to **claude.ai → Settings → Connectors** and connect each of these with **your own Westgate Google account**:
   - **Gmail**
   - **Google Drive**
   - **Google Sheets**
3. **Claude in Chrome extension**, used to read PDF quotes. Install it from the Chrome Web Store (search "Claude in Chrome"), then sign in. Keep a Chrome window open that's logged into your Westgate Gmail.
4. **Sheet access.** Open the Westgate Open Lines sheet in your browser and make sure you can edit it. If you can only view it, ask whoever owns the sheet for edit access.

## 2. Download this folder

1. Accept the GitHub invite in your email. You'll need a free GitHub account.
2. Install **GitHub Desktop** from desktop.github.com and sign in.
3. In GitHub Desktop: **File → Clone Repository**, pick `westgate-quote-matcher`, then click **Clone**. Remember where it saves the folder (usually `Documents/GitHub/westgate-quote-matcher`).

## 3. Start it

1. Open the Claude Code app and choose **Open folder**. Pick the `westgate-quote-matcher` folder.
2. Type `/start-quote-watch` and press Enter.
3. If Claude asks to allow a tool or connector, click **Allow**.
4. Claude checks your inbox right away. **The first time**, it shows you the changes it wants to make and waits for your OK.
   - Look the list over and reply "yes" if it's right.
   - When you trust it, tell Claude: *"create my CLAUDE.local.md with quote-writes: pre-approved"*. After that it writes clear matches without asking and tells you what it wrote.

## 4. Things to know

- **It checks about every 30 minutes, only while the Claude window is open.** If you close the app, checking stops. Reopen the folder and run `/start-quote-watch` again.
- **It turns itself off after 7 days.** Run `/start-quote-watch` again to restart.
- **You'll see a long message every 30 minutes** starting with "Check the user's Gmail..." or a `/quote-check` line. That's the timer telling Claude to check. It's not you, and you don't need to answer.
- **To check right now:** type `/quote-check`.
- **To stop:** type "stop the quote watch".
- **Want to double-check a change?** In the sheet, use **File → Version history**, or ask Claude "list the rows you changed today".
- `processed.json` in this folder is your personal log of emails already handled. Don't delete it, or Claude will re-check old emails. It skips duplicates, so nothing gets written twice.

## Getting updates

When there's a new version, open GitHub Desktop, select this repository, and click **Fetch origin**, then **Pull**. Your personal files (`processed.json`, `CLAUDE.local.md`) are never overwritten.
