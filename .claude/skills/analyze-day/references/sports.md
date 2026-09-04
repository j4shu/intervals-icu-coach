# Sport ladders

One block per sport family. Each block lists the tools that run on top of the shared
floor in `analysis.md` Step 4, the units to report in, the metrics that survive a session
character difference, and how to read them.

Sport strings map to families as below. This table owns the mapping; the family blocks
repeat their own strings for convenience only.

| Sport string  | Family |
| ------------- | ------ |
| `Ride`        | Bike   |
| `VirtualRide` | Bike   |
| `Run`         | Run    |
| `VirtualRun`  | Run    |
| `Swim`        | Swim   |

Any sport string not listed is Other; the Other block below carries it.

Families are for display tables only. Statistical calls take the exact sport string.

---

## Bike: Ride, VirtualRide

`compute_zone_time` `zone_metric: power`.

| Tool                             | Arguments                                                                              |
| -------------------------------- | -------------------------------------------------------------------------------------- |
| `get_activity_histogram`         | `metric: power_watts`                                                                  |
| `get_activity_splits`            | default unit                                                                           |
| `get_power_curves`               | `sport` = exact sport string, 90-day window                                            |
| `get_events`                     | the target date, `include_full: true`                                                  |
| `compute_activity_segment_stats` | `stat: np`, then `stat: if` with `ftp_watts` from the profile, then `stat: decoupling` |
| `compute_zone_energy`            | the target date, mechanical kJ by power zone                                           |

### Plan versus actual, bike only

This is the one sport with authored structure on the calendar. Run it every time a bike
activity is present.

1. From `get_activity_details` `include_full`, read `paired_event_id`. Otherwise match on
   the event whose `paired_activity_id` is this activity.
2. The event's `workout_doc` gives planned steps, reps, `%ftp` targets, planned
   `normalized_power`, and planned `zoneTimes`. `workout_doc_summary.target_previews`
   gives targets already resolved to watts with the FTP basis stated.
3. `get_extended_metrics` returns `compliance_percent`, computed upstream. Report it and
   the per-rep detail, since the single number hides which rep faded.
4. Compare each rep from `get_activity_intervals` against its target. Report the fade
   across reps, not just the average.

The event `type` can read `Ride` while the activity reads `VirtualRide`. Match on family.

When no paired event exists, say the session was unplanned and skip to progression.

### Reading it

- `pw_hr` is power per heart rate. Rising across sessions at similar intensity is
  aerobic improvement. This is the primary character-robust bike signal.
- `aerobic_decoupling_percent` under about 5 is good aerobic durability. Higher means
  output fell away relative to heart rate through the session.
- `variability_index` near 1.0 is a steady ride. Above about 1.15 is a punchy or
  interval ride, expected on VO2 work.
- `polarization_index` is high for polarized sessions and low for sustained threshold.
  Read it against the session's intent, not as good or bad.
- `joules_above_ftp_kj` and per-interval `w_prime_balance` show how deep the hard reps
  went and whether recovery between them was real.

Report line tail (see `references/report.md`): NP, IF, average heart rate, surface
(indoor or outdoor).

Baseline metrics: `training_load`, `if`, `pw_hr`, `aerobic_decoupling_percent`.
Efforts delta: `effort_family: power`, `duration_seconds: [60, 300, 1200, 3600]`.

---

## Run: Run, VirtualRun

`compute_zone_time` `zone_metric: pace`.

| Tool                             | Arguments                                       |
| -------------------------------- | ----------------------------------------------- |
| `get_activity_histogram`         | `metric: pace_seconds_per_km`, report as min/mi |
| `get_activity_splits`            | default unit, mi                                |
| `get_pace_curves`                | `sport` = exact sport string, 90-day window     |
| `compute_activity_segment_stats` | `stat: drift`, then `stat: decoupling`          |

Report pace as min/mi, the athlete's preferred unit.

### Reading it

- Pace at a given heart rate is the primary character-robust run signal. A faster pace
  at equal heart rate is progression; a faster pace at higher heart rate is not.
- `drift` and `decoupling` over the session show durability. Compare like for like,
  since a hills session drifts for reasons a flat session does not.
- Treadmill pace is belt-reported, not GPS. Outdoor runs carry elevation, so check
  `elevation_gain_m` before comparing an outdoor run to a treadmill run on pace alone.
- `stride_length_m` at a given pace is a mechanics signal worth tracking.

Report line tail (see `references/report.md`): distance, average pace in min/mi, average
heart rate, surface (outdoor or treadmill).

Baseline metrics: `pace_seconds_per_mile`, `training_load`, `average_heart_rate_bpm`.
Efforts delta: `effort_family: pace`, `distance_meters: [400, 1000, 1609, 5000]`.

---

## Swim

`compute_zone_time` `zone_metric: pace`.

| Tool                     | Arguments                                                   |
| ------------------------ | ----------------------------------------------------------- |
| `get_activity_histogram` | `metric: pace_seconds_per_km`, convert for report           |
| `get_pace_curves`        | `sport: Swim`, `distance_meters: [50, 100, 200, 400, 1500]` |
| `get_activity_intervals` | `include_full: true`, for per-rep pace and heart rate       |

Report all swim pace as **per 100 yards**. The pool is 25 yd and the swim library is in
yards. `get_pace_curves` returns `pace_seconds_per_mile`; convert with
`s_per_100y = pace_seconds_per_mile * 0.0568`. Sanity check: 400 m in 395 s is 1:30/100y.

Skip `get_activity_splits`. On a pool swim it returns one mile-long split and tells you
nothing.

Terse `get_activity_intervals` returns `distance_m` and sample indices but no per-rep
pace or heart rate, which is why swim needs `include_full: true`. Rep duration in seconds
is `end_index` minus `start_index` when `icu_median_time_delta` is 1.

`interval_summary` on the activity gives the set in the athlete's own notation, for
example `["3x 800y 1:31"]`, already per 100 yards.

### Reading it

- Pace at a given heart rate is the primary character-robust swim signal.
- `stride_length_m` is stroke length. Holding pace at a longer stroke, or holding stroke
  length deeper into a set, is technique progression. Compare it rep to rep within the
  session as well as across sessions.
- Rep-to-rep consistency across a main set is often the intent. The description usually
  states the target, for example later reps within 10 s of the first.
- `pace_zone_times` and `threshold_pace` come from the athlete's swim settings.
- There is no power, so `pw_hr` reads 0 and decoupling is absent. Say so once rather
  than reporting zeros.

Report line tail (see `references/report.md`): distance in yards, average heart rate, max
heart rate, surface (pool or open water).

Baseline metrics: `training_load`, `average_speed_mph`. Activity rows carry no swim pace
field, so the pace signal comes from the curves and the rep splits.
Efforts delta: `effort_family: pace`, `distance_meters: [50, 100, 200, 400, 1500]`.

---

## Other

The catch-all: every sport string the family map does not list. `WeightTraining`, `Yoga`,
`Walk`, `Hike`, `Rowing`, `Elliptical`, `Workout`, and anything else the athlete logs. Do
not invent a per-sport ladder for a new sport string; run this block for it.

`compute_zone_time` `zone_metric: heart_rate`.

| Tool                     | Arguments                                                       |
| ------------------------ | --------------------------------------------------------------- |
| `get_activity_histogram` | `metric: heart_rate_bpm`, only when the activity has heart rate |

Keep every one of these short. Report duration, load, average and max heart rate, heart
rate zone distribution, and the N-back comparison. N-back for an Other session compares
load and duration against its own exact sport string, nothing more. Then state plainly
what the file does not carry, for example no power, no pace, no `kg_lifted`, so there is
nothing further to analyze. One or two lines each in the findings; these sessions do not
get a rep table or a plan-versus-actual pass.

When the activity has no heart rate at all, skip `compute_zone_time` and the histogram,
say so, and report duration and load only.

Include the load in the day roll-up. Two to three of these a week is real chronic load
even though each session reads thin.

Grouping in the findings: report all Other-family sessions under a single `OTHER`
section, one block per session, rather than giving each its own top-level section.

Report line tail (see `references/report.md`): average heart rate when the file carries
one, nothing otherwise. No surface; the distinction does not mean anything for these
sports.

Baseline metrics: `training_load`, `moving_time_seconds`, `average_heart_rate_bpm`.
No efforts delta; these sports have no effort curve.
