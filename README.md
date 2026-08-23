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

## Telegram

`bin/analyze-day` sends the finished report to a Telegram bot. Create one with
[@BotFather](https://t.me/BotFather), send it a message, then read your chat id from
`https://api.telegram.org/bot<TOKEN>/getUpdates` and export both values:

```
export TELEGRAM_BOT_TOKEN='...'
export TELEGRAM_CHAT_ID='...'
```

These live in `~/.zshrc`, which only interactive shells source. Move them to a file the
script sources directly if you ever run `analyze-day` from launchd or cron.

Without the variables the send is skipped. A failed send warns on stderr but does not
fail the run, since the report is already on disk.
