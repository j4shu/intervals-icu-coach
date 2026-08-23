# intervals-icu-coach

Post-workout automation for triathlon training to get an AI analysis of the day.

## Usage

After an indoor ride, first run `bin/upload-ride` to upload the activity to intervals.icu:

```
bin/upload-ride  # TrainerDay -> Garmin -> intervals.icu
```

For any other activity (run, swim, outdoor ride) Garmin already has the data,
so run `bin/analyze-day` on its own.

```
bin/analyze-day
```

Each report is written to `days/<date>.md` and sent to Telegram.

## Requirements

`claude`, [`icuvisor`](https://github.com/ricardocabral/icuvisor), [`trainer-day-to-garmin`](https://github.com/j4shu/trainerday-to-garmin)

## Configuration

`bin/analyze-day` sources `.env` in the repo root, which is gitignored (override the
path with `COACH_ENV`). Create it mode 600:

```
touch .env && chmod 600 .env
```

```
export INTERVALS_ICU_ATHLETE_ID='...'
export TELEGRAM_BOT_TOKEN='...'
export TELEGRAM_CHAT_ID='...'
```

`INTERVALS_ICU_ATHLETE_ID` reaches the icuvisor MCP server through `.mcp.json` and is
required. The Telegram values are optional: without them the send is skipped, and a
failed send warns on stderr but does not fail the run, since the report is already on
disk.

The file is read by the script rather than the shell, so it works the same from a
terminal, launchd, or cron. It is ignored by git, but it holds live credentials: do not
force-add it, and note that git worktrees do not inherit it, so a worktree run needs its
own copy or `COACH_ENV` pointed at the main checkout.

For the Telegram values, create a bot with [@BotFather](https://t.me/BotFather), send it
a message, then read your chat id from `https://api.telegram.org/bot<TOKEN>/getUpdates`.
