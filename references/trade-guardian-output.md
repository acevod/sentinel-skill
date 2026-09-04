# Trade Guardian — Output Templates

## Full Guardian (position size/leverage/reduction in "Full" tier)

```
TRADE GUARDIAN

Requested: [action, e.g. "Buy $500 BTC" / "Open 20x ETH long"]

MARKET CHECK:
- Current price: $[price]
- [Symbol] momentum: [24h % move, direction]
- Funding: [normal/elevated/extreme, with number if elevated+]
- Volume: [normal/elevated, brief note]

PORTFOLIO CHECK:
- Current [asset] allocation: [X]% of equity
- Estimated allocation after trade: ~[Y]%
- [Concentration flag if triggered, e.g. "Crypto-correlated group now ~Z%"]

DEVIL'S ADVOCATE:
- [Concrete reason #1, tied to an actual threshold crossed]
- [Concrete reason #2]
- [Concrete reason #3 if applicable]

BULL CASE:
- [Concrete reason #1]
- [Concrete reason #2]

GUARDIAN VERDICT: [Low risk / Moderate risk / High-risk entry — one line]

Do you still want me to execute [the trade]?
```

## Light Guardian (one threshold crossed into "light" tier only)

Condensed version — skip full Devil's Advocate/Bull Case lists, just state the one or two factors that triggered escalation and ask for confirmation. Still state the current price somewhere in the confirmation line, even condensed — the user is confirming a purchase and shouldn't have to ask what price they're buying at:

```
Quick check before I execute [action] at ~$[price]:
- [The one or two factors that triggered Light tier, e.g. "this would put BTC at ~12% of equity"]

Verdict: [short risk note]. Go ahead?
```

## Simple execution (no threshold crossed)

No Guardian formatting at all — just execute and report the result plainly, same as a normal MCP tool call. Don't manufacture risk commentary for a trade that didn't cross any threshold.

## Execution receipt (after confirmed execution, any tier)

```
✅ Executed: [action, size, symbol]
Fill price: [price]
Resulting position/allocation: [brief state after]
```

If execution fails (insufficient balance, invalid params, API error), report the exact error instead of the receipt — don't retry silently.

## Style notes
- Keep each section to bullet points, not paragraphs — this is meant to be scannable, not a report.
- Never pad Devil's Advocate/Bull Case with generic filler ("markets are always risky") — every bullet must trace back to an actual number from Market Check or Portfolio Check.
- Don't repeat the same warning twice in one confirmation cycle.
