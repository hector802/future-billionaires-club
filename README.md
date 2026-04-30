# Future Billionaires Club

A four-player paper-portfolio game playable on a phone. Each player starts a quarterly game with $10,000 in pretend money invested across a curated universe of US stocks, UK stocks, mining / energy / juniors, and a small crypto-and-commodities selection. The leaderboard ranks the four players by current portfolio value.

## Launching

- **Locally**: open `index.html` in any modern browser (Chrome / Edge / Safari). No dev server, no install, no build step.
- **On a phone**: deploy this folder to any static-file host (e.g. GitHub Pages) and open the URL on the phone. Bookmark or "Add to Home Screen" for an app-like icon.

## Folder layout

```
Future Billionaires Club/
├── index.html         — the entire app (React + Babel-standalone + Tailwind, all CDN)
├── style.css          — dashboard + game styling
├── manifest.json      — PWA manifest for "Add to Home Screen"
├── .nojekyll          — disables Jekyll on GitHub Pages
└── README.md
```

## Players & names

The app ships with four generic players (Player 1 through Player 4). On any device, tap the **✎** button next to a player to rename them. Names are saved to that device's `localStorage` only — they never leave the browser and never get committed to the source repo.

## What's in the app

### 1. Bloomberg-style landing dashboard

A dark-terminal page with:

- **Index strip** — synthetic S&P 500 / FTSE 100 levels + GOLD + OIL + BTC.
- **Heatmap** — 30 of the largest names in the universe; tiles flash green/red when prices change.
- **Top movers** — biggest gainers & losers since the last refresh.
- **Scrolling ticker tape** — every name in the universe rotates across the bottom.
- **PLAY** — jumps into the game.
- **↻ REFRESH PRICES** — fetches the latest live prices from Yahoo Finance.

### 2. The game

The home screen shows a **race chart** (per-player portfolio value over time, weekly cadence) and a **leaderboard** ranking the four players by current portfolio value. Tap a player to enter their seat.

Inside a player's view there are four tabs:

- **🏆 Home** — leaderboard + race chart + player picker (so players can hot-seat).
- **💸 Trade** — browse the universe by bucket, search by name or ticker, tap a name to see a kid-friendly description and buy.
- **📊 Portfolio** — current player's holdings with units, cost basis, current value, P&L. Tap a holding to sell some or all.
- **📜 History** — every trade timestamped.

### 3. Save & load

- **💾 Save** (header) — downloads `fbc-YYYY-MM-DD.json` to your Downloads folder.
- **📂 Load** (header) — pick a previously downloaded `.json` save to replace the current game state.
- The app also auto-saves to `localStorage` so closing and re-opening the tab doesn't lose progress.

## Universe

| Bucket | Count | Notes |
|---|---|---|
| **S&P 500 (US)** | ~50 | Most kid-recognisable names plus the US-listed names in the seeded portfolio. Architecture supports the full 500 — adding rows to the `UNIVERSE` const in `index.html` is the only step. |
| **FTSE 100 (UK)** | ~25 | Most kid-recognisable London-listed names. |
| **Resources** | ~50 | Diversified majors, gold majors, lithium, uranium, oil majors, royalty, and a handful of junior names. |
| **Crypto & Commodities** | 7 | BTC, ETH, SOL, SOLT, GOLD, SILVER, OIL. |
| **Total** | ~130 | |

## Live prices

Prices come from Yahoo Finance via a CORS proxy chain (`cors.lol`, `corsproxy.io`, `allorigins.win`, `codetabs`). FX conversions for foreign listings (`GBPUSD=X`, `AUDUSD=X`, `CADUSD=X`) are fetched alongside.

- The app fetches once on first load (when the cache is empty) and never auto-polls afterwards.
- Tap **↻ REFRESH PRICES** to pull fresh prices on demand.
- Last-known prices are persisted to `localStorage` (`fbc_prices_v1`, `fbc_hist_v1`) so the app opens instantly on subsequent visits.

For GBp-quoted London listings the app divides the raw price by 100 before FX-converting GBP→USD.

## Race chart

The home view shows each player's portfolio value at weekly checkpoints (every Friday close + today), back-calculated from the seeded trades using the day's historical prices and that day's FX rates. All four players start together at $10,000 on the quarter-start date.

## State schema (v2)

```jsonc
{
  "schema_version": 2,
  "game": {
    "quarter": "Q2 2026",
    "start": "2026-04-01",
    "end": "2026-06-30",
    "currency": "USD",
    "starting_cash": 10000
  },
  "players": {
    "p1": {
      "id": "p1",
      "name": "Player 1",
      "short": "P1",
      "emoji": "🦄",
      "color": "#e91e63",
      "cash": -71.49,
      "holdings": {
        "TSLA": { "units": 2.417582, "cost": 880.00 }
      }
    }
  },
  "order": ["p1","p2","p3","p4"],
  "trades": [
    { "id": "t_seed_0", "player_id": "p1", "ts": "2026-04-01T09:30:00Z",
      "action": "buy", "ticker": "SOL", "units": 18.553658, "price": 47.43, "value": 880.00, "note": "seeded" }
  ],
  "created_at": "2026-...",
  "last_modified": "2026-..."
}
```

## Mobile-first

Layouts are mobile-first. Touch targets are >44 px; bucket tabs, search, and the trade sheet are all thumb-reachable. Desktop scales up cleanly.

## Tech stack (all via CDN — no install)

- **Tailwind CSS** (cdn.tailwindcss.com)
- **React 18** (unpkg)
- **Babel-standalone** — transpiles inline JSX in-browser
- **Chart.js 4** — race chart
- **JetBrains Mono** — dashboard font (Google Fonts)

## Changelog

- **v1.5** — Player names stripped from source; entered locally and persisted to device `localStorage` only. PWA manifest added so phones can "Add to Home Screen". Schema bumped 1→2.
- **v1.4** — Manual-only refresh + persistent prices. Weekly chart with $10,000 baseline.
- **v1.3.1** — Hardening: ErrorBoundary, loading placeholder, distinct fallbacks.
- **v1.3** — Race chart on home view (multi-line graph, weekly Friday checkpoints).
- **v1.2** — Per-ticker live indicator, simulator drift suppression for live-fresh tickers, $ P&L per holding.
- **v1.1** — Live Yahoo Finance prices via CORS proxy chain, FX-converted.
- **v1** — First release. Bloomberg-style landing dashboard, multi-player game, seeded quarterly portfolio, save / load to disk + localStorage, mobile-first responsive layout.
