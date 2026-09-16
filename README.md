# intervals-icu-coach

Post-workout automation for triathlon training to get an AI analysis of the day.

## Usage

Run the `/analyze-day` skill in [pi](https://pi.dev).

Optional: After an indoor ride, use `trainerday-to-garmin` to upload the
activity to Intervals.icu before running the skill.

## Requirements

`pi` (with [`pi-mcp-adapter`](https://github.com/nicobailon/pi-mcp-adapter)),
[`icuvisor`](https://github.com/ricardocabral/icuvisor)

## Layout

The analyze-day skill lives in `.pi/skills/analyze-day`. The `icuvisor` MCP
comes from `.mcp.json` and is configured independently.
