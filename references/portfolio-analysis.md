# Portfolio Analysis

This module runs whenever the user asks about their own current holdings or account state — see the trigger list in `SKILL.md`. The rule that defines this module: **never return a bare number**. A balance or portfolio question always gets analysis, not just a figure.

## Step 1 — Pull the raw data

Via the connected Binance Agent OS MCP, pull:
- Spot balances (all non-zero assets)
- Open futures/margin positions (symbol, side, size, entry price, mark price, leverage, margin type)
- Account equity / total balance in base currency (USDT)
- Open orders (not filled yet — relevant for "pending exposure" context, not counted in current allocation)

If this data was already fetched earlier in the same turn (e.g. by Trade Guardian's Step 4), reuse it instead of re-fetching.

## Step 2 — Build the breakdown

For each asset/position, compute:
- **Allocation %**: value of this holding ÷ total equity
- **Unrealized P&L**: for open positions, in both absolute and % terms
- **Leverage** (if applicable) and whether margin type is cross or isolated

Aggregate:
- **Total unrealized P&L** across all open positions
- **Total leverage exposure**: sum of notional value of leveraged positions ÷ equity (gives a sense of overall portfolio leverage, not just per-position)

## Step 3 — Run risk flags

Reuse the funding-rate and concentration definitions already established in `references/trade-guardian-thresholds.md` sections 3 and 5 respectively — don't redefine separate numbers here, so a single "40% concentration" definition stays consistent across the whole skill.

- **Concentration**: single asset > 40% of equity, or correlated group (BTC/ETH/SOL-style "crypto beta") > 60% of equity
- **Funding rate exposure**: for any open futures position, note if its funding rate is in "elevated" or "extreme" territory (same bands as Trade Guardian) — this is a cost that's actively accruing, not just a one-time entry consideration
- **Leverage concentration**: if most of the portfolio's total leverage sits in one or two positions rather than spread out, flag it — one blown position doing outsized damage is a different risk shape than several smaller leveraged bets

Only include a flag if it's actually triggered — don't manufacture a risk section on a portfolio that's genuinely clean.

## Step 4 — Output

```
PORTFOLIO SNAPSHOT

Total equity: $[X]
Unrealized P&L: [+/-$Y] ([+/-Z]%)

HOLDINGS:
- [Asset]: [amount] (~$[value], [X]% of equity)[ — Nx leverage, [side], entry $[price], mark $[price], P&L [+/-$][%] if a position]
- ...

RISK FLAGS: (only if triggered)
- [Concentration flag with the actual %]
- [Funding rate flag with the actual number]
- [Leverage concentration flag]

[If no flags triggered]: No concentration, leverage, or funding risk flags at current levels.
```

Keep it scannable — bullet points, not paragraphs. This is meant to be read in a few seconds, not studied like a report.

## Relationship to other modules

- If the user's question implies they're considering a new trade (e.g. "how's my portfolio, should I add more ETH?"), finish the Portfolio Analysis output first, then let the conversation naturally continue into Trade Guardian if they confirm a specific trade — don't pre-empt Trade Guardian's own Devil's Advocate by duplicating it here.
- Portfolio Analysis is a read-only module — it never calls any order-placing MCP tool.
