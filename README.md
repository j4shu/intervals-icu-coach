# intervals-icu-coach

Post-workout automation for triathlon training to get an AI analysis of the day.

## Usage

Run `/analyze-day [YYYY-MM-DD]` in
[Claude Code](https://code.claude.com). The date defaults to today.

Optional: After an indoor ride, use `trainerday-to-garmin` to upload the
activity to Intervals.icu before running the skill.

## Requirements

[Claude Code](https://code.claude.com),
[`icuvisor`](https://github.com/ricardocabral/icuvisor)

## Layout

The analyze-day skill lives in `.claude/skills/analyze-day`. The `icuvisor` MCP
comes from `.mcp.json` and is configured independently.
