# intervals-icu-coach

Post-workout automation for triathlon training to get an AI analysis of the day.

## Usage

After an indoor ride, first run `bin/analyze-day` to upload the activity to intervals.icu:

```
bin/upload-ride  # TrainerDay -> Garmin -> intervals.icu

```

For any other activity (run, swim, outdoor ride) Garmin already has the data,
so run `bin/analyze-day` on its own.

```
bin/analyze-day
```

## Requirements

`uv`, `jq`, `gh`, `claude`, and `icuvisor`.
The intervals.icu API key lives in `~/git/trainerday-to-garmin/.env`.
