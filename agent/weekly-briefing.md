# Weekly Briefing Agent

Runs every Sunday morning. Reads every family schedule source, reconciles them against the shared Google Family calendar, and emails a **Week Ahead** briefing. It proposes changes. It does not write to the calendar on its own.

Tools it needs: Google Calendar, Gmail, Google Drive, and HTTP fetch (for iCal feeds).

---

## 0. Load config and state
1. In the private Drive `_system` folder, read `briefing-config.json`.
2. Read the newest `briefing-state-YYYY-MM-DD.json`. That's last week's picture of the world.
3. The briefing window is **this week**, Sunday through Saturday, starting today. The **heads-up window** is the following 7 days. School items and far-out notable dates (breaks, midterms) can be called out up to 5 weeks ahead.

## 1. Gather sources
| Source | How | Notes |
|---|---|---|
| Google Family calendar | List events, today → +5 weeks | This is the baseline that everything gets compared to. |
| League iCal feeds | Fetch each `team_feeds` URL and parse VEVENTs | Each game has a stable `UID` (e.g. `Game-12345678`) and a `LAST-MODIFIED`. |
| Kitchen calendar photo | Newest image in the Drive inbox that shows a calendar grid | Extract with `prompts/kitchen-calendar-extraction.md`. If there's no new photo, reuse `state.paper`. |
| Drive Family Inbox | Files not in `state.processed_inbox.drive_file_ids` | Screenshots and PDFs. If a PDF page comes back nearly empty, it's an image. List it under "couldn't read". |
| Gmail Family Inbox | `gmail_query`, skipping already-processed message IDs | Copied Dojo and GroupMe text. Attachments can't be read here, so ask for them to go to Drive. |

## 2. Extract school and message items
From inbox items, pull out anything with a date or a day name. Pay special attention to:
- **Wear or bring** items: spirit days, dress-up days, money, supplies, snacks
- **Deadlines**: forms, book choices, projects
- **Tests**, field trips, no-school days, early dismissals

Use config to assign each item to a kid: by teacher name, by grade (e.g. spirit-week grade colors), or "all".

## 3. Reconcile
For each item from paper, feeds, and inbox, find a matching Google event:
- Same date, and the titles mean the same thing. Use judgment: "Sam's Party" matches "Football Game @ Sam's".
- Feed games match by `feed-uid` in the event description.
- Recurring practices count as matches for the paper's "K1&K2 practice"-style entries.

Then classify each item:
| Result | What happens in the briefing |
|---|---|
| Matched | Listed normally |
| Missing from Google | Listed with a ➕ link. This is the default for **every** dated item from any source, school items included (spirit days, tests, deadlines). Items with no specific date ("this week") get no link. |
| Feed game moved or changed (time, place, or `LAST-MODIFIED` vs. state) | Call-out: "moved from X to Y" |
| Feed game gone from feed but on Google | Call-out: "possibly cancelled" |
| Paper entry now `removed`, or gone since last week | Call-out: "crossed out on paper but still on Google" |
| On Google during a no-school break (practices) | Call-out: "still scheduled during break" |

## 4. Call-outs (top of email)
Only flag things that need a decision or a look:
- Overlaps, and back-to-back events at different locations
- Tight handoffs (under 45 minutes between events)
- Changes and cancellations (see step 3)
- Unknown abbreviations, in the week they fall
- Wear or bring items for this week, short, one line per day
- **Kitchen photo freshness.** If the newest photo is older than `photo_stale_days`, or the paper has no entries past the heads-up window: "📸 Kitchen calendar photo is N days old — snap a new one to Family Inbox."
- Anything in the inbox that couldn't be read

## 5. Write the email
**To:** owner · **Cc:** partner · **Subject:** `Week Ahead: <Mon D> – <Mon D>` (add `· N to add` if any ➕ items)

Sections, in order:
1. **🔎 Call-outs**
2. **📅 This week.** Day by day. Each line: time · title · place. If it's missing from Google, put a ➕ link right after it. List the partner's work days here only for weekends and holidays.
3. **👀 Next week**, more briefly
4. Footer: the sources used and when each was last updated

**Rules:**
- State what the sources say. **No suggestions where context is missing** (no "buy a gift?", no "how many snacks?").
- Event titles start with the kid's name: `<Kid> <Short activity> <emoji>`.
- Use the special handling for names listed in config `shorthand`.
- **Email HTML:** Gmail strips CSS backgrounds, so use plain colored link text and never buttons with white text. Write a plain-text alternative too.

**➕ link format** (Google Calendar template URL):
```
https://calendar.google.com/calendar/render?action=TEMPLATE
  &text=<title>&dates=<YYYYMMDD/YYYYMMDD for all-day | YYYYMMDDTHHMMSS/YYYYMMDDTHHMMSS>
  &details=<source + details>&location=<place>&ctz=<timezone>&src=<family_calendar_id>
```
All-day end dates are exclusive. URL-encode every value.

## 6. Save state
Create a new `briefing-state-<today>.json` in `_system`:
- `paper`: the latest extraction, or last week's if there's no new photo
- `last_kitchen_photo`: its date and Drive file ID
- `feed_games`: UID → {start, end, location, last_modified, google_event_id if it was added}
- `school_items`: still-upcoming items, including any not yet on the calendar
- `processed_inbox`: every Gmail and Drive ID read so far
- `open_questions`: anything flagged and not yet resolved

Once the new file has been created **and read back successfully**, trash the previous state file so only one remains. (The Drive connector can't overwrite a file, so create-then-trash is how it gets replaced.) If creating the new file fails, keep the old one.

## Never
- Create, edit, or delete calendar events on its own
- Put family names, IDs, URLs, or schedule data in the public repo
- Guess times, locations, or what an abbreviation means
