# Report template

The shape of `days/<date>.md`. Fill this skeleton; do not improvise a different flow.

Nothing here names a sport. Per-sport fields come from `references/sports.md`.

## Skeleton

```
# <YYYY-MM-DD> (<Weekday>)

## Verdict

<How the day went. The closing sentence carries total load and total moving time.>

## Wellness

| | Today | 7-day mean |
| --- | --- | --- |
| HRV | <n> | <n> |
| RHR | <n> | <n> |
| Sleep | <h>h<mm>m | <h>h<mm>m |

- **<label>:** <form as TSB with CTL and ATL, and what the day's load meant against that state.>
  - <supporting number.>
- **<label>:** <sleep quality (1-4), sleep score, and whichever subjectives were logged, or the plain statement that none were.>

## <Sport>: "<session name>"

<duration> | load <n> | <sport-supplied tail from references/sports.md>

<Intent from the description, and plan versus actual where the sport ladder produced one.>

| <rep> | <...> |
| --- | --- |

- **<label>:** <what the numbers mean, or how the session sat against its baseline.>
  - <supporting number.>
- **<label>:** <next claim.>

**Caveats**

- <whatever the findings flagged.>

## Other Workouts

- **<Sport> "<name>":** <duration>, load <n>.
  - <what the file carries and what it does not.>

## Next

<One specific next action.>
```

## Rules

- Emit all six sections in this order every time, whatever the day held. A section with
  nothing in it carries the single word `None` and nothing more, not a bullet.
- One `## <Sport>: "<name>"` section per session, in chronological order. The session name
  disambiguates two sessions of the same sport. When an activity has no name, use the
  sport plus its start time.
- On a rest day there is no session heading to hang anything on, so emit one `## Sessions`
  heading carrying `None`.
- The fact line's core is `<duration> | load <n>`; everything after it is the tail the
  sport's block declares. It stays a fact line, not a bullet.
- Verdict, the session's intent and plan-versus-actual, and Next are prose. Everything
  else listed in the skeleton as a bullet is a bullet.
- A bullet is `- **<label>:** <claim>`. The label is whatever names that bullet's point;
  there is no fixed vocabulary. The claim is full sentences, since the hedges matter.
- Bullets are flat by default. Nest a child only when it is evidence for its parent's
  claim. Two levels, never three. A child may be a terse fragment; the parent carries the
  sentence.
- Drop the rep table when the session had no rep structure, and the `**Caveats**` block
  when there is nothing to flag. Reach for a table only where there is a real comparison
  across three or more rows and a spread worth seeing; reps that landed on top of each
  other are one bullet, not a table.
- Caps: verdict 3 sentences; wellness the table plus 2 parent bullets, 5 bullets total;
  each session 1 table plus 5 interpretation bullets; `**Caveats**` 4 bullets; Other
  Workouts 1 parent plus 2 children per session; Next 2 sentences.
- The file holds the report and nothing else: no preamble, no sign-off, no mention of
  having written it. No banners, no emojis, no em dashes.
