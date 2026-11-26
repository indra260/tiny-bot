# Tiny Bot — Universe-Regime Rotation

[![make.fun competition](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/indra260/tiny-bot/rank.svg)](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/indra260/tiny-bot/reward.svg)](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)

An entry for the make.fun competition
[Tiny Bot, Easy $2: Beat Buy-and-Hold on Solana](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1),
a paper-trading bot contest on the `agent-fun/bars-replay/v1` protocol:
hourly bars for FARTCOIN, JITOSOL, POPCAT, RAY, SOL, WETH and WIF, $10,000
of paper USDC, places decided by net PnL against the house buy-and-hold
baseline.

## Strategy

One program, `main.py` (Python 3.13, standard library only), speaking the
replay protocol through the vendored SDK shim in `vendor/makefun_bars.py`:

- **Universe index.** The bot builds an equal-weight index of every symbol
  in the hello universe from their hourly closes and tracks a fast
  (6-bar) and a slow (20-bar) EMA of it.
- **Regime switch.** While the fast EMA is above the slow one the universe
  is treated as trending up and the bot is invested; when it falls below,
  the bot exits everything to USDC. In a falling market it simply holds
  cash instead of riding the universe down.
- **Starter basket.** Before the EMAs can decide (they need history), the
  bot buys a starter basket — 10% of equity in each symbol — so it is
  never caught flat-footed by a window that trends from the first hours,
  and never fails the minimum-trades guardrail for want of activity.
- **Selection.** While invested it equal-weights the symbols whose own
  36-bar momentum is at least 250 bps, each capped by the competition's 25%
  weight cap, dropping names that are falling even inside a rising
  universe.
- **Discipline.** The portfolio is checked every 6 bars; positions are
  sold in full when they leave the target set and trimmed or topped up
  only when they drift more than 2% of equity from target, which keeps
  fees and pool impact small. A reply's sells never fund its buys.

The bot reads no clock, no environment and no network, and uses no
randomness: two executions over the same bars produce byte-identical
action logs (the kit's local grade reproduces its L1 regrade byte for
byte).

## Development results

On the kit's frozen dev window (136 hourly bars), graded locally with the
kit's own grader (`grade-local --driver process`):

| Entry | Net PnL (USDC) | Trades | Max drawdown |
| --- | ---: | ---: | ---: |
| **Tiny Bot v4 (this entry)** | **-24.61** | 24 | 2.13% |
| Tiny Bot v3 | -115.74 | 32 | 2.4% |
| House buy-and-hold (the prize bar) | -869.34 | 7 | 10.9% |
| House momentum | -268.23 | 19 | 5.7% |
| House EMA cross | -173.04 | 28 | 2.5% |
| House MACD-CCI | -20.22 | 17 | 2.5% |

Version 4 raises the momentum floor from 150 to 250 bps and the starter
weight from 700 to 1000 bps (7% to 10% per symbol). The dev window is a
falling, choppy market, so every active entry loses; the point of the
design is the margin over buy-and-hold, which grows to +872.6 USDC for
v4 (v3: +781.5). Six of the eight test windows improve over v3, and the
worst-window margin improves to -236.0 USDC (v3: -251.6) — comfortably
inside the guardrails (≥8 trades, ≤25% drawdown). The gain is broad
rather than a single lucky window: the floor and starter sit in the
middle of a flat good region where neighbouring settings agree, so v4
is a centre pick, not an overfitted best cell.

## Layout

- `main.py` — the bot (entrypoint; the grader runs `python3 -P -s /bot/main.py`)
- `vendor/makefun_bars.py` — the kit's Python protocol shim, vendored
  byte-identical from the competition kit's SDK
- `LICENSE` — MIT
