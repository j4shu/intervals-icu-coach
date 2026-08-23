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

`claude`, [`icuvisor`](https://github.com/ricardocabral/icuvisor), [`trainer-day-to-garmin`](https://github.com/j4shu/trainerday-to-garmin), Telegram

## Configuration

```
touch .env && chmod 600 .env
```

```
export INTERVALS_ICU_ATHLETE_ID='...'
export TELEGRAM_BOT_TOKEN='...'
export TELEGRAM_CHAT_ID='...'
```
