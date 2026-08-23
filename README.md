# intervals-icu-coach

Post-workout automation for triathlon training to get an AI analysis of the day.

## Usage

After an indoor ride, run both steps at once:

```
bin/post-ride  # upload-ride, then analyze-day
```

Or run the upload on its own:

```
bin/upload-ride  # TrainerDay -> Garmin -> intervals.icu
```

For any other activity (run, swim, outdoor ride) Garmin already has the data,
so run `bin/analyze-day` on its own.

```
bin/analyze-day
```

## Requirements

`claude`, [`icuvisor`](https://github.com/ricardocabral/icuvisor), [`trainer-day-to-garmin`](https://github.com/j4shu/trainerday-to-garmin)
