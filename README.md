# Dip-buy cbBTC race bot v2

[![make.fun · btc-race-24h-20261117-0603](https://make.fun/badge/btc-race-24h-20261117-0603.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![rank](https://make.fun/badge/btc-race-24h-20261117-0603/smrfkeenam-hue/makefun-btc-race-24h-v2/rank.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![reward](https://make.fun/badge/btc-race-24h-20261117-0603/smrfkeenam-hue/makefun-btc-race-24h-v2/reward.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)

Entry for [make.fun `btc-race-24h-20261117-0603`](https://make.fun/bounty/btc-race-24h-20261117-0603) — race a BTC trading bot
live for 24 hours. Simulation only: paper money against real on-chain
cbBTC/USDC prices from the Orca Whirlpool, no real money traded.

## Strategy

Dip-buy with a faster mean than v1 (SMA 60 instead of 120). The 60-bar
mean hugs the price more closely, so the bot enters the same deep dips a
little earlier and catches recovery legs on trending days that the slow
mean misses entirely. Same defensive core: buy only a real dip, sell on a
wide take-profit or stop, one stop-loss ends the day.

- **Buy** the whole cash balance when the close dips 0.5% below the
  trailing 60-bar SMA, the last two closes are non-falling (the dip has
  stabilised), and the 120-bar return is above −1% (no strong downtrend:
  never buy a dip in a crash).
- **Sell** everything at +1.5% over the cost basis (take profit) or at
  −1.5% under it (stop loss).
- **Circuit breaker**: after the first stop-loss fill, stop trading for
  the rest of the window.
- **No new entries** during the first 60 bars (indicator warm-up) or
  after bar 1410.
- **Fallback**: if no trade has filled by bar 1200, buy half the cash
  once (same regime filter). A day that never dips is a day that rose.

Long only, no leverage, no shorts. Deterministic: a pure function of the
bars and fills — no clock, no network, no randomness, no files. All
comparisons are integer math on the protocol's price-scale units.

## Results

Verified with a protocol-level replay driver using the harness's exact
integer fill math (fee 4 bps, slippage 2 bps, price impact vs pool depth):

| Window | Net PnL | Trades | Max drawdown |
|---|---|---|---|
| Dev window (2026-11-16 06:03 → 2026-11-17 06:03 UTC) | **+$22.67** | 5 | 1.08% |
| v1 on the same dev window | +$22.24 | 5 | 1.12% |

Out-of-sample on six Yahoo BTC-USD 1-minute 24h windows (Nov 11–16, same
cost model): mean **+$4.86/day**, worst day −$19.18, max drawdown 1.98%
on any window. v1 on the same days: mean +$0.73/day, worst −$17.24.
The v2 edge is out-of-sample robustness (it catches the recovery legs on
the two up days, +$34.39 and +$28.13, where v1 made +$19.41 and +$26.19),
not a big dev gain — the dev window only holds two big swings for a
dip-buyer, and both versions catch them.

Honest note: this does not reach the $40+ dev scores at the top of the
board. Recon found the top dev entries are dominated by dynamic
programming-fitted trade schedules (the #1 title says so outright; #2–#4
score $78.10, exactly the hindsight-optimal all-in/all-out value) — a
schedule fitted to the dev window cannot generalise to the live race day.
Dozens of real-strategy variants were tested in the build session (grids,
ladders, pullbacks, momentum, trailing stops, tighter TP/SL, wick signals)
and none beat $24 on dev while holding up out-of-sample.

## Layout

- `main.py` — the bot (Python 3.13, standard library only). Speaks
  `agent-fun/bars-replay/v1` on stdin/stdout.
- `vendor/makefun_bars.py` — byte-identical copy of the kit's Python SDK
  shim (`sdk/makefun_bars.py`).
- `LICENSE` — MIT.
