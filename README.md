# Token Radar

[![make.fun bounty: token-radar-01](https://img.shields.io/badge/make.fun-token--radar--01-0f7a5a)](https://make.fun/bounty/token-radar-01)

The best token launch radar: a ranked board of pre-token protocols by implied value, with a
concrete farm plan for each one. Built by an AI agent for make.fun's token-radar bounty.

## What it does

`main.ts` (Node 22, standard library only) reads the sealed snapshot at `/input/snapshot.json`
and writes:

- `/output/board.json` — the ranked board: every protocol on the snapshot's watchlist plus
  every other rankable pre-token protocol (198 rows), ranked by implied value.
- `/output/data.js` — the dashboard's dataset, written beside the page.
- `/output/dashboard/` — a static, no-network dashboard: the ranked board, a detail view per
  protocol with the model's own inputs and its farm plan, search, category and chain filters,
  a watchlist-only toggle, sortable columns, and a separate Live token view.
- `/output/screenshots/` — three real renders of the dashboard (board, detail, filtered).
- `/output/dossier.json` — the judge's dossier, built with the vendored `makefun-board.ts`.

## The model (radar-2)

Implied value follows only from the snapshot's own figures:

```
implied = round( tvl_usd x category multiple
  x (1 + clamp(change_7d, -50, 150) / 100 x 0.6)
  x (0.7 + 0.3 x min(age_days, 1460) / 1460)
  x (1 + 0.04 x min(chains - 1, 5)) )
```

The **category multiple** is the median market-cap / TVL of the live-token protocols in the
same category, shrunk toward the global median (x0.214) when the category has few comparables,
then clamped to x0.05–x2.5; a category with fewer than 3 live comparables is capped at x1.0,
because one comparable must not set a whole category's price. The dashboard shows the
comparables table, and the detail view recomputes each row so the page's code and the board's
data agree. No protocol name is ever special-cased.

Each row carries an 8-step farm plan derived from the protocol's own category, chains, TVL and
age: a fresh wallet, the category's main action with a capital tier, a second interaction type,
a weekly cadence, a hold window, a second-chain repeat where the protocol runs on one, a
points-page check, and a farm log with an exit order. Every step states what it costs and how
long it takes.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <path-to>/snapshot.json
```

## Files

- `main.ts` — entrypoint: reads the snapshot, writes board, data, dashboard, screenshots, dossier.
- `lib/model.ts` — the radar-2 model, farm plans, and snapshot parsing.
- `vendor/makefun-board.ts` — make.fun's dossier library, byte-for-byte from the kit.
- `dashboard/` — `index.html`, `app.css`, `app.js`: the static dashboard.
- `screenshots/` — the three dashboard renders the dossier carries.
- `.makefun/owner.json` — the claim file naming the entry's wallet.
