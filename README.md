# intervals-icu-coach

Post-workout automation for triathlon training to get an AI analysis of the day.

## Usage

After an indoor ride, upload the activity to intervals.icu from the `trainerday-to-garmin`
repo, then run the analyze-day skill in a pi session in this repo:

```
# 1. Upload the latest TrainerDay ride (run from the trainerday-to-garmin repo)
cd ~/git/trainerday-to-garmin && uv run main.py

# 2. In a pi session in this repo, run the analyze-day skill, which writes days/<date>.md
#    Invoking the skill from a pi session: trigger it by its name/description, e.g.
#    "run the analyze-day skill".
```

## Requirements

`pi` (with the [`pi-subagents`](https://github.com/nicobailon/pi-subagents) and [`pi-mcp-adapter`](https://github.com/nicobailon/pi-mcp-adapter) packages), [`icuvisor`](https://github.com/ricardocabral/icuvisor), [`trainer-day-to-garmin`](https://github.com/j4shu/trainerday-to-garmin)

## Layout

The analyze-day skill lives in `.pi/skills/analyze-day` and its analyst subagent in
`.pi/agents/analyze-day-analyst.md`. The skill is triggered from an interactive pi session;
icuvisor MCP comes from `.mcp.json` and is configured independently (its own credentials and
athlete id), so this repo needs no `.env`.
