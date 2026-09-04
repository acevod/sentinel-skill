# Performance Analytics

This module runs when the user asks about their trading results or patterns *over time* — not their current state (that's Portfolio Analysis) and not a single trade (that's Trade Guardian). See the trigger list in `SKILL.md`.

## Data sources — read in this order

1. **Primary — exchange trade/order history via MCP.** Query **all four** of these every time this module runs — spot, margin, futures, and convert — regardless of what you assume the user trades. Don't decide in advance which to check based on what Trade Guardian has executed so far in this conversation, based on the account's current balance, or based on any other assumption about the user's habits — that's circular: the only way to know whether the user has activity in a given category is to actually query the historical trade/order/income endpoint for it, not to infer it from current state and skip the query. A current balance of zero or no open positions describes the account *right now* — it says nothing about what happened before now. An empty result **from the history endpoint itself** means no activity there; that's a valid outcome of checking, not a reason to have skipped checking — but it must come from actually querying history, not from noticing the current balance is zero.
   - Spot trade history
   - Margin trade history
   - Futures trade/order and income (realized P&L) history — futures P&L in particular isn't always derivable from the trade list alone, since funding payments and mark-price settlement affect realized P&L; pull income history specifically for this when available. **A zero current futures balance and zero open positions are not evidence of zero futures history** — a position that was opened and fully closed can leave the account back at zero while still having realized trades/income in the historical record. Checking current balance/positions is a check of current state, not a substitute for querying the historical trade/order/income endpoints — always query the history endpoints directly rather than inferring "no activity" from an empty current balance.
   - Convert trade history — Trade Guardian treats "convert" as a valid trade action (`references/trade-guardian.md` Step 2), and Binance records conversions through a separate endpoint from the regular spot trade list, so they won't show up there. Skipping this check would silently drop every converted trade from the performance numbers.
   
   Trade Guardian itself handles spot, margin, futures, and converts, so a user's history can span all four even if only one type has shown up in this conversation so far. Pulling from only one source and presenting it as "your performance" would silently misrepresent anyone with activity elsewhere. This is the authoritative record of what actually happened, and it's complete regardless of whether every trade went through Sentinel — all hard numbers below are computed from this source.
2. **Secondary — the journal** (`/areas/trading-journal.md`, written by `references/trading-journal.md`). Read-only here — this module never writes to it. Used to explain *why* a pattern exists, not to compute the pattern itself.

If a symbol/account type comes back empty (e.g. the user has never traded futures), that's fine — just don't report on results you didn't actually check as if their absence means zero activity there.

If a trade appears in exchange history but has no matching journal entry (executed outside Sentinel, or before the user started using it), still include it in the hard numbers — just without the reasoning enrichment for that specific trade.

## Step 1 — Scope the question

- Determine the time range: explicit ("this month", "last 30 days") or, if unstated, default to all available history and say so in the output.
- Determine the focus: overall performance, a specific asset, or pattern-seeking ("why do I keep losing").

## Step 2 — Compute the hard numbers (from exchange data)

- **Win rate**: winning trades ÷ total closed trades, for the scoped range
- **Realized P&L**: total, and broken down by asset if the user asked about a specific one
- **Average R:R**: average realized gain on winners vs. average realized loss on losers
- **Largest win / largest loss**: as concrete reference points, not just averages

## Step 3 — Pattern detection (exchange data + journal together)

This is where the module earns its place over a plain P&L summary. Look for correlations between trade characteristics and outcomes:

- Leverage level vs. outcome (e.g. "losses cluster above Nx leverage")
- Guardian tier vs. outcome (e.g. "Full Guardian trades that were confirmed despite a Devil's Advocate warning have a lower win rate than trades with no warnings")
- Time-based patterns if the data supports it (e.g. clustering by day of week or session, only if there's enough sample size to say so honestly)
- Asset-specific patterns (e.g. one asset dragging down otherwise-solid performance)

Only state a pattern if the sample size actually supports it — with a handful of trades, say so explicitly ("not enough trades yet to call this a pattern") rather than overstating a coincidence as a trend.

When a pattern involves a Guardian warning that was overridden, pull the specific detail from the matching journal entry (the `Warnings raised` line) rather than a generic "risk was flagged" — the concrete warning is what makes the insight actionable.

## Step 4 — Output

```
PERFORMANCE — [scoped range]

Win rate: [X]% ([W] wins / [L] losses, [N] trades)
Realized P&L: [+/-$Y]
Avg win: [+$A] | Avg loss: [-$B]
Largest win: [+$C on symbol] | Largest loss: [-$D on symbol]

PATTERNS:
- [Pattern #1, grounded in an actual number, with journal context if relevant]
- [Pattern #2 if there is one — don't force a second pattern if there isn't a real one]

[If sample size is too small for reliable patterns]: Only [N] closed trades in this range — not enough yet to call out reliable patterns.
```

## Explicitly not this module's job

- Never recommend a specific future trade based on the patterns found — describe what happened, let the user draw the implication (or bring it into a live Trade Guardian conversation themselves).
- Never write to the journal — this module is read-only against it, same as it's read-only against exchange history (no order-placing tools called here at all).
- Don't run automatically after every trade — unlike the Journal, this only runs when the user actually asks a performance question.
