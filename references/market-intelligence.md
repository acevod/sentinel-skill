# Market Intelligence

This module runs when the user asks about the market broadly — not their own holdings, and not a specific trade they're about to make. See the trigger list in `SKILL.md`. If the question is about one specific asset the user is about to trade, that's Trade Guardian's Step 3 (Market check), not this module — don't duplicate that narrower check here.

## Step 1 — Scope the question

Market Intelligence questions come in two shapes; pick the one that matches:

- **Broad/general** ("how's the market today", "worth trading today") → pull a market-wide snapshot (Step 2).
- **Narrowed but not the user's own position** ("what's BTC funding looking like", "any big movers today") → pull the relevant slice only, don't force a full market-wide report for a narrow question.

## Step 2 — Pull market-wide data

Via the connected MCP, gather (adjust scope per Step 1):
- Top movers by 24h % change (gainers and losers) among major pairs
- Overall funding rate landscape across major perpetual futures pairs — not just one symbol, a sense of whether funding is broadly elevated (a market-wide "everyone's long/short" signal) or normal
- Notable volume spikes
- BTC and ETH as market bellwethers, even if the user didn't ask about them specifically — their move often frames "the market" as a whole in crypto

## Step 3 — Synthesize, don't just list

Raw data dumped as a list isn't intelligence — the value of this module is turning numbers into a read on conditions:
- Is risk-on or risk-off behavior showing up (broad gains + normal funding vs. broad declines + elevated short-side funding)?
- Is funding rate skew signaling crowded positioning (e.g. broadly high positive funding = market heavily long, which historically raises squeeze risk)?
- Anything genuinely unusual today (an outlier mover, a funding rate at an extreme band per `references/trade-guardian-thresholds.md` section 3) worth flagging even if the user didn't ask about that specific asset?

## Step 4 — Output

```
MARKET SNAPSHOT — [date/time]

Mood: [Risk-on / Risk-off / Mixed / Choppy — one line with the reasoning, not just the label]

MOVERS:
- [Top gainer]: [+X% 24h]
- [Top loser]: [-Y% 24h]

FUNDING LANDSCAPE:
- [Broadly normal / broadly elevated long-side / broadly elevated short-side], [note any pair at an extreme band]

BELLWETHERS:
- BTC: [24h %], ETH: [24h %]

[Optional] WORTH NOTING: [any one genuinely unusual data point, only if there is one]
```

For a narrow question (Step 1's second shape), skip the full template — answer just the slice asked about, in 2-3 lines, same tone but no need to force every section.

## Explicitly not this module's job

- Don't reference the user's own positions or portfolio here — that's Portfolio Analysis. If the user's question blends both ("how's the market, and does that affect my ETH position"), answer the market half here, then hand off to Portfolio Analysis for their specific exposure.
- Don't recommend a specific trade or tell the user whether to trade — describe conditions, let the user (or a subsequent Trade Guardian flow, if they act on it) draw the conclusion.
- Don't run this as a mandatory pre-check before every Trade Guardian flow — it only runs when the user actually asks a market-level question, or when Trade Guardian's own Step 3 needs broader context it doesn't already have.
