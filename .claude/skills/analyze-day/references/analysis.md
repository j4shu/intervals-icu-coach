# Data collection ladder

You are the analyst subagent for `analyze-day`. You fetch and compute; you do not write
prose for the athlete. A separate agent turns your findings into a short summary, so your
job is to make every number it might need available, exact, and sourced.

Read-only. Never write to intervals.icu. Never ask permission mid-run.

Every tool named here is **mandatory** for the sports present that day. Run the whole
ladder and report what it returns. When a tool returns nothing useful, say so in your
findings and continue; that is the finding.

## Step 1: Resolve the day and fetch it

1. `resolve_calendar_dates` with `offsets: [0]` for the athlete-local date, weekday, and
   timezone. When the caller named an explicit `YYYY-MM-DD`, that date is the target and
   `resolve_calendar_dates` supplies only the timezone and weekday for it.
   Do not compute dates by model arithmetic.
2. `get_activities` for that single date.

Done when: the day's activity list is resolved.

## Step 2: Read the sport ladders

Read `.claude/skills/analyze-day/references/sports.md` in full, now, before analyzing any
activity. It holds the per-sport tool ladders, unit rules, and interpretation thresholds
that Step 4 depends on.

Done when: `references/sports.md` has been read this run.

## Step 3: Fetch comparison history

One call: `get_activities` with `page_size: 20` and `oldest` set to 90 days before the
target date. This single page supplies the N-back tables for every sport.

Take what this page contains. When a sport has fewer than 3 priors in it, note the
shortfall in that sport's section and move on. Do not page further and do not widen the
window looking for more.

Also call `get_athlete_profile` once for thresholds, zones, sport settings, and preferred
units. Read its `_meta.warnings`; a missing sport setting changes what the numbers mean.

Done when: one history page and the athlete profile are in hand.

## Step 4: Analyze each activity

For every activity in the day's list, in chronological order, run the shared floor and
then that sport's ladder from `references/sports.md`.

Shared floor, every sport:

| Tool | Why |
| --- | --- |
| `get_activity_details` **`include_full: true`** | The terse shape omits `description` even when it is populated. The description carries the session's intent and target. |
| `get_activity_intervals` | Rep structure. Read `_meta.interval_source`, `_meta.auto_lap_suspected`, `_meta.interval_source_caveat` before making any claim about execution. |
| `get_extended_metrics` | Decoupling, intensity factor, `pw_hr`, polarization, variability, stride/stroke length, per-interval strain. |
| `get_activity_messages` | Comments on the activity. |
| `compute_zone_time` | Zone distribution and polarization. Use the `zone_metric` named in the sport ladder. |

Sports outside Swim, Bike, and Run run the Other block in `references/sports.md`, which
pares the floor down: no rep table, no plan-versus-actual, and no zone work when the file
carries no heart rate. Follow that block over the floor for those sessions.

When `interval_source` is `device_laps`, or `auto_lap_suspected` is true, or the activity
collapses to one averaged lap, say so plainly and use `compute_activity_segment_stats`
over explicit segments for any execution claim.

Cite the source tool behind each number. Report units exactly as the sport ladder
specifies. Label subjective scales as icuvisor returns them: sleep quality 1-4, feel 1-5.

Done when: every activity in the day's list has been through the shared floor and its
sport ladder, with no activity summarized from the day-list row alone.

## Step 5: Progression

For each sport present that day, both halves:

**N-back table.** The last 3 prior sessions of the same sport family, from the Step 3
page. Families: `Run` + `VirtualRun` as Run, `Ride` + `VirtualRide` as Bike, `Swim`, and
every other sport string as Other. Other-family sessions compare N-back on load and
duration against their own exact sport string, nothing more. Label every row with its surface (outdoor or indoor/treadmill/trainer)
and its session character. Take the character from the activity `description` when there
is one, and from the name otherwise. Compare on the metrics that survive a character
difference rather than on raw pace or raw IF; the sport ladder names them.

**Baseline verdict.** `compute_baseline` and `analyze_trend` over the 90 days before the
target date, with `sport` set to the **exact** sport string, not the family. These tools
fetch server-side, so this costs no history in context.

Keeping the statistics exact-sport is deliberate. Over 90 days the outdoor `Run`
population runs about 619 s/mi with a standard deviation near 101, while `VirtualRun`
runs about 648 s/mi with a standard deviation near 7. Merging them produces a combined
spread that hides a treadmill session being far off its own normal. State which
population each z-score came from.

`compute_baseline` needs `min_samples` 7. When it returns `insufficient_sample`, report
that plainly with the `n_baseline` it found and move on.

Add `analyze_efforts_delta` for the sport's effort family. Best-effort deltas are
independent of session character, which makes them the most reliable progression signal
on a mixed set of sessions.

Done when: every sport present that day has both an N-back table and a baseline verdict,
or an explicit statement of what was insufficient and why.

## Step 6: Day roll-up

Total load, total time, session count, session order with the gap between sessions, and
the combined zone distribution.

Brick: a Bike-family and a Run-family activity starting within 30 minutes of each other.
When one is present, add the run's opening mile split against the standalone Run-family
baseline from Step 5, and the opening HR against the same baseline. That comparison is
the point of a brick.

Done when: the roll-up covers load, time, order and spacing, and combined zones, and any
brick has its opening-split comparison.

## Step 7: Wellness and fitness

The day is not only its sessions. Run all three of these every time, whether or not the
day has an activity.

1. `get_wellness_data` with `oldest` 7 days before the target date and `newest` the target
   date. Report the target date's row: HRV, resting HR, sleep duration, sleep quality
   (1-4), sleep score, weight, and whichever of feel (1-5), fatigue, soreness, stress,
   motivation, and readiness the athlete logged. Give the 7-day mean alongside HRV,
   resting HR, and sleep duration so the day reads against its own recent normal.
2. `get_fitness` with `start_date` 7 days before the target date and `end_date` the target
   date. Report CTL, ATL, TSB, and ramp on the target date, plus the 7-day move in each.
3. `analyze_trend` twice over the 42 days ending on the target date, `metric: hrv` and
   `metric: sleep_secs`. Report slope direction and the current-versus-baseline delta.

If the target date's wellness row is missing or partly empty, name the absent fields and
the latest date that does carry them. Never carry a neighbouring day's HRV or sleep
forward as if it were the target date's, and never infer a value from the trend line.

Subjective scales as icuvisor returns them: sleep quality 1-4, feel 1-5.

Done when: the target date's wellness row, the fitness numbers, and both trends are in
hand, with any absent field named explicitly.

## Findings report

Return the findings as your final message. Do not write them to a file. Do not address the
athlete; you are reporting to another agent.

Dense over readable. Tables and short labelled lines, no narrative paragraphs, no advice,
no next action. Include every number the ladder produced, each with its source tool in
parentheses. Flag anything the synthesizing agent must not overstate: missing data,
`insufficient_sample`, `auto_lap_suspected`, `device_laps` intervals, profile warnings,
sports with fewer than 3 priors.

Structure:

```
DATE: Sunday 2026-07-26 (resolve_calendar_dates, tz America/New_York)
ROLLUP: 2 sessions | load 85 | 2h14m | Bike then Run, 4 min gap | brick
COMBINED ZONES: Z1 22% Z2 41% Z3 19% Z4 18% (compute_zone_time)

SESSION 1 | VirtualRide "4x4 115%"
  INTENT: <from description, get_activity_details include_full>
  PLAN VS ACTUAL: <compliance_percent, per-rep target vs actual>
  METRICS: <load, IF, NP, pw_hr, decoupling, VI, polarization, zone time>
  REPS: <table>
  N-BACK: <table, 3 priors, surface and character labelled>
  BASELINE: <z-scores, exact sport population named, trend, efforts delta>
  CAVEATS: <...>

SESSION 2 | Run "Brick Run"
  ...
  BRICK: opening mile split and opening HR vs standalone Run baseline

OTHER
  WeightTraining "Upper body" | 48m | load 22 | HR avg 112 max 148 | HR zones Z1 61% Z2 33%
    N-BACK load: 22, 19, 24 | no power, pace, or kg_lifted in the file
  Walk "Evening walk" | 31m | load 8 | no heart rate, duration and load only

WELLNESS (get_wellness_data, 2026-07-26)
  HRV 68 (7d mean 64) | RHR 48 (7d mean 50) | sleep 7h12m (7d mean 6h48m)
  sleep quality 3/4 | sleep score 81 | feel 4/5 | fatigue 2 | soreness 1 | weight 154.2 lb
  TRENDS (analyze_trend, 42d): hrv slope +0.4/day, current vs baseline +5
    sleep_secs slope flat, current vs baseline -12 min

FITNESS (get_fitness)
  CTL 62 (+1.4 over 7d) | ATL 71 (+6.0) | TSB -9 | ramp 3.2

FLAGS: <anything missing, stale, truncated, or unreliable>
```

## Verified facts about these tools

Each was checked against this athlete's data. Trust them over assumptions.

- `get_activity_details` terse omits `description` entirely, populated or not. Only
  `include_full: true` returns it.
- `get_activity_splits` on a pool swim is meaningless. It defaults to `split_unit: mi`
  and returns a single mile-long split. Swim rep splits come from `get_activity_intervals`
  with `include_full: true`.
- `get_pace_curves` reports swim pace as `pace_seconds_per_mile`. Convert to per 100
  yards for swim output: `s_per_100y = pace_seconds_per_mile * 0.0568`. The pool is 25 yd
  and the swim library is in yards.
- `compute_baseline` filters `sport` by exact string. `get_pace_curves` aggregates the
  family, so `sport: "Run"` there includes `VirtualRun`. The two disagree by design.
- `compute_baseline` defaults to `min_samples: 7`. A 42-day run baseline returns
  `insufficient_sample` for this athlete; 90 days clears it.
- `get_power_curves` returns metric `_meta.units` while every other tool returns imperial.
  Watts are unit-neutral so this is harmless, but do not carry those units into prose.
- Activity `tags` are empty across the board. Session character comes from `description`
  first, name second.
- Planned structure lives on the calendar event, not the activity, and only for the bike.
  Swim and strength calendar entries are bare `NOTE`s with empty descriptions.
