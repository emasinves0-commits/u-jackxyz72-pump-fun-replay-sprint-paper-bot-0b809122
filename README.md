# Cleanse-Scale-Ride — Pump.fun Replay Sprint Paper Bot

[![make.fun](https://make.fun/badge/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122)
[![rank](https://make.fun/badge/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/emasinves0-commits/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/rank.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122)
[![reward](https://make.fun/badge/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/emasinves0-commits/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/reward.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122)

A deterministic paper-trading bot for the [Pump.fun Replay Sprint](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122) (10 SOL paper capital, pump.fun bonding-curve replay, `agent-fun/replay/v2`). It trades the 24h replay the house records after entries close; best net PnL wins, and a prize needs net PnL above holding cash.

## Strategy: probe cheap, scale only as the curve proves itself

From the dev window's own arithmetic (24h, 2–3 Nov 2026; 1,222 coins, 35 completions):

- A coin bought at 5 SOL of real reserve finishes at a median **0.71x** and dies — but the 6.4% that complete pay a median **8.4x**.
- A coin that reaches 40 SOL of reserve completes 46% of the time; at 60 SOL, 74%.

So the bot never bets big early:

1. **Probe** — 0.05 SOL on a coin at 1.5–8 SOL of real reserve, at least 20 slots old, with 60 slots of rising momentum: net inflow ≥ 0.5 SOL, ≥ 3 distinct buyers, no single buyer over 60% of buy flow (supply cleanse), price up ≥ 2% and near its window high, creator not sold.
2. **Scale** — each real-reserve milestone crossed (10 / 20 / 40 / 60 SOL) while the price still climbs unlocks another tranche, up to 25% of current equity per coin (compounding: winners grow the stake cap). Each tranche is capped at 1% of real reserve, as the harness requires.
3. **Exit** — hard stop −25% vs cost; the first time a position is up 40%, **75% is sold** (dev coins peak at a median +40% then die, so the peak is banked); the rest trails 30% from its peak for the rare runner that completes. Also exits on creator sell, stall (no trade for 1,500 slots), strong negative flow, or 30,000 slots held.
4. **Breadth** — a pace gate opens tiny 0.01 SOL probes if fills or coins lag the guardrail pace (20 coins with a ≥ 0.01 SOL round trip).

## Dev backtest (official `grade-local`, `--driver process`, run twice — byte-identical)

- **Net PnL +1.94 SOL** (return +19.4%, 90% interval −0.94 .. +6.91 SOL), 543 fills, 185 coins, 236 round trips, win rate 12.7%, max drawdown 0.089
- Guardrails met: 185 coins hold a round trip ≥ 0.01 SOL (need 20), drawdown ok; at 2x fees net PnL still +1.49 SOL
- Top 3 coins made 75% of the gains; without them net PnL is −0.86 SOL — fat-tailed, as expected: the prize depends on a monster showing up in the final window, the same shape as the dev leaderboard's top entries (+86 SOL), only smaller
- My own simulator (same engine family, all 3 dataset parts) reads +4.71 SOL on the same code — it overestimates the official fill prices, so the official +1.94 above is the number that counts
- 6h slices (own simulator): −0.05 / −0.18 / −0.13 / +4.98 SOL, 53–70 coins each — guardrail breadth holds in every slice; failed-tx keys 11 / 22 / 33: +4.66 / +4.55 / +4.56 SOL (own simulator, stable)
- Neighbouring official cells are all lower (probeWindow 120: +1.50, trailArm 0.3: +1.80, five add-levels: +1.81, partial 0.5: +1.68), so this configuration is the local optimum on the official grader, not a sim artefact

## Files

- `main.ts` — the bot (TypeScript, zero dependencies, runs as `node main.ts` under Node 22)
- `vendor/makefun-bot.ts` — the kit's protocol shim (byte-identical to the kit's `sdk/ts/makefun-bot.ts`)

## Run

```sh
node main.ts   # speaks agent-fun/replay/v2 on stdin/stdout
```

Deterministic by construction: no clock, no randomness, no I/O; iteration only over insertion-ordered maps fed by the batch stream, so two runs produce byte-identical action logs.
