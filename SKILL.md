---
name: polymarket-clob-prediction-market
description: Enforces Polymarket-style CLOB prediction-market semantics (YES/NO shares, orderbook execution, sizing, settlement, and PnL) so agents don’t behave like stock traders.
---

# Polymarket CLOB Prediction Market Skill

## When to use this skill
Use this skill when an agent is:
- Building or auditing **paper trading / execution / portfolio accounting** for Polymarket-like markets.
- Confused about **SELL without inventory**, “shorting”, or treating outcomes like **stocks**.
- Implementing **position sizing** like “$1 buys how many shares?” and accidentally adding floors that exceed budget.
- Calculating PnL incorrectly (e.g., equity-style returns instead of **binary payout at resolution**).
- Simulating fills without respecting **bid/ask**, **depth**, **tick size**, and **min order size**.

---

## Instructions

### 1) Use prediction-market semantics (not equities)
- A market resolves to exactly one winning outcome.
- Each winning share pays **$1.00** at resolution; losing shares pay **$0.00**.
- YES and NO are separate outcome tokens.
- “Bearish” exposure is typically **BUY NO**, not “short YES”.

---

### 2) Define allowed actions and inventory rules
Allowed actions:
- `BUY_YES`
- `SELL_YES` (only if holding YES shares)
- `BUY_NO`
- `SELL_NO` (only if holding NO shares)
- `HOLD` / `SKIP`

Forbidden by default:
- `SELL_*` without inventory (do NOT assume stock-style shorting).

If synthetic shorting is explicitly enabled:
- Model it as **buying the opposite outcome token**, and document the equivalence.

---

### 3) Orderbook execution must be CLOB-realistic
- Execution uses **best bid/ask**, not mid by default.
- Large sizes may consume multiple levels → **slippage** must be modeled.
- Mid-price fills allowed only in explicitly named modes (e.g., `--no-fees`, `idealized_fill`).
- Respect:
  - `tick_size`
  - `min_order_size`
  - `size_step`
  - price/size rounding

---

### 4) Sizing: “how many shares with $X”
Given:
- `budget_usdc`
- `price`
- `size_step`
- `min_order_size`
- optional `min_notional`

Algorithm:
```
raw_shares = budget_usdc / price
shares = floor(raw_shares / size_step) * size_step
notional = shares * price
```

Reject (SKIP) if:
- `shares < min_order_size`
- `notional < min_notional`
- `notional + fees > cash_available`

Rules:
- Never add artificial “minimum shares floor” that exceeds budget.
- Budget is a hard cap unless explicitly overridden.

---

### 5) Portfolio accounting (wallet + positions + basis)

Maintain:
- `cash_usdc`
- per market:
  - `yes_shares`
  - `no_shares`
  - `yes_cost_usdc`
  - `no_cost_usdc`
- realized PnL
- fees paid

---

### 6) Settlement (resolution)
At market resolution:

If YES wins:
```
payout = yes_shares * 1.0
cash_usdc += payout
yes_shares = 0
no_shares = 0
```

If NO wins:
```
payout = no_shares * 1.0
cash_usdc += payout
yes_shares = 0
no_shares = 0
```

---

### 7) Logging and explainability (mandatory)

Each decision must log:
```
market_id
outcome
side
price_used
size
notional
fees
cash_before
cash_after
```

Each skip must log:
```
reason_code
```

Standard reason codes:
- `NO_SNAPSHOT`
- `SPREAD_TOO_WIDE`
- `PRICE_OUT_OF_BOUNDS`
- `MIN_ORDER`
- `MIN_NOTIONAL`
- `NO_CASH`
- `NO_LIQUIDITY`
- `MAX_POSITIONS_PER_WINDOW`

---

## Examples

- `$1 budget, BUY_YES @ 0.40`
  -> `shares≈2.5` (rounded by step)
  -> `notional≈$1`

- `SELL_YES without inventory`
  -> `SKIP (NO_INVENTORY)`

- `Market resolves YES, holding 8 YES shares`
  -> `cash += $8`
  -> positions cleared

