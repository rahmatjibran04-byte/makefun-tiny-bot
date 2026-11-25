# Tiny Bot — Beat Buy-and-Hold on Solana

[![make.fun](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/rahmatjibran04-byte/makefun-tiny-bot/rank.svg)](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/rahmatjibran04-byte/makefun-tiny-bot/reward.svg)](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)

A tiny trend-following paper-trading bot for the [Tiny Bot, Easy $2: Beat Buy-and-Hold on Solana](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1) competition on [make.fun](https://make.fun). It reads hourly bars for FARTCOIN, JITOSOL, POPCAT, RAY, SOL, WETH and WIF and decides when to hold each asset and when to sit in USDC. No outside data, no network — one program, fully deterministic.

## Strategy

Per-symbol **EMA(5)/EMA(13) crossover**, long only:

- **Entry** — EMA(5) crosses above EMA(13) *and* the close is above the close 12 bars ago (12h momentum filter, keeps the bot out of dead-cat bounces in a falling market).
- **Exit** — EMA(13) crosses back above EMA(5), *or* the close falls below the fast EMA(5), *or* take-profit +30%, *or* stop-loss −8%.
- **Sizing** — equity split over up to 7 slots (one per symbol), capped at 25% of equity per position; sells first, then buys in universe order; max slippage 100 bps.
- **Determinism** — float64 EMAs only, no clock, no environment, no randomness: two runs produce byte-identical action logs.

See the header comment in [`main.ts`](main.ts) for the full rules.

## Run

Requires Node 22.18+ (the grader runs Node 22.23.4; the bot is erasable TypeScript run as-is):

```sh
node main.ts
```

It speaks the `agent-fun/bars-replay/v1` protocol on stdin/stdout via the vendored shim in `vendor/makefun-bars.ts` (a byte-identical copy of the kit's `sdk/makefun-bars.ts`).

## Dev-window backtest

On the kit's dev window (136 hourly bars, 29 Oct – 3 Nov 2026, a falling market), graded with the kit's own `grade-local` (driver `process`, two runs reproduced byte for byte):

| | Net PnL | Trades | Max drawdown |
| --- | --- | --- | --- |
| **This bot** | **−US$118.77** | **39** | **1.49%** |
| Buy-and-hold baseline | −US$869.34 | 7 | 10.92% |
| EMA-crossover baseline | −US$173.04 | 28 | 2.51% |

Margin over buy-and-hold: **+US$750.57** (and +US$54.27 over the house EMA baseline). Guardrails met (≥8 trades, drawdown ≤25%, 0 rejected orders). My own approximate backtest of the same strategy gave −US$68 / 39 trades / 1.3% drawdown, and across every sliding 72-bar window (the final window's length) it meets the guardrails in 65/65 windows (minimum 10 trades, worst drawdown 1.6%) and beats buy-and-hold in 45/65. The final window is unseen, so nothing is guaranteed — the edge is structural (avoiding downtrends by sitting in USDC), not fitted to one window: the same parameters win on both halves of the dev window.

## Layout

- `main.ts` — the bot (entrypoint).
- `vendor/makefun-bars.ts` — the bars-replay protocol shim from the kit.
- `LICENSE` — MIT.

## License

MIT — see [LICENSE](LICENSE).
