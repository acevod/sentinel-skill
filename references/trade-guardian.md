# Trade Guardian

This is the module that runs whenever the user wants to **execute, modify, or close** a trade — spot, margin, or futures. It turns a plain MCP order call into a risk-aware confirmation flow, following one rule above all:

**Challenge the trade, don't block the user.**

The flow is always: **Classify → Identify → Market check → Portfolio check → Devil's Advocate + Bull Case → Confirm → Execute → Verify → Receipt → (hand off to Trading Journal)**

Read `references/trade-guardian-thresholds.md` for the numeric limits that decide how deep the analysis goes, and `references/trade-guardian-output.md` for exactly how to format each response tier. This file only covers the workflow logic.

## Step 1 — Classify the request

Decide the tier: **Simple**, **Light Guardian**, or **Full Guardian**.

Never classify by absolute dollar amount alone — a small-looking number can still be a large % of a small account. Pull the current account balance/equity via the connected MCP first (a lightweight check, not the full Step 4 exposure analysis) so the position-size-vs-equity threshold can actually be computed, not guessed.

Check the request against every threshold in `references/trade-guardian-thresholds.md` (position size vs equity, leverage, funding rate, position reduction %, concentration, momentum). If **any single threshold** is crossed into "light" or "full" territory, escalate to that tier — thresholds don't average out.

- **Simple** → skip to Step 7 (Execute) directly, no Guardian analysis needed.
- **Light Guardian** → run Steps 2–5 in condensed form (see the Light Guardian template in `references/trade-guardian-output.md`).
- **Full Guardian** → run Steps 2–5 in full.

## Step 2 — Identify the exact trade parameters

Pin down: action (buy/sell/open/close/convert), asset/pair, size (in quote currency and in the asset), order type, leverage (if futures), and margin type. If anything is ambiguous, ask before proceeding — do not guess trade parameters.

## Step 3 — Check relevant market data

Pull what's relevant to the trade: current price, 24h % change, 24h volume, and — for futures — funding rate and open interest if available. This feeds both the momentum check in `references/trade-guardian-thresholds.md` and the Devil's Advocate section. If Market Intelligence has already pulled broad market context earlier in this conversation, reuse it instead of re-fetching — but still check this specific symbol's data if it wasn't already covered.

## Step 4 — Check portfolio/exposure

Pull current account/position info via the connected MCP: existing exposure to this asset, exposure to correlated assets (e.g. BTC/ETH/SOL treated as one "crypto beta" group), current leverage on the symbol, and any open orders on it. Compute what the allocation would look like *after* the trade. If Portfolio Analysis already pulled this in the same turn, reuse it.

## Step 5 — Devil's Advocate + Bull Case

For Light/Full Guardian tier, explicitly write out:
- **Devil's Advocate**: concrete reasons this trade could be wrong, grounded in the actual numbers pulled in Steps 3–4 (not generic boilerplate — only raise a factor if it actually crossed a threshold in `references/trade-guardian-thresholds.md`).
- **Bull Case**: the legitimate case for the trade, just as concrete.

Then give a one-line **Guardian Verdict** (e.g. "High-risk entry", "Reasonable entry with elevated funding", "Low risk").

## Step 6 — Confirm

Ask the user to explicitly confirm before executing. Do not proceed on silence or ambiguity. One confirmation is enough — if the user says yes, move to execution without re-litigating the same warning again.

## Step 7 — Execute

Execute via the connected Binance Agent OS MCP tools. Hard-block (do not attempt, regardless of tier) on objective problems: insufficient balance, invalid symbol/parameters, unsupported pair, or an API error — surface the exact error to the user instead of retrying blindly.

## Step 8 — Verify

After execution, query the order/position status back through the MCP to confirm it actually filled as expected (price, size, side). Don't just trust the initial response — confirm.

## Step 9 — Execution receipt

Give the user a short, clear confirmation: what executed, at what price/size, and the resulting position/allocation. See `references/trade-guardian-output.md` for format.

## Step 10 — Hand off to Trading Journal

If — and only if — Step 8 (Verify) confirmed the trade actually executed, immediately continue into `references/trading-journal.md` to log the entry before ending the turn. Present the execution receipt and the journal confirmation together in the same response.

## Design principles

- Guardian informs and challenges; it does not gatekeep. A confirmed trade gets executed unless there's an objective execution problem.
- Not every action needs full analysis — a small convert or a trade well under every threshold should feel as fast as a plain executor.
- Never repeat the same warning twice in one confirmation cycle — that's nagging, not risk management.
- If `references/trade-guardian-thresholds.md` is missing or a threshold is undefined for the request type, default to treating the trade as significant (ask + explain) rather than silently executing.
- A rejected or abandoned trade (user says no, or confirmation never comes) does **not** get a journal entry — only executed trades are logged.
