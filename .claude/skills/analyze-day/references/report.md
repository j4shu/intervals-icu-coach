# Report template

The shape of `days/<date>.md`. Fill this skeleton; do not improvise a different flow.

Nothing here names a sport. Per-sport fields come from `references/sports.md`.

## Skeleton

```
# <YYYY-MM-DD> (<Weekday>)

## Verdict

<How the day went. At most 3 sentences. The closing sentence carries total load and
total moving time.>

## Wellness

| | Today | 7-day mean |
| --- | --- | --- |
| HRV | <n> | <n> |
| RHR | <n> | <n> |
| Sleep | <h>h<mm>m | <h>h<mm>m |

<Form as TSB with CTL and ATL, and what the day's load meant against that state. Sleep
quality (1-4), sleep score, and whichever subjectives were logged, or the plain statement
that none were. At most 2 short paragraphs.>

## <Sport>: "<session name>"

<duration> | load <n> | <sport-supplied tail from references/sports.md>

<Intent from the description, and plan versus actual where the sport ladder produced one.>

| <rep> | <...> |
| --- | --- |

<Interpretation: what the numbers mean, and how the session sat against its baseline.>

Caveats: <missing data, insufficient_sample, auto_lap_suspected, device_laps intervals,
profile warnings, fewer than 3 priors.>

## Other Workouts

<Sport> "<name>", <duration>, load <n>. <What the file carries and what it does not.>

## Next

<One specific next action. At most 2 sentences.>
```

## Rules

- Emit all six sections in this order every time, whatever the day held. A section with
  nothing in it carries the single word `None` and nothing more.
- One `## <Sport>: "<name>"` section per session, in chronological order. The session name
  disambiguates two sessions of the same sport. When an activity has no name, use the
  sport plus its start time.
- On a rest day there is no session heading to hang anything on, so emit one `## Sessions`
  heading carrying `None`.
- The fact line sits directly under the session heading, before any prose. Core is
  `<duration> | load <n>`; everything after it is the tail the sport's block declares.
- Within a session, hold this order: intent and plan versus actual, then the rep table if
  the session had reps, then interpretation, then `Caveats:` last. Drop the table when the
  session had no rep structure. Drop the `Caveats:` line when there is nothing to flag.
- Caps: verdict 3 sentences; wellness the table plus 2 short paragraphs; each session
  1 table plus 2 paragraphs, with the `Caveats:` line exempt; Other Workouts 2 lines per
  session; Next 2 sentences.
- The file holds the report and nothing else: no preamble, no sign-off, no mention of
  having written it. No banners, no emojis, no em dashes.
