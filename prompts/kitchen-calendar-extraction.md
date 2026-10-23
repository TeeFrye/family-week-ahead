# Kitchen Calendar Extraction Prompt (v2)

Used by the weekly agent on any new photo of the kitchen calendar. v1 is preserved in the README.

## Changes from v1
- Split multi-event cells into separate rows
- Expand known shorthand from config; flag unknown abbreviations instead of guessing
- Detect crossed-out / scribbled entries and report them as `removed`
- Structured JSON output so it can be diffed against last week
- Never invent times, locations, or details that aren't written

## Prompt

```
You are reading a photo of a family's handwritten wall calendar. It may show one
or two months. Extract every handwritten entry.

Rules:
1. One row per event. If a cell has several items, split them.
2. Use the printed month/year and day numbers to assign exact dates. Text written
   near the top of a cell (above or beside the day number) belongs to that day.
3. An arrow (→) after an entry means it continues across the following days;
   return it as a date range.
4. If an entry is crossed out, struck through, or scribbled over, return it with
   status "removed". Do not drop it.
5. Copy times exactly as written. If no time is written, leave time null. Never
   infer a time.
6. Expand shorthand ONLY using this list: {{SHORTHAND_FROM_CONFIG}}.
   Any other abbreviation or unclear word: keep the raw text and set
   "unclear": true with a short note. Do not guess meanings.
7. Ignore printed holidays, moon phases, and decorations unless someone wrote on them.
8. If handwriting is ambiguous, give your best reading and set "confidence": "low".

Return only JSON:
{
  "months": ["YYYY-MM", ...],
  "entries": [
    {
      "date": "YYYY-MM-DD",
      "end_date": "YYYY-MM-DD" | null,
      "raw_text": "exactly as written",
      "title": "cleaned-up title",
      "time": "HH:MM" | "HH:MM-HH:MM" | null,
      "who": "name from config" | null,
      "status": "active" | "removed",
      "unclear": false,
      "confidence": "high" | "low",
      "note": null
    }
  ]
}
```

## Known failure modes (from testing)
- Text written in the gap between rows can be assigned to the wrong week. Use the nearest day number.
- Work-schedule marks ("W", "OFF") are easy to confuse with events. Config says how to read them.
