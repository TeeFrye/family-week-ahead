# Kitchen Calendar → Google Calendar

**A documented experiment in solving a real family problem with the right amount of tool.**

---

## The Problem (v1, March 2026)

Our family's schedule — three kids, sports practices, school events, birthday parties, dinner plans — runs on a physical calendar in the kitchen. It works better for my wife than for me. The breakdown happens on my end: I can't see it unless I'm standing in front of it, so I constantly feel behind on what's coming, what needs coordinating, and where I can actually help.

The goal isn't to change how she works. The goal is to bridge the gap between her analog system and my digital one — without adding anything to her plate.

---

## What Changed: The Problem (v2, October 2026)

I came back to this after Phase 1 and did what I should have done more of the first time: I interviewed my user (my wife), and then I looked at the actual data instead of my assumptions about it.

**What I learned:**

| Assumption (v1) | Reality (v2) |
|---|---|
| The gap is paper → digital | The gap is **5+ sources that never get cross-checked** |
| It's my problem | **We both** miss things — spirit days, "bring money for X," schedule changes |
| School info arrives by email | Almost none does. It lives in **ClassDojo push notifications** and **GroupMe** |
| Sports schedules need to be read off the wall | The league publishes **iCal feeds** — structured data, no AI needed |
| Sync events automatically | We want to **review before anything lands** on the shared calendar |
| The output is a calendar | The output is a **weekly briefing** — the calendar is just one place it writes to |

The paper calendar is still the source of truth. It's just not the only source.

**v2 problem statement:** *Family schedule information arrives through many channels, each owned by a different person or app, and nobody reconciles them. Things fall through the cracks not because nobody wrote them down, but because nobody compared notes.*

---

## The v2 Solution: A Sunday-Morning Briefing Agent

A scheduled AI agent runs every Sunday morning, reads every source, reconciles them against our shared Google Family calendar, and emails a **Week Ahead** briefing. It **proposes**; we decide.

```
 Kitchen calendar photo ─┐
 League iCal feeds ──────┤
 Family Inbox (Drive) ───┼──► Reconcile ──► Week Ahead email ──► ➕ one-tap add to Google Calendar
 Family Inbox (Gmail) ───┤        ▲
 Google Family calendar ─┘        │
                          Last week's state (diffs: moved, crossed out, removed)
```

**The briefing has three parts:**
1. **🔎 Call-outs** — overlaps, tight handoffs, likely-cancelled events, unclear abbreviations, stale inputs
2. **📅 This week** — day by day; anything not yet in Google gets a ➕ link that opens a pre-filled event
3. **👀 Next week** — heads-up on what's coming

**The capture habit (the part that actually solves "we missed spirit day"):**
ClassDojo and GroupMe have no usable integration path. Instead of fighting that, there's one rule: *if you see it, send it to the Family Inbox.* A share-sheet shortcut saves any screenshot to a Drive folder in one tap; copied text can be emailed to a `+family` Gmail address. The agent reads both every Sunday.

### Design decisions

| Decision | Why |
|---|---|
| **Propose, don't write** | A bad OCR read should never put junk on the calendar my wife relies on. One tap to accept keeps us in control. |
| **Read feeds directly, don't subscribe** | Google refreshes subscribed feeds slowly and they only show for the subscriber. Reading them lets the agent detect *changes* via stable game IDs. |
| **Universal capture inbox over per-app integrations** | One habit covers ClassDojo, GroupMe, flyers, and texts. Zero API work, zero ToS risk. |
| **Images → Drive, text → Gmail** | Tested: the Gmail connector can't read attachments; Drive OCRs screenshots cleanly. |
| **Photo is optional** | The briefing runs every Sunday regardless. If the kitchen photo is stale, it says so in Call-outs. |
| **No guessing** | The agent states what the sources say. No "buy a gift?" or "how many snacks?" when it lacks context. |
| **One state file, replaced weekly** | The agent saves what it saw this week (paper entries, game IDs, inbox items it has already read) so next week it can report what changed. The Drive connector can't overwrite files, so it creates the new file, confirms it can read it back, and then trashes the old one. Only one copy is kept. |
| **Private config lives outside this repo** | Names, calendar IDs, feed URLs and weekly state live in a private Drive folder. This repo holds only the method. |

---

## Experiment Phases

### ✅ Phase 1: Validate the AI extraction (Claude.ai — no code)

**Goal:** Confirm that Claude can reliably read a photo of a handwritten kitchen calendar and extract structured events.

**Results:**
- Handwriting: **read reliably** — all 8–9 entries correct.
- Multiple kids: **kept separate** correctly.
- Ambiguity: **flagged "SB" (Spring Break)** rather than guessing; split a 3-item cell into 3 events.

**Learnings that fed v2:** flag abbreviations, split multi-event cells, ask rather than guess, use structured output.

### ✅ Phase 2: Refine the prompt
v2 extraction prompt in [`prompts/kitchen-calendar-extraction.md`](prompts/kitchen-calendar-extraction.md). Re-tested on two months of real calendar (Oct 2026): correctly read crossed-out entries as removed, matched differently-worded duplicates across sources (e.g. "X's Party" ↔ "Colts Game @ X's"), and surfaced an unknown abbreviation instead of inventing a meaning.

### ✅ Phase 3: Validate every input, by hand
Before automating anything, each source was tested end to end in one session:

| Source | Method | Result |
|---|---|---|
| Kitchen calendar | Photo → vision extraction | ✅ |
| League schedules | iCal feed | ✅ found 8 games missing from Google, including an overlap with a party |
| School PDFs | Drive text extraction | ⚠️ text yes, embedded images no → screenshot those pages |
| Dojo / flyer screenshots | Drive OCR | ✅ full flyer read cleanly |
| Copied messages | Gmail `+family` label | ✅ |
| Briefing email | Two test sends, iterated on format | ✅ (v1 buttons were invisible — Gmail strips CSS backgrounds) |

### 🔄 Phase 4: Automate
- [x] Agent instructions — [`agent/weekly-briefing.md`](agent/weekly-briefing.md)
- [x] Private config + weekly state file (Drive)
- [x] "Send to Family" iOS share-sheet shortcut — [`shortcuts/send-to-family.md`](shortcuts/send-to-family.md)
- [ ] Scheduled Sunday-morning run
- [ ] Two weeks of live runs → measure: items caught that we'd have missed

### 🔮 Later, only if friction justifies it
- GameChanger feed for a second sport
- GroupMe API (only if the capture habit isn't enough)
- Two-way: let a reply to the briefing email apply changes

---

## What I'm Learning

- **Discovery beats building.** The v1 plan was a well-built answer to the wrong question. Twenty minutes of questions and one look at real data reframed it.
- **Validate inputs before automating.** Every source got a manual test before any scheduling. Two of them (Gmail attachments, PDF images) failed in ways that changed the design.
- **The best integration is sometimes a habit.** A one-tap capture flow beat four app integrations.
- **Keep humans in the loop where trust is fragile.** A shared family calendar is high-trust; proposals beat automation there.
- **Ship the smallest thing that works, then iterate on real feedback.** The briefing format went through two rounds of edits from its actual reader.

---

## Repo Layout

```
agent/weekly-briefing.md                 # what the Sunday agent does, step by step
agent/config.example.json                # shape of the private config (no real data)
prompts/kitchen-calendar-extraction.md   # v2 photo-extraction prompt
shortcuts/send-to-family.md              # iOS share-sheet shortcut setup
```
