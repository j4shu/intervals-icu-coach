---
name: analyze-day
description:
  Concise summary of a day's completed workouts, from a full data analysis.
disable-model-invocation: true
allowed-tools:
  - mcp__icuvisor__*
  - Bash(date *)
  - Bash(jq *)
  - Write(days/*)
---

# Analyze the day's workouts

This skill never writes to intervals.icu and never asks permission mid-run.

Target date: `$ARGUMENTS`, an athlete-local `YYYY-MM-DD`. Empty means the host's
current local date.

icuvisor tools are the `mcp__icuvisor__<tool>` MCP tools. The reference files
live in `${CLAUDE_SKILL_DIR}/references/`; every `references/...` path in them
resolves there.

Read files with the Read tool, never `cat` or `ls`. The only shell commands are
`date` and `jq`, one plain command per call: no `cd`, pipes, loops, or `&&`.
Anything else stops the run for a permission prompt.

## Step 1: Analyze

Read `${CLAUDE_SKILL_DIR}/references/analysis.md` in full and execute its ladder
exactly, yourself. As you go, retain only what the report template needs; do not
narrate raw tool dumps. Read-only on intervals.icu, and do not write a file in
this step.

Done when: every step of the ladder has run and its findings are in hand.

## Step 2: Write the report

Read `${CLAUDE_SKILL_DIR}/references/report.md` in full and fill its skeleton.
It owns the report's shape: the sections, their order, the session headings and
fact lines, and the length caps. Do not improvise a different flow.

Write the filled template to `days/<date>.md`, naming it with the date Step 1
resolved and overwriting any existing report for that date.

Done when: the file exists and follows the template.

## Output

Your final chat message is not the report. It is the file you wrote:

```
Wrote days/<date from Step 1>.md
```
