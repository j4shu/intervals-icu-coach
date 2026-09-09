---
name: analyze-day
description: Concise summary of a day's completed workouts, from a full data analysis.
disable-model-invocation: true
---

# Analyze the day's workouts

This skill never writes to intervals.icu and never asks permission mid-run.

Argument: an optional athlete-local `YYYY-MM-DD` named in the invoking prompt. Omitted
means today.

You run the data ladder and write the report yourself. Read
`.pi/skills/analyze-day/references/analysis.md` in full and execute its ladder exactly,
retaining only what the report template needs. Then fill
`.pi/skills/analyze-day/references/report.md`.

icuvisor tools run behind the `mcp` tool. Call `mcp` with the icuvisor tool name and its
arguments, adding `server: "icuvisor"` to disambiguate. Call `mcp` with
`instructions: "icuvisor"` only if you need the server usage guide.

## Step 1: Analyze

Read `.pi/skills/analyze-day/references/analysis.md` in full and execute its ladder
exactly. As you go, retain only what the report template needs; do not narrate raw tool
dumps. Read-only on intervals.icu, and do not write a file in this step.

Done when: every step of the ladder has run and its findings are in hand.

## Step 2: Write the report

The report is a file, not a chat message. Its name must come from the same date the
analysis covers, so resolve the date before naming it.

- If the argument was a `YYYY-MM-DD`, that is the date.
- If it was omitted, call `mcp` with `tool: "resolve_calendar_dates"` and
  `args: { "offsets": [0] }`, and use the athlete-local date it returns.

Read `.pi/skills/analyze-day/references/report.md` in full and fill its skeleton. It owns
the report's shape: the sections, their order, the session headings and fact lines, and
the length caps. Do not improvise a different flow.

Write the filled template to `days/<date>.md`, overwriting any existing report for that
date.

Done when: the file exists and follows the template.

## Output

The report file is the filled `references/report.md` template.

Your final chat message is not the report. It is exactly this one line:

```
Wrote days/<date>.md
```

A caller parses it, so do not reword it, pad it, or add anything after it.
