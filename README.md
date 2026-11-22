[![make.fun](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/makkutsiyah-ai/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/makkutsiyah-ai/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01)

# Token Radar — pre-token protocols ranked by implied value

Entry for the [make.fun "Build the best token launch radar" bounty](https://make.fun/bounty/token-radar-01).

A program (`main.ts`, Node 22, standard library only) reads a sealed snapshot of
the DeFiLlama protocol index and ranks every pre-token protocol it records by an
implied value, then builds a static dashboard a reader can use to find and farm
the next token launch.

## The model (radar-2)

One function scores every row, from the snapshot's own figures only:

```
implied = round( tvl_usd x 0.25
  x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
  x (0.6 + min(age_days, 1095) / 1095 x 0.4)
  x (1 + 0.04 x min(chains, 10)) )
```

- **TVL** anchors the value: a protocol holding more value is worth more if it
  ever issues a token.
- **Seven-day growth** is momentum, clamped so one wild week cannot decide the
  board.
- **Age** is how long the protocol has run with no token; an unlisted age uses
  365 days.
- **Chain count** is distribution: each chain up to 10 adds 4%.

No protocol name is special-cased. The board covers the whole watchlist (the
floor) plus every other rankable pre-token protocol in the snapshot — 198 rows
on the dev snapshot. Protocols whose token already trades are set aside and
shown with their market caps, never mixed into the ranking.

Each row also carries a six-step farm plan derived from the protocol's own
category, chains, TVL and age, with a cost and a time per step.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

This writes `out/board.json`, `out/data.js`, `out/dashboard/`,
`out/screenshots/` and `out/dossier.json`. Run it twice: both runs must write
the same bytes.

## Layout

- `main.ts` — entrypoint; reads the snapshot, writes the board, data and dossier.
- `lib/model.ts` — the model, the farm plans, snapshot parsing and board checks.
- `vendor/makefun-board.ts` — make.fun's dossier library, copied from the kit.
- `dashboard/` — the static page (`index.html`, `app.css`, `app.js`), no network.
- `screenshots/` — board, detail and filtered views of the dashboard.
