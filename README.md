# intervals-icu-coach

Post-workout automation for triathlon training to get an AI analysis of the day.

## Usage

Run the `/analyze-day` skill in [pi](https://pi.dev):

Optional: After an indoor ride, use `trainerday-to-garmin` to upload the activity to Intervals.icu.

## Requirements

`pi` (with the [`pi-subagents`](https://github.com/nicobailon/pi-subagents) and [`pi-mcp-adapter`](https://github.com/nicobailon/pi-mcp-adapter) packages), [`icuvisor`](https://github.com/ricardocabral/icuvisor)

## Layout

The analyze-day skill lives in `.pi/skills/analyze-day` and its analyst subagent in
`.pi/agents/analyze-day-analyst.md`. The skill is triggered from an interactive pi session;
icuvisor MCP comes from `.mcp.json` and is configured independently.
