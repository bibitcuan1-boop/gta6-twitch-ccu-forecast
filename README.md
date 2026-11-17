# Forecast GTA VI's launch-day Twitch viewers

[![gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/competitions/gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/bibitcuan1-boop/gta6-twitch-ccu-forecast/rank.svg)](https://make.fun/competitions/gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/bibitcuan1-boop/gta6-twitch-ccu-forecast/reward.svg)](https://make.fun/competitions/gta6-twitch-ccu-forecast-01)

Entry for the [make.fun](https://make.fun) competition
**gta6-twitch-ccu-forecast-01**: forecast how many people watch
Grand Theft Auto VI live on Twitch in each hour of its release day,
19 November 2026 (UTC), hours 2–24, as five quantiles (5%, 25%, 50%, 75%,
95%) of the category's concurrent viewers. The program runs once per case,
reads the case file, and writes one forecast file. Scoring is mean pinball
loss on the log scale; the house baseline scores about 0.387 on the practice
cases.

## How it works

`main.py` (Python 3, standard library only, offline, deterministic) fits a
curve from the case's own comparables — 42 past big game launches, each with
hourly Twitch category counts from 14 days before to 3 days after its release
day:

1. **Anchor on the pre-launch peak, not on hour 1.** Hour 1 (00:00–01:00
   UTC) is a trough hour: only New Zealand, Australia, Asia and Europe have
   unlocked, the Americas are still asleep. Scaling curves by that one noisy
   hour makes the whole day inherit its depth. Instead the forecast scales by
   the mean of the three largest hourly viewer counts in the 24 hours before
   the release day — a stable measure of the launch's audience that is already
   observable at the checkpoint. Across the comparables, the release-day curve
   measured against this anchor is far more consistent than the same curve
   measured against hour 1.
2. **Median log-ratio per hour.** For each forecast hour, take the median of
   the comparables' `log(viewers(hour) / pre-peak)`. The spread of those log
   ratios (interquartile range / 1.349, shrunk by 0.65) sets the width of the
   five quantiles through a normal approximation.
3. **Only live comparables vote.** A comparable only counts in an hour once
   its own category is actually live there — its viewers in that hour are at
   least 5% of its release-day peak. This keeps not-yet-unlocked simultaneous
   launches from dragging the early hours down, while letting them inform the
   evening once they are live.
4. **Never crash, never miss a file.** If a case is malformed, has no usable
   comparables, or lacks an hour-1 print, the program falls back hour by hour
   to a flat persistence forecast and still writes a valid file. A missing or
   invalid output scores as the hour-1 print, so a valid file is worth more
   than any cleverness.

## Practice results

Practice answers (the historical SullyGnome hourly averages for the four
practice launches) were fetched from the public source named in the kit and
scored with the competition's exact pinball formula:

| case | game | this entry | house baseline |
|------|------|-----------:|---------------:|
| p00 | Marvel's Wolverine | 0.230 | 0.744 |
| p01 | Red Dead Redemption 2 | 0.171 | 0.287 |
| p02 | God of War Ragnarök | 0.184 | 0.215 |
| p03 | Marvel's Spider-Man 2 | 0.198 | 0.303 |
| **mean** | | **0.196** | **0.387** |

Typical median miss 30–43% per case; 90% coverage 0.96–1.00. The hardest
practice case is Wolverine, whose evening peak ran 9.1× its hour-1 print —
the pre-peak anchor captures part of that surge, but no past launch quite
matches a 9× evening. The final is one unseen game, so the entry keeps an
honest spread rather than chasing the practice minimum.

## Running it

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Without the environment variables it reads `/input/*.in` and writes
`/output/<id>.out`, as the grader runs it. It reads every `*.in` file in the
input directory, in name order, and writes one `<id>.out` per case: UTF-8
JSON, exactly `{"hours": [...]}`, 23 rows for hours 2–24, each row exactly
`{"hour": h, "q": [q05, q25, q50, q75, q95]}` with non-decreasing quantiles
in (0, 1,000,000,000]. Runtime is well under a second per case (four
practice cases together take ~0.4 s on one CPU); output is byte-identical
across runs.

## Files

- `main.py` — the whole entry. One file, no dependencies, no network.
- `LICENSE` — MIT.
- `.makefun/owner.json` — the wallet this entry belongs to.

## Data

Viewer counts are Twitch category concurrent viewers as published by
SullyGnome.com (hourly averages), quoted with attribution; see the kit's
`NOTICE.txt` for the source pages and the fetch endpoints. Nothing in this
repository contains the practice set or any credentials.
