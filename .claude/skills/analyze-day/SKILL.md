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

- Lead with the verdict: how the day went, in one or two sentences.
- Keep the numbers that carry the verdict. Drop the ones that only confirm it.
- Wellness comes next, before the sessions: HRV and resting heart rate against their
  7-day means, sleep duration and quality, and form as TSB with CTL and ATL.
  Say what it means for the day, not only what it read: whether the day's
  load sat well against the state the athlete was in.
- One line per session for swim, bike, and run. Reach for a table only when the day has a
  real comparison in it, such as reps fading across a set or a sport moving against its
  baseline.
- Everything that is not swim, bike, or run goes in a single trailing "Also" line, however
  many such sessions there were: sport, duration, load, and nothing else unless the
  findings flagged something about one. These carry load but do not carry the verdict.
- Every number you keep must appear in the findings. Do not compute, extrapolate, or
  round a missing value into existence. Do not add a metric the findings did not report.
- Carry the findings' caveats through. If the analyst flagged `insufficient_sample`,
  auto-lap intervals, or missing data behind a claim, say so in a clause rather than
  dropping it.
- No tool citations in the summary. The athlete asked how the day went, not where the
  number came from. Name a tool only when the athlete asks for the evidence.

Done when: the summary reads in under a minute and every claim in it traces to the
findings.

## Step 3: Write the report

The summary is a file, not a chat message. Its name must come from the same date the
analysis covers, so resolve the date before naming it.

- If the argument was a `YYYY-MM-DD`, that is the date.
- If it was omitted, call `resolve_calendar_dates` with offset 0 and use the athlete-local
  date it returns.

Write the summary to `days/<date>.md`, overwriting any existing report for that date. The
file holds the summary and nothing else: no preamble, no sign-off, no mention of having
written it.

Done when: the file exists and contains the summary.

## Output

The report file: verdict, then wellness, then one section per swim, bike, or run
session, then the "Also" line. Aim for a screen of text. Markdown headings and short
lines, tables if necessary. No banners, no emojis, no em dashes. End with one specific
next action.

Your final chat message is not the report. It is exactly this one line:

```
Wrote days/<date>.md
```

A caller parses it, so do not reword it, pad it, or add anything after it.
