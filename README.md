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

A dark-terminal page that doubles as the entry surface. Shared visual language with the rest of the app: JetBrains Mono throughout, navy surface, amber for primary actions, green / red for up / down.

- **Index strip** — synthetic S&P 500 / FTSE 100 levels + GOLD + OIL + BTC.
- **Heatmap** — 30 of the largest names in the universe; tiles flash green/red when prices change.
- **Top movers** — biggest gainers & losers since the last refresh.
- **Scrolling ticker tape** — every name in the universe rotates across the bottom.
- **+ NEW GAME** — opens the create-a-game flow. The system generates a credential pair (email + password); the player saves them and uses them to log back in.
- **▶ LOG INTO EXISTING** — opens a credential-entry form. Email + password unlock that game's state.
- **↻ REFRESH PRICES** — fetches the latest live prices from Yahoo Finance.
- **Migration banner** — appears once on first run after the credential upgrade, surfacing the auto-generated credentials for the seeded "Family" game so they can be written down.

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

## State schema

The app holds a *registry* of games at the localStorage key `fbc_registry_v1`. Each game inside the registry carries an opaque id, a system-generated email, a SHA-256-hashed password, and the same per-game state shape used in earlier versions.

```jsonc
// fbc_registry_v1
{
  "schema_version": 1,
  "games": {
    "g_abc123": {
      "id": "g_abc123",
      "email": "quietfox4421@fbc.local",
      "password_hash": "5e3f...",           // SHA-256 hex; plain password not stored
      "name": "Family",
      "created_at": "2026-...",
      "last_played": "2026-...",
      "state": {
        "schema_version": 3,
        "game":   { "quarter": "Q2 2026", "start": "2026-04-01", "end": "2026-06-30", "currency": "USD", "starting_cash": 10000 },
        "players": {
          "p1": { "id": "p1", "name": "Player 1", "short": "P1", "emoji": "🦄", "color": "#e91e63",
                  "cash": 10000, "holdings": {} }
        },
        "order":  ["p1","p2","p3","p4"],
        "trades": [],
        "created_at":   "2026-...",
        "last_modified":"2026-..."
      }
    }
  },
  "created_at": "2026-..."
}
```

### Auxiliary keys

- `fbc_active_game_v1` — id of the most recently opened game (cleared when the user returns to the dashboard).
- `fbc_pending_reveal_v1` — set once during the legacy migration so the dashboard can surface the auto-generated credentials for the "Family" game; cleared as soon as the user dismisses the banner.
- `fbc_state_v1` — the legacy single-game key. Read-only after migration. Retained as a backup snapshot for one full release; no longer written to.
- `fbc_prices_v1` / `fbc_hist_v1` — live and historical price caches (unchanged).

### Credentials

- Email format: `{adjective}{animal}{4-digit}@fbc.local`, e.g. `quietfox4421@fbc.local`.
- Password format: `{word}-{word}-{word}-{2-digit}`, e.g. `lemon-bright-otter-37`.
- Hashed with SHA-256 via `crypto.subtle.digest`. The plain password is shown to the player once at game creation and never persisted.
- There is no password recovery. A lost credential pair means the game cannot be reopened. The credential reveal modal makes that consequence explicit.

## Mobile-first

Layouts are mobile-first. Touch targets are >44 px; bucket tabs, search, and the trade sheet are all thumb-reachable. Desktop scales up cleanly.

## Tech stack (all via CDN — no install)

- **Tailwind CSS** (cdn.tailwindcss.com)
- **React 18** (unpkg)
- **Babel-standalone** — transpiles inline JSX in-browser
- **Chart.js 4** — race chart
- **JetBrains Mono** — dashboard font (Google Fonts)

## Changelog

- **v2.1** — Educational side rails + manual credential management.
  - Two side rails flank the centred game body on wide screens. They carry a rotating selection of cards drawn from a 32-card pool covering compounding examples (£100 in Amazon 1997 → £270k, similar pieces for Apple, Berkshire, S&P 500, gold, Bitcoin, Vodafone), investor history (Buffett net worth + age-11 first stock + method + record, Munger, Lynch, Templeton, Baring), concept explainers (share, market cap, dividend, P/E, diversify, the rule of 72, price ≠ value), historical manias (Tulip 1637, South Sea 1720, dot-com 2000, GFC 2008, twenty-year cycles), and market-now snapshots (largest by market cap, gold vs the pound, inflation).
  - Right rail appears at viewport widths ≥ 1100px. Left rail also appears at ≥ 1400px. Below 1100px both rails collapse and the body keeps its mobile layout.
  - Cards are colour-coded on the left edge by category (compound = green, legend = amber, concept = blue, history = red, market = white). Each fresh page load picks a new random eight cards per side from the pool.
  - Pure presentation-layer change. No game state, no registry, no fbc_state_v1 backup, no universe, no pricing logic touched.
  - Plus: a third dashboard CTA — *⚙ SET CREDENTIALS* — opens a modal that lets you set or reset a game's email and password without touching DevTools. Useful when the auto-generated credentials are lost, or when you want to pick your own.
- **v2.0** — Dashboard becomes the shell, credential access, terminal aesthetic across the whole game.
  - Single visual system from open to close. The Bloomberg-terminal language of the landing — JetBrains Mono, navy / amber palette, dense numeric rows, 2px corners, no drop-shadow chrome — carries into the game body, the trade modal, the leaderboard, the holdings table and the trade history.
  - The PLAY button is replaced by *+ NEW GAME* and *▶ LOG INTO EXISTING* on the dashboard. Equal visual weight; primary amber + secondary outline.
  - Each game is identified by a system-generated email and password. The system shows them once at creation with *Copy* and *Download* affordances; the player must save them off because there is no recovery.
  - State refactor: a registry of games at `fbc_registry_v1` replaces the single-game `fbc_state_v1`. The legacy key is migrated losslessly into a default game named "Family" with auto-generated credentials surfaced via a one-time dashboard banner; the legacy key is then left intact as a backup.
  - Persistence flows through the registry; the live state writes happen via an App-level effect on every state change.
  - Game logic, universe, pricing chain, charts, trade history and leaderboard are unchanged.
- **v1.6.1** — Bug fix. Selling 100% of a position now zeroes the line and removes it from the portfolio. The previous ALL preset floored the dollar value (e.g. $123.46 became "123" in the input) and the round-trip dollars → units conversion left a small residual stake. The sell handler now treats ALL (and any sell within a cent of the displayed holding value) as an explicit close-out, sells the exact held units, and deletes the holding. Belt-and-braces sub-cent dust sweep added on the partial path. No schema change.
- **v1.5** — Player names stripped from source; entered locally and persisted to device `localStorage` only. PWA manifest added so phones can "Add to Home Screen". Schema bumped 1→2.
- **v1.4** — Manual-only refresh + persistent prices. Weekly chart with $10,000 baseline.
- **v1.3.1** — Hardening: ErrorBoundary, loading placeholder, distinct fallbacks.
- **v1.3** — Race chart on home view (multi-line graph, weekly Friday checkpoints).
- **v1.2** — Per-ticker live indicator, simulator drift suppression for live-fresh tickers, $ P&L per holding.
- **v1.1** — Live Yahoo Finance prices via CORS proxy chain, FX-converted.
- **v1** — First release. Bloomberg-style landing dashboard, multi-player game, seeded quarterly portfolio, save / load to disk + localStorage, mobile-first responsive layout.
