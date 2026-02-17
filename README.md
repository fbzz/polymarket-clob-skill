# Polymarket CLOB Skill — README

This skill defines how an agent should behave when trading or simulating trades in a Polymarket-style prediction market.

## What this is
A reusable specification that forces agents to:
- Treat markets as binary YES/NO outcomes
- Use orderbook (CLOB) execution logic
- Size positions correctly using a USD budget
- Track portfolio, positions, and cost basis
- Calculate PnL using prediction‑market settlement rules (0/1 payout)

## Why it exists
Most trading agents default to stock‑market logic:
- short selling
- equity-style returns
- share pricing assumptions

Prediction markets work differently:
- shares priced in [0,1]
- payout happens only at resolution
- YES and NO are separate tokens

This skill prevents those mistakes.

## Where to use
- Paper trading engines
- Execution simulators
- Strategy agents
- Backtests
- Portfolio/PnL accounting modules

## What it enforces
- Budget‑based sizing ($ → shares)
- Inventory-aware SELL rules
- Bid/ask execution realism
- Settlement-based PnL
- Structured logging of decisions and skips

## Files
- `polymarket-clob-prediction-market.md` → full implementation spec

