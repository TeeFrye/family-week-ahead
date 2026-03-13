# Kitchen Calendar → Google Calendar

**A documented experiment in solving a real family problem with the right amount of tool.**

---

## The Problem

Our family's schedule — three kids, sports practices, school events, birthday parties, dinner plans — are run on a physical calendar in the kitchen. It works better for her than me. The breakdown happens on my end: I can't see it unless I'm standing in front of it, which means I constantly feelbehind on what's coming, what needs coordinating, and where I can actually help.

The goal isn't to change how she works. The goal is to bridge the gap between her analog system and my digital one — without adding anything to her plate.

---

## What I Considered Building

Before writing a line of code, I mapped out the full solution space:

| Option | What it is | Effort | Tradeoff |
|---|---|---|---|
| **Just use Claude.ai** | Upload photo → extract events → add to calendar manually | None | Manual each time, but works today |
| **iOS Shortcut** | One-tap automation on my phone that calls Claude API + Google Calendar | Low–Medium | No code, lives on my phone, feels like a real tool |
| **Web App (GitHub Pages)** | Full UI with photo upload, event review, Google Calendar push via OAuth | High | Overkill for the actual problem |
| **Custom Claude Integration** | A prompt + Google Calendar MCP already connected in Claude.ai | None | Already possible without building anything |

**Decision:** Start with the simplest thing that works. Graduate to more complexity only if the friction justifies it.

---

## Why This Repo Has No Code (Yet)

This is a decision log and experiment record, not a code project. That's intentional.

One of the most important product skills is knowing when *not* to engineer something. A well-scoped solution to a real problem — even if it's just a repeatable prompt — is worth documenting publicly. This repo tracks the thinking, the iterations, and what actually worked.

---

## Experiment Phases

### ✅ Phase 1: Validate the AI extraction (Claude.ai — no code)

**Goal:** Confirm that Claude can reliably read a photo of a handwritten kitchen calendar and extract structured events before investing any build time.

**How:**
1. Take a photo of the kitchen calendar
2. Upload to Claude.ai
3. Use the prompt below
4. Review accuracy of extracted events

**Prompt v1:**
```
Here's a photo of our family's kitchen calendar. Please extract all events you can read and format them as a list like this:

- [Date] | [Day] | [Time if visible] | [Event] | [Who it's for]

If a date or time isn't clear, note that. If something is ambiguous, include your best read and flag it.
```

**What I'm measuring:**
- Can it read handwriting accurately? - **Yes, reliably. Got all 8–9 handwritten entries correctly.**
- Does it handle multiple kids' events without mixing them up? - **Yes, kept Kid1 and Kid2 musical together, Kid1 field trip separate, friends birthday separate.**
- How does it handle partial dates or unclear writing? - **Flagged "SB" (short-hand for Spring Break) correctly rather than guessing. April 17 had three items in one cell and extracted all three.**

**Status:** [ ] Not started | [ ] In progress | [x] Done  
**Notes:** *(add observations here as you test)*

---

### 🔄 Phase 2: Refine the prompt

**Goal:** Improve extraction quality based on Phase 1 findings. Likely iterations:
- Better handling of recurring events
- Asking for output in a specific format for easier copy/paste
- Testing with different calendar layouts or lighting conditions

**Prompt iterations will be tracked in `/prompts/`**
- Add instruction to flag abbreviations and to not always include them as-is
- Add instruction to split multi-event cells into separate rows
- Add insturction to ask questions, confirm start and end times, not to guess, etc
- Possibly ask for output as JSON for the Shortcut phase

---

### 📱 Phase 3: Build the iOS Shortcut

**Goal:** One tap on my phone → photo of calendar → events added to Google Family Calendar automatically.

**How it works (high level):**
1. Shortcut triggers → prompts me to select or take a photo
2. Photo is sent to Claude API with the extraction prompt
3. Claude returns structured event data (JSON)
4. Shortcut parses the response and creates calendar events via the Google Calendar API or Apple Calendar

**What I'll need to learn:**
- How to call an external API from iOS Shortcuts (using the "Get Contents of URL" action)
- How to pass an image as base64 to the Claude API
- How to parse a JSON response inside Shortcuts
- Whether to target Apple Calendar (easier) or Google Calendar (preferred)

**Decision point:** Apple Calendar is significantly simpler to write to from Shortcuts (no OAuth required — it's a native action). Google Calendar requires API credentials. Will start with Apple Calendar and evaluate.

**Resources to explore:**
- [Anthropic API docs — vision/image input](https://docs.anthropic.com)
- iOS Shortcuts "Get Contents of URL" action
- Apple Calendar Shortcut actions

---

### 🔮 Phase 4 (Optional): Web app

Only worth building if:
- The iOS Shortcut breaks down in some way
- My wife wants to use it too (needs a browser UI)
- I want the portfolio artifact specifically

Likely stack if built: Single-file HTML + Claude API + Google Calendar OAuth. Same pattern as my other GitHub Pages projects.

---

## What I'm Learning

- How to scope a problem before picking a solution
- Claude API image input (vision)
- iOS Shortcuts as lightweight automation without code
- When a prompt is the right tool and when code actually earns its complexity

---

## Prompts

*Tracked in `/prompts/` as they evolve.*

---
