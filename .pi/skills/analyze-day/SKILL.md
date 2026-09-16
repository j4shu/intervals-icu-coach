---
name: analyze-day
description:
  Concise summary of a day's completed workouts, from a full data analysis.
disable-model-invocation: true
---

# Analyze the day's workouts

This skill never writes to intervals.icu and never asks permission mid-run.

Argument: an optional athlete-local `YYYY-MM-DD` named in the invoking prompt.
Omitted means the host's current local date.

icuvisor tools run behind the `mcp` tool. Call `mcp` with the icuvisor tool name
and its arguments, adding `server: "icuvisor"` to disambiguate. Call `mcp` with
`instructions: "icuvisor"` only if you need the server usage guide.

## Step 1: Analyze

Read `.pi/skills/analyze-day/references/analysis.md` in full and execute its
ladder exactly, yourself. As you go, retain only what the report template needs;
do not narrate raw tool dumps. Read-only on intervals.icu, and do not write a
file in this step.

Done when: every step of the ladder has run and its findings are in hand.

## Step 2: Write the report

Read `.pi/skills/analyze-day/references/report.md` in full and fill its
skeleton. It owns the report's shape: the sections, their order, the session
headings and fact lines, and the length caps. Do not improvise a different flow.

Write the filled template to `days/<date>.md`, naming it with the date Step 1
resolved and overwriting any existing report for that date.

Done when: the file exists and follows the template.

## Output

Your final chat message is not the report. It is the file you wrote:

```
Wrote days/<date from Step 1>.md
```
