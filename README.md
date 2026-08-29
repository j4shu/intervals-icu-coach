# intervals-icu-coach

Post-workout automation for triathlon training to get an AI analysis of the day.

## Usage

After an indoor ride, run `bin/upload-ride` to upload the activity to intervals.icu:

```
bin/upload-ride  # TrainerDay -> Garmin -> intervals.icu
```

Then run `bin/analyze-day` to create the report, which is written to `days/<date>.md` and sent to Telegram:

```
bin/analyze-day
```

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
