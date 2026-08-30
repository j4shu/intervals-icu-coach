---
name: analyze-day
description: Concise summary of a day's completed workouts, synthesized from a full subagent analysis.
disable-model-invocation: true
---

# Analyze the day's workouts

Read-only. This skill never writes to intervals.icu and never asks permission mid-run.

Argument: an optional athlete-local `YYYY-MM-DD`. Omitted means today.

You do not fetch or compute anything yourself. A subagent runs the full tool ladder and
reports its numbers back; your job is to read those findings and write the athlete a short,
readable summary.

## Step 1: Spawn the analyst

One `Agent` call, `subagent_type: general-purpose`, with this prompt, `<date>` replaced by
the argument or the literal word `today`:

```
Analyze the athlete's training day: <date>.

Read .claude/skills/analyze-day/references/analysis.md in full and execute its ladder
exactly, then return its findings report as your final message. Read-only: never write to
intervals.icu, never write a file.
```

Do not narrow the ladder, do not pre-fetch data for it, and do not run a second agent.

Done when: the agent has returned its findings.

## Step 2: Synthesize

The findings are dense on purpose. Compress them to be human-readable.

- Keep the numbers that carry the verdict. Drop the ones that only confirm it.
- Read the wellness numbers, do not just report them. Say whether the day's load sat well
  against the state the athlete was in.
- Sessions outside swim, bike, and run carry load but do not carry the verdict. Say no
  more about one than the findings flagged.
- Every number you keep must appear in the findings. Do not compute, extrapolate, or
  round a missing value into existence. Do not add a metric the findings did not report.
- Carry the findings' caveats through rather than dropping them.
- No tool citations in the report. The athlete asked how the day went, not where the
  number came from. Name a tool only when the athlete asks for the evidence.

Done when: every claim you intend to write traces to the findings.

## Step 3: Write the report

The report is a file, not a chat message. Its name must come from the same date the
analysis covers, so resolve the date before naming it.

- If the argument was a `YYYY-MM-DD`, that is the date.
- If it was omitted, call `resolve_calendar_dates` with offset 0 and use the athlete-local
  date it returns.

Read `.claude/skills/analyze-day/references/report.md` in full and fill its skeleton. It
owns the report's shape: the sections, their order, the session headings and fact lines,
and the length caps. Do not improvise a different flow.

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
