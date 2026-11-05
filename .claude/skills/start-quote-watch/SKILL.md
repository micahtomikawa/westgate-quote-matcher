---
name: start-quote-watch
description: Start checking the user's inbox for vendor quotes every 30 minutes and writing matches into Westgate Open Lines. Use for /start-quote-watch or "start watching my email for quotes".
---

# Start quote watch

1. **Check the connections first.** Gmail and Google Sheets must be available, and Claude in Chrome should be connected for PDF quotes.
   - If Gmail or Sheets is missing, tell the user which one and how to turn it on (claude.ai → Settings → Connectors), then stop.
   - If Chrome isn't connected, say PDF quotes will be skipped until it is, and continue.
2. **Run `/quote-check` once now.** Use the Skill tool with `quote-check`. If the user hasn't pre-approved writes, this first run shows proposed changes and waits for their OK.
3. **Schedule the repeat.** Invoke the `loop` skill with args `30m /quote-check`.
4. **Tell the user, plainly:**
   - It checks about every 30 minutes, only while this Claude window is open and idle.
   - Each check posts a long instruction message into the chat that looks like it came from them. They don't need to reply to it.
   - It stops by itself after 7 days. Run `/start-quote-watch` again to restart. Say "stop the quote watch" to stop it sooner.
   - To stop being asked before each write, they can create a file named `CLAUDE.local.md` in this folder containing the line `quote-writes: pre-approved`. Offer to create it for them.
