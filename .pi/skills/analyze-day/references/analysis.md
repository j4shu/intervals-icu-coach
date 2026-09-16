# Data collection ladder

Run this ladder for the target day and retain only what `references/report.md` needs. When
a tool returns nothing useful, say so plainly in the report and continue; that is the
finding.

Never write a file in this step; the report in `days/<date>.md` is the only file this skill
writes.

The tool names below are icuvisor names; call them exactly.

Every tool named here is **mandatory** for the sports present that day. Run the whole
ladder, then carry into the report only what it flushes out.

## Step 1: Resolve the day and fetch it

1. `resolve_calendar_dates` with `offsets: [0]` for the athlete-local date, weekday, and
   timezone. When the caller named an explicit `YYYY-MM-DD`, that date is the target and
   `resolve_calendar_dates` supplies only the timezone and weekday for it.
   Do not compute dates by model arithmetic.
2. `get_activities` for that single date.

Done when: the day's activity list is resolved.

## Step 2: Read the sport ladders

Read `.pi/skills/analyze-day/references/sports.md` in full, now, before analyzing any
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
units. Read `sport_settings` per exact sport string: a sport string with no entry has no
zones to report, and it shows up as an absent `sport_settings` entry rather than as a
warning.

Done when: one history page and the athlete profile are in hand.

## Step 4: Analyze each activity

For every activity in the day's list, in chronological order, run the shared floor and
then that sport's ladder from `references/sports.md`.

Shared floor, every sport:

| Tool | Why |
| --- | --- |
| `get_activity_details` **`include_full: true`** | The terse shape omits `description` even when it is populated, and that field carries the session's intent and target. |
| `get_activity_intervals` | Rep structure. Read `_meta.interval_source`, `_meta.auto_lap_suspected`, `_meta.interval_source_caveat` before making any claim about execution. |
| `get_extended_metrics` | Decoupling, intensity factor, `pw_hr`, polarization, variability, stride/stroke length, per-interval strain. |
| `get_activity_messages` | The athlete's own comments, which the numbers do not carry. |

Per-activity zone distribution does not come from a tool. Read it from the activity row's
`icu_zone_times` (power) or `icu_hr_zone_times` (heart rate) and cross-check it against
`get_activity_histogram`. `compute_zone_time` is a date-range weekly aggregate, never a
per-activity or per-day one; see the verified facts below.

Sports outside Swim, Bike, and Run run the Other block in `references/sports.md`, which
pares the floor down: no rep table, no plan-versus-actual, and no zone work when the file
carries no heart rate. Follow that block over the floor for those sessions.

When `interval_source` is `device_laps`, or `auto_lap_suspected` is true, or the activity
collapses to one averaged lap, say so plainly and use `compute_activity_segment_stats`
over explicit segments for any execution claim.

Track the source tool behind each number for your own fidelity, but the report file carries
no tool citations. Report units exactly as the sport ladder specifies.

Done when: every activity in the day's list has been through the shared floor and its
sport ladder, with no activity summarized from the day-list row alone.

## Step 5: Progression

For each sport present that day, both halves:

**N-back table.** The last 3 prior sessions of the same sport family, from the Step 3
page. Families are Run, Bike, Swim, and Other, per the family map at the top of
`references/sports.md`, which owns the mapping. Label every row with its
surface, in the vocabulary its family's block names, and its session character. Take the
character from the activity `description` when there is one, and from the name otherwise. Compare on the metrics that survive a character
difference rather than on raw pace or raw IF; the sport ladder names them.

**Baseline verdict.** `compute_baseline` and `analyze_trend` over the 90 days before the
target date, with `sport` set to the **exact** sport string, not the family. These tools
fetch server-side, so this costs no history in context.

Keeping the statistics exact-sport is deliberate. Over 90 days the outdoor `Run`
population runs about 619 s/mi with a standard deviation near 101, while `VirtualRun`
runs about 648 s/mi with a standard deviation near 7. Merging them produces a combined
spread that hides a treadmill session being far off its own normal. State which
population each z-score came from.

`compute_baseline` needs `min_samples` 7, which is why the window is 90 days: a 42-day run
baseline comes back `insufficient_sample` for this athlete. When it does, report that
plainly with the `n_baseline` it found and move on.

The two windows must not touch. `baseline_end_date` has to fall strictly earlier than
`current_start_date`, so a baseline that ends the same day the current window starts is
rejected. That is easy to hit by accident, because the natural phrasing "the 90 days
ending at the target" against "the 90 days before that" produces exactly that shared
boundary. Leave a day or more between the windows.

The rejection is also hard to read: the user-facing text is the generic "invalid
compute_baseline arguments" and the specific reason is wrapped rather than shown, so a
failure usually means a boundary or date-format problem rather than a bad metric name.
Check the seam first, and do not conclude the tool is broken. The usable payload nests
under `result`, with `insufficient_sample` in `_meta` alongside it.

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
   date. Report the target date's row: HRV, resting HR, sleep duration, sleep quality,
   sleep score, weight, and whichever of feel, fatigue, soreness, stress, motivation, and
   readiness the athlete logged. Give the 7-day mean alongside HRV,
   resting HR, and sleep duration so the day reads against its own recent normal.
2. `get_fitness` with `start_date` 7 days before the target date and `end_date` the target
   date. Report CTL, ATL, TSB, and ramp on the target date, plus the 7-day move in each.
3. `analyze_trend` twice over the 42 days ending on the target date, `metric: hrv` and
   `metric: sleep_secs`. Report slope direction and the current-versus-baseline delta.

If the target date's wellness row is missing or partly empty, name the absent fields and
the latest date that does carry them. Never carry a neighbouring day's HRV or sleep
forward as if it were the target date's, and never infer a value from the trend line.

Done when: the target date's wellness row, the fitness numbers, and both trends are in
hand, with any absent field named explicitly.

Carry missing data, `insufficient_sample`, `auto_lap_suspected`, `device_laps` intervals,
profile warnings, and sports with fewer than 3 priors into the report's Caveats.

## Verified facts about these tools

Each was checked against this athlete's data. Trust them over assumptions.

- The server must be connected before any tool name resolves. `tools.search` returning
  zero items, or `tools.describe({ path: "icuvisor_<tool>" })` returning `tool_not_found`,
  means the server is disconnected, not that the name is wrong: call
  `mcp({ connect: "icuvisor" })` and retry. `mcp({})` with no arguments reports the
  connection state and tool count.
- Read every tool's `inputTypeScript` before its first call, in one `mcpScript` loop over
  `tools.describe({ path: "icuvisor_<tool>" })`, once the server is connected. The name
  needs the `icuvisor_` prefix even then: a bare name returns `tool_not_found` and a bare
  `tools.call` fails, and the mcp tool's `describe` needs the prefix too. The
  argument names are not guessable: `get_events` wants `oldest` **and** `newest`,
  `compute_zone_energy` and `compute_zone_time` want `start_date`/`end_date`, and
  `compute_activity_segment_stats` wants a time or distance range on every call.
- `compute_zone_time` takes dates, not an `activity_id`, and it sits on the upstream
  weekly bucket. A single-day range returns `status: unavailable` with
  `insufficient_reason: missing_precomputed_zone_times` and `n: 0` for every sport. A
  7-day range returns the whole week's seconds, not the day's. Use it for week-scale
  polarization context only.
- `get_activity_splits` on a pool swim is meaningless. It defaults to `split_unit: mi`
  and returns a single mile-long split. Swim rep splits come from `get_activity_intervals`
  with `include_full: true`.
- `compute_baseline` filters `sport` by exact string. `get_pace_curves` aggregates the
  family, so `sport: "Run"` there includes `VirtualRun`. The two disagree by design.
- `get_activity_histogram` can return `insufficient_sample: true` with
  `reason: stream_fetch_failed` on a strength file, which carries `time` and `heartrate`
  streams but no `watts` or `cadence`. Fall back to the row's `icu_hr_zone_times`, which is
  populated on those activities, and say the histogram was unavailable.
- `get_activity_histogram` takes `metric: pace_seconds_per_km` or `heart_rate_bpm`, but the
  pace buckets come back in **seconds per mile**, with `_meta.emitted_unit` set to
  `seconds_per_mile` and the bucket boundaries matching. For a run that is already the
  report unit; apply the Swim block's conversion for a swim.
- `interval_summary` and `paired_event_id` exist only under `get_activity_details`
  `include_full: true`. The terse shape omits both, so a ride reads as unplanned and a set
  reads as unknown until the full call is made.
- The activity `threshold_pace` field is in metres per second, not seconds. The swim
  activity's `1.075768` is the athlete's 85.0 s/100y, matching
  `threshold_pace_seconds_per_100y` in the profile.
- Neither `include_full: true` payload truncates. `get_activity_details` stays small,
  measured at about 6 KB on a swim, and nests the extra fields under `activity.full`;
  `get_activity_intervals` is the large one at about 150 KB, so project that down to the
  fields in use rather than emitting it raw.
- On any activity, `get_activity_intervals` `include_full: true` nests per-interval detail
  under `intervals[].full`, so heart rate is at `intervals[].full.average_heartrate` rather
  than at the top level of the interval object. A projection reading `average_heartrate`
  directly sees nothing and wrongly concludes the field is absent. The terse `groups` block
  carries a per-group average bpm and cadence and is a cheap cross-check.
- `get_pace_curves` requires `oldest` and `newest`; `sport: Swim` alone fails.
- `get_power_curves` returns metric `_meta.units` while every other tool returns imperial.
  Watts are unit-neutral so this is harmless, but do not carry those units into prose.
- Activity `tags` are empty across the board. Session character comes from `description`
  first, name second.
- Planned structure lives on the calendar event, not the activity, and only for the bike.
  Swim and strength calendar entries are bare `NOTE`s with empty descriptions.
