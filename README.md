# Family Week Ahead

**A scheduled AI agent that reconciles a family's scattered schedule sources and emails a Sunday-morning "Week Ahead" briefing, with one-tap links to add anything missing to the shared calendar.**

This repo is built in public. It documents the product thinking as much as the code: what the problem turned out to be, which assumptions broke, and why each design choice was made.

---

## The Problem

### v1 (March 2026): "Get the paper calendar into Google"
Our family's schedule runs on a handwritten calendar in the kitchen. It works for my wife, but I can't see it unless I'm standing in front of it. The first plan was to photograph it and have AI turn it into Google Calendar events.

### v2 (October 2026): "Nobody compares notes"
Before automating anything, I interviewed my user (my wife) and looked at the real data. The problem was bigger than one calendar.

| Assumption (v1) | Reality (v2) |
|---|---|
| The gap is paper → digital | The gap is **five or more sources that never get cross-checked** |
| It's my problem | **We both** miss things: spirit days, tests, schedule changes |
| School info arrives by email | Almost none does. It lives in **ClassDojo push notifications** and **GroupMe** |
| Sports schedules need to be read off the wall | The league publishes **iCal feeds**: structured data, no AI needed |
| Sync events automatically | We want to **review before anything lands** on the shared calendar |
| The output is a calendar | The output is a **weekly briefing**. The calendar is one place it can write to. |

The paper calendar is still the source of truth. It just isn't the only source.

**Problem statement:** *Family schedule information arrives through many channels, each owned by a different person or app, and nobody reconciles them. Things fall through the cracks not because nobody wrote them down, but because nobody compared notes.*

---

## What It Does

Every Sunday morning a cloud agent reads every source, compares them against our shared Google "Family" calendar, and emails a briefing to me with my wife on cc. **It proposes, and we decide.**

```
 Kitchen calendar photo ─┐
 League iCal feeds ──────┤
 Family Inbox (Drive) ───┼──► Reconcile ──► Week Ahead email ──► ➕ one-tap add to Google Calendar
 Family Inbox (Gmail) ───┤        ▲
 Google Family calendar ─┘        │
                          Last week's state (what moved, got crossed out, or was cancelled)
```

**The email:**
1. **🔎 Call-outs**: double-booked times at different places, tight handoffs (under 45 minutes), games that moved or vanished from the feed, crossed-out paper entries still on Google, practices scheduled during school breaks, unknown abbreviations, and a reminder when the kitchen photo is stale
2. **📅 This week**: day by day. Anything not yet on Google has a **➕ Add** link that opens a pre-filled event.
3. **👀 Next week**, then a short **Further out** list (breaks, midterms) up to five weeks ahead
4. **Footer**: which sources were read and how fresh each one is

**Getting information in.** ClassDojo and GroupMe have no usable integration, so there is one habit instead: *if you see it, send it to the Family Inbox.*
- **Screenshots and PDFs** → upload to a Google Drive folder from the Drive app
- **Copied text** → email it to a `+family` Gmail address, where a filter labels it

The agent reads both every Sunday and remembers what it has already processed.

---

## Design Decisions

| Decision | Why |
|---|---|
| **Propose, don't write** | A bad read should never put junk on the calendar my wife relies on. One tap to accept keeps us in control. |
| **➕ for every dated item that's missing, and only those** | Spirit days, tests, deadlines, and paper entries all get an add link so we can choose. The agent re-reads the calendar just before sending, so nothing already added is offered again. |
| **Read feeds directly, don't subscribe** | Google refreshes subscribed feeds slowly. Reading them lets the agent detect *changes* through stable game IDs. |
| **One capture inbox instead of per-app integrations** | One habit covers ClassDojo, GroupMe, flyers, and texts. No API work and no terms-of-service risk. |
| **Images → Drive, text → Gmail** | Tested: the Gmail connector can't read attachments, and Drive OCRs screenshots cleanly. Image-heavy PDFs lose their images, so screenshot those pages. |
| **The photo is optional** | The briefing runs every Sunday regardless. If the kitchen photo is more than 14 days old, it says so in Call-outs. |
| **State facts, no guesses** | No "buy a gift?" or "how many snacks?" when the agent lacks context. Unknown abbreviations are flagged, never interpreted. |
| **Titles lead with the kid's name** | Format: `<Kid> <Activity> <emoji>`, so you can tell whose event it is at a glance. |
| **One state file, replaced weekly** | The agent saves what it saw this week so next week it can report what changed. The connector can't overwrite files, so it creates the new file, reads it back, then trashes the old one. |
| **Private data lives outside this repo** | Names, calendar IDs, feed URLs, and weekly state are in a private Drive folder. This repo holds only the method. |
| **Plain link text in the email** | The first test's buttons were invisible: Gmail strips CSS backgrounds but keeps white text. |

---

## How It Got Here

### ✅ Phase 1: Validate AI extraction (no code)
Claude read a photo of the handwritten calendar: every entry correct, kids kept separate, and an abbreviation ("SB") flagged instead of guessed.

### ✅ Phase 2: Refine the prompt
v2 extraction prompt: [`prompts/kitchen-calendar-extraction.md`](prompts/kitchen-calendar-extraction.md). Tested on two months of real calendar. It read crossed-out entries as removed, matched differently worded duplicates across sources (e.g. "X's Party" ↔ "Colts Game @ X's"), and flagged an unknown abbreviation.

### ✅ Phase 3: Validate every input by hand
| Source | Method | Result |
|---|---|---|
| Kitchen calendar | Photo → vision extraction | ✅ |
| League schedules | iCal feed | ✅ found 8 games missing from Google, including an overlap with a party |
| School PDFs | Drive text extraction | ⚠️ text yes, embedded images no → screenshot those pages |
| Dojo / flyer screenshots | Drive OCR | ✅ full flyer read cleanly |
| Copied messages | Gmail `+family` label | ✅ |

### ✅ Phase 4: Iterate on the briefing with its real readers
Five test sends, each changed by feedback:
1. Buttons were invisible → plain link text
2. A separate "not on Google" section → folded into the day-by-day list with ➕ next to each item
3. Helper suggestions → removed. State only what the sources say.
4. School items had no add link → ➕ is now the default for any dated item missing from Google
5. Items already added still offered ➕ → the agent re-checks the calendar right before sending

The first test also showed that the links work: 10 of 12 proposed events were added from one email.

### 🔄 Phase 5: Run it live
- [x] Agent instructions: [`agent/weekly-briefing.md`](agent/weekly-briefing.md)
- [x] Private config and weekly state file (Drive)
- [x] Scheduled Sunday-morning cloud routine
- [ ] Two weeks of live runs → measure: items caught that we would have missed

### 🅿️ Parked
- **iOS share-sheet shortcut** ([`shortcuts/send-to-family.md`](shortcuts/send-to-family.md)): Google Drive wouldn't load as a save location in the Files app, so for now screenshots go in through the Drive app's upload button. Worth revisiting if uploading becomes the bottleneck.

### 🔮 Later, only if friction justifies it
- A second sport's schedule feed
- GroupMe API (only if the capture habit isn't enough)
- Two-way: reply to the briefing to apply changes

---

## What I'm Learning

- **Discovery beats building.** The v1 plan was a well-built answer to the wrong question. Twenty minutes of questions and one look at real data reframed it.
- **Validate inputs before automating.** Every source got a manual test before anything was scheduled. Two of them (Gmail attachments, PDF images) failed in ways that changed the design.
- **The best integration is sometimes a habit.** One capture rule beat four app integrations.
- **Keep humans in the loop where trust is fragile.** A shared family calendar is high-trust, so proposals beat automation there.
- **Ship small and iterate on real feedback.** The email went through five versions in one evening, each from a concrete reaction by its actual reader.
- **Cut what doesn't work quickly.** The shortcut hit a platform wall. A slightly clunkier path that works beats a slick one that doesn't.

---

## Repo Layout

```
agent/weekly-briefing.md                 # what the Sunday agent does, step by step
agent/config.example.json                # shape of the private config (no real data)
prompts/kitchen-calendar-extraction.md   # v2 photo-extraction prompt
shortcuts/send-to-family.md              # iOS share-sheet shortcut (parked)
```
