# Data collection ladder

Run this ladder for the target day and retain only what `references/report.md`
needs. When a tool returns nothing useful, say so plainly in the report and
continue; that is the finding.

Never write a file in this step; the report in `days/<date>.md` is the only file
this skill writes.

The tool names below are icuvisor names; call them exactly, as
`mcp__icuvisor__<name>`.

Every tool named here is **mandatory** for the sports present that day. Run the
whole ladder, then carry into the report only what it flushes out.

## Step 1: Resolve the day and fetch it

1. Resolve the target date and weekday in the shell, in the host's local
   timezone, which is the athlete's. If the caller named a `YYYY-MM-DD`, run
   `date -j -f "%Y-%m-%d" "<the date>" +"%F %A"`; otherwise run `date +"%F %A"`.
   Both print the date and weekday that go in the report header. Never compute a
   date by model arithmetic.
2. `get_activities` for that single date.

Done when: the day's activity list is resolved.

## Step 2: Read the sport ladders

Read `references/sports.md` in full, now, before
analyzing any activity. It holds the per-sport tool ladders, unit rules, and
interpretation thresholds that Step 4 depends on.

Done when: `references/sports.md` has been read this run.

## Step 3: Fetch comparison history

One call: `get_activities` with `page_size: 100` over the 42 days ending on the
target date. This page supplies the N-back tables for every sport. Check
`_meta.more_available`: when it is true the page stopped before the window edge,
so follow its `next_page_token` for the rest before building the tables, and say
so if you stop early.

When a sport has fewer than 3 priors in it, note the shortfall in that sport's
section and move on. Do not widen the window looking for more.

Also call `get_athlete_profile` once for thresholds, zones, sport settings, and
preferred units. Read `sport_settings` per exact sport string: a sport string
with no entry has no zones to report, and it shows up as an absent
`sport_settings` entry rather than as a warning.

Done when: history covering the 42-day window and the athlete profile are in
hand.

## Step 4: Analyze each activity

For every activity in the day's list, in chronological order, run the shared
floor and then that sport's ladder from `references/sports.md`.

Shared floor, every sport:

| Tool                                            | Why                                                                                                                                              |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `get_activity_details` **`include_full: true`** | The terse shape omits `description` even when it is populated, and that field carries the session's intent and target.                           |
| `get_activity_intervals`                        | Rep structure. Read `_meta.interval_source`, `_meta.auto_lap_suspected`, `_meta.interval_source_caveat` before making any claim about execution. |
| `get_extended_metrics`                          | Decoupling, intensity factor, `pw_hr`, polarization, variability, stride/stroke length, per-interval strain.                                     |
| `get_activity_messages`                         | The athlete's own comments, which the numbers do not carry.                                                                                      |

Per-activity zone distribution does not come from a tool. Read it from the
activity row's `icu_zone_times` (power) or `icu_hr_zone_times` (heart rate) and
cross-check it against `get_activity_histogram`. `compute_zone_time` is a
date-range weekly aggregate, never a per-activity or per-day one; see the
verified facts below.

Sports outside Swim, Bike, and Run run the Other block in
`references/sports.md`, which pares the floor down: no rep table, no
plan-versus-actual, and no zone work when the file carries no heart rate. Follow
that block over the floor for those sessions.

When `interval_source` is `device_laps`, or `auto_lap_suspected` is true, or the
activity collapses to one averaged lap, say so plainly and use
`compute_activity_segment_stats` over explicit segments for any execution claim.

Track the source tool behind each number for your own fidelity, but the report
file carries no tool citations. Report units exactly as the sport ladder
specifies.

Done when: every activity in the day's list has been through the shared floor
and its sport ladder, with no activity summarized from the day-list row alone.

## Step 5: Progression

For each sport present that day, both halves:

**N-back table.** The last 3 prior sessions of the same sport family, from the
Step 3 page. Families are Run, Bike, Swim, and Other, per the family map at the
top of `references/sports.md`, which owns the mapping. Label every row with its
surface, in the vocabulary its family's block names, and its session character.
Take the character from the activity `description` when there is one, and from
the name otherwise. Compare on the metrics that survive a character difference
rather than on raw pace or raw IF; the sport ladder names them.

**Baseline verdict.** `compute_baseline` and `analyze_trend` over the 42 days
ending on the target date, with `sport` set to the **exact** sport string, not
the family. `compute_baseline` also takes the 42 days before that as its
baseline window. These tools fetch server-side, so this costs no history in
context.

Keeping the statistics exact-sport is deliberate. Merging two populations under
one spread hides a session that sits far off its own normal, which is how a
treadmill run reads as unremarkable next to outdoor runs. State which population
each z-score came from.

`compute_baseline` needs `min_samples` 7, and a 42-day window will not always
hold seven sessions. When it returns `insufficient_sample`, or when a sport has
nothing at all inside the window, report that plainly with the `n_baseline` it
found and move on. A sport the athlete has not touched in 42 days is a normal
outcome, not a broken call, and it is not a reason to widen the window.

The two windows must not touch. `baseline_end_date` has to fall strictly earlier
than `current_start_date`, so a baseline that ends the same day the current
window starts is rejected. That leaves the two adjacent rather than separated:
end the baseline the day before the current window starts, and they share no
date. Do not leave a gap between them, because a gap shifts the 42-day baseline
off the days it is meant to cover.

The rejection is also hard to read: the user-facing text is the generic "invalid
compute_baseline arguments" and the specific reason is wrapped rather than
shown, so a failure usually means a boundary or date-format problem rather than
a bad metric name. Check the seam first, and do not conclude the tool is broken.
The usable payload nests under `result`, with `insufficient_sample` in `_meta`
alongside it.

Add `analyze_efforts_delta` for the sport's effort family, over the same two
42-day windows. Best-effort deltas are independent of session character, which
makes them the most reliable progression signal on a mixed set of sessions.

Done when: every sport present that day has both an N-back table and a baseline
verdict, or an explicit statement of what was insufficient and why.

## Step 6: Day roll-up

Total load, total time, session count, session order with the gap between
sessions, and the combined zone distribution.

Brick: a Bike-family and a Run-family activity starting within 30 minutes of
each other. When one is present, add the run's opening mile split against the
standalone Run-family baseline from Step 5, and the opening HR against the same
baseline. That comparison is the point of a brick.

Done when: the roll-up covers load, time, order and spacing, and combined zones,
and any brick has its opening-split comparison.

## Step 7: Wellness and fitness

The day is not only its sessions. Run all three of these every time, whether or
not the day has an activity.

1. `get_wellness_data` over the 7 days ending on the target date. Report the
   target date's row: HRV, resting HR, sleep duration, sleep quality, sleep
   score, weight, and whichever of feel, fatigue, soreness, stress, motivation,
   and readiness the athlete logged. Give the mean of those 7 days alongside
   HRV, resting HR, and sleep duration so the day reads against its own recent
   normal.
2. `get_fitness` over the 14 days ending on the target date. It returns weekly
   buckets anchored on Mondays, so a narrower span yields a single bucket and no
   move to read. Take the target date's CTL, ATL, TSB, and ramp from that date's
   wellness row, and report the move between the two most recent weekly anchors,
   which sit seven days apart.
3. `analyze_trend` twice over the 42 days ending on the target date,
   `metric: hrv` and `metric: sleep_secs`. Report slope direction and the
   current-versus-baseline delta.

If the target date's wellness row is missing or partly empty, name the absent
fields and the latest date that does carry them. Never carry a neighbouring
day's HRV or sleep forward as if it were the target date's, and never infer a
value from the trend line.

Done when: the target date's wellness row, the fitness numbers, and both trends
are in hand, with any absent field named explicitly.

Carry missing data, `insufficient_sample`, `auto_lap_suspected`, `device_laps`
intervals, profile warnings, and sports with fewer than 3 priors into the
report's Caveats, and name the 42-day window the comparison came from.

## Verified facts about these tools

Each was checked against this athlete's data. Trust them over assumptions.

- The argument names are not guessable; read each tool's schema before its
  first call. `get_events` wants `oldest` **and** `newest`,
  `compute_zone_energy` and `compute_zone_time` want `start_date`/`end_date`,
  and `compute_activity_segment_stats` wants a time or distance range on every
  call.
- `compute_zone_time` takes dates, not an `activity_id`, and it sits on the
  upstream weekly bucket. A single-day range returns `status: unavailable` with
  `insufficient_reason: missing_precomputed_zone_times` and `n: 0` for every
  sport. A 7-day range returns the whole week's seconds, not the day's. Use it
  for week-scale polarization context only.
- `get_activity_splits` on a pool swim is meaningless. It defaults to
  `split_unit: mi` and returns a single mile-long split. Swim rep splits come
  from `get_activity_intervals` with `include_full: true`.
- `compute_baseline` filters `sport` by exact string. `get_pace_curves`
  aggregates the family, so `sport: "Run"` there includes `VirtualRun`. The two
  disagree by design.
- `analyze_trend` takes activity-row metrics (`training_load`,
  `average_speed_mph`, `pace_seconds_per_mile`, `average_heart_rate_bpm`,
  `moving_time_seconds`) and rejects the per-activity extended ones (`pw_hr`,
  `if`, `aerobic_decoupling_percent`) with "requires per-activity extended
  metrics; use get_extended_metrics or compute_baseline". Send those to
  `compute_baseline`, which accepts them.
- `get_activity_histogram` can return `insufficient_sample: true` with
  `reason: stream_fetch_failed` on a strength file, which carries `time` and
  `heartrate` streams but no `watts` or `cadence`. Fall back to the row's
  `icu_hr_zone_times`, which is populated on those activities, and say the
  histogram was unavailable.
- `get_activity_histogram` takes `metric: pace_seconds_per_km` or
  `heart_rate_bpm`, but the pace buckets come back in **seconds per mile**, with
  `_meta.emitted_unit` set to `seconds_per_mile` and the bucket boundaries
  matching. For a run that is already the report unit; apply the Swim block's
  conversion for a swim.
- `interval_summary` and `paired_event_id` exist only under
  `get_activity_details` `include_full: true`. The terse shape omits both, so a
  ride reads as unplanned and a set reads as unknown until the full call is
  made.
- The activity `threshold_pace` field is in metres per second, not seconds. The
  swim activity's `1.075768` is the athlete's 85.0 s/100y, matching
  `threshold_pace_seconds_per_100y` in the profile.
- Neither `include_full: true` payload truncates. `get_activity_details` stays
  small, measured at about 6 KB on a swim, and nests the extra fields under
  `activity.full`; `get_activity_intervals` is the large one at about 150 KB, so
  project that down to the fields in use rather than emitting it raw.
- On any activity, `get_activity_intervals` `include_full: true` nests
  per-interval detail under `intervals[].full`, so heart rate is at
  `intervals[].full.average_heartrate` rather than at the top level of the
  interval object. A projection reading `average_heartrate` directly sees
  nothing and wrongly concludes the field is absent. The terse `groups` block
  carries a per-group average bpm and cadence and is a cheap cross-check.
- `get_pace_curves` requires `oldest` and `newest`; `sport: Swim` alone fails.
- `get_power_curves` returns metric `_meta.units` while every other tool returns
  imperial. Watts are unit-neutral so this is harmless, but do not carry those
  units into prose.
- Activity `tags` are empty across the board. Session character comes from
  `description` first, name second.
- Planned structure lives on the calendar event, not the activity, and only for
  the bike. Swim and strength calendar entries are bare `NOTE`s with empty
  descriptions.
