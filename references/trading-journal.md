# Trading Journal

This module never triggers from user phrasing directly — it runs automatically as Step 10 of `references/trade-guardian.md`, right after a trade is confirmed executed. Its job: turn each execution into a durable, structured record that Performance Analytics can later read.

## Storage location

All entries live in a single memory file: **`/areas/trading-journal.md`**

Use Claude's memory tools (`memory_read`, `memory_write`, `memory_append`, `memory_str_replace`) to read and write it — this is what makes the journal persist across chat sessions, not just within one conversation.

- If the file doesn't exist yet, create it on the first logged trade with the frontmatter shown below.
- Every entry is tagged `[stated]`-equivalent in spirit (it's a factual record of what the user actually executed), but since this is transactional data, not a personal fact about the user, standard memory frontmatter still applies — see format below.

## Entry format

Each entry is numbered sequentially, zero-padded to 3 digits, and never reused even if a trade is later corrected — corrections get a note, not a renumber. Fields adapt to the trade type — a spot trade has no leverage or margin type, so those lines are simply omitted rather than shown as N/A.

Futures example:
```
### #001 — 2026-09-02 14:32 UTC — BUY BTC/USDT (Full Guardian)
- Action: Open long, 5x leverage, $800 (≈8.2% of equity at the time)
- Fill: 0.0091 BTC @ $87,420
- Guardian verdict at execution: "Moderate risk — elevated funding (0.06%)"
- Warnings raised: elevated funding rate flagged; user confirmed anyway
- Status: OPEN
```

Spot example (no leverage/margin fields):
```
### #002 — 2026-09-04 10:15 UTC — BUY BNB/USDT (Full Guardian)
- Action: Spot buy, $5 requested (≈62% of equity at the time)
- Fill: 0.006 BNB @ $719.99 (filled to nearest lot size, slightly under request)
- Guardian verdict at execution: "High-risk entry — concentrates ~74% of a small account into one asset"
- Warnings raised: concentration, no diversification left; user confirmed anyway
- Status: OPEN
```

When a position tied to an existing entry is later reduced or closed (detected in Trade Guardian Step 4 — portfolio/exposure check — by matching symbol + side against open journal entries), **update that same entry** rather than creating a new one:

```
### #001 — 2026-09-02 14:32 UTC — BUY BTC/USDT (Full Guardian)
- Action: Open long, 5x leverage, $800 (≈8.2% of equity at the time)
- Fill: 0.0091 BTC @ $87,420
- Guardian verdict at execution: "Moderate risk — elevated funding (0.06%)"
- Warnings raised: elevated funding rate flagged; user confirmed anyway
- Status: CLOSED — 2026-09-04 09:15 UTC
- Exit: 0.0091 BTC @ $89,100
- Realized P&L: +$15.29 (+1.9%)
```

A partial close gets a `Status: PARTIAL (X% reduced)` line and its own exit/P&L sub-line, but keeps the same entry number — don't split one position into multiple entries.

## User-facing confirmation

The entry format above is what gets written to memory — the user never sees that raw block. What they see is a one-line confirmation appended directly after the execution receipt from `references/trade-guardian-output.md` (same response, not a separate message):

```
📓 Logged as journal entry #[NNN].
```

For an update to an existing entry (reduce/close), reference the original entry number and show the realized P&L on that line, since that's the number the user actually wants to see in the moment:

```
📓 Journal entry #[NNN] closed — realized P&L: [+/-$X] ([+/-Y]%).
```

Nothing more elaborate than that — the full detail lives in the entry itself, retrievable later via Performance Analytics or a direct "show me entry #NNN" request; the inline confirmation is just a receipt that the record exists.

## What triggers a new entry vs. an update

- **New entry**: any trade that *opens* new exposure — a fresh buy, a new position, a convert. Also log spot buys/sells that aren't tied to an existing open position.
- **Update existing entry**: any trade that *reduces or closes* exposure already tracked under an open entry (matched by symbol + side). If no matching open entry exists (e.g. the position predates journal use, or was opened outside Sentinel), create a new entry noting `Status: CLOSED (opened outside Sentinel — no prior entry)` instead of fabricating an open-side history.

## User notes

If the user adds a comment about a trade (either right after execution or later, e.g. "note on #001: I got greedy on the leverage there"), append it to that entry as a `- User note:` line rather than creating a separate entry. This is what gives Performance Analytics its richest signal later.

## What NOT to log here

- Rejected or abandoned trades (user said no, or never confirmed) — these never get an entry, per `references/trade-guardian.md` design principles.
- Read-only queries (balance checks, market checks) — the journal is trade-execution history only, not an activity log.

## File size management

This file grows indefinitely with active trading, and memory files are size-capped. When a read shows the file is close to its cap:
- Don't stop logging new trades.
- Roll the oldest entries into a compact dated summary block (e.g. "Entries #001–#040 (Sep 2026): 28 wins / 12 losses, net +$340 — see exchange history for full detail") and keep full detail only for recent entries.
- Performance Analytics can still recover exact historical numbers from exchange trade history (its primary source, per `SKILL.md`'s cross-module data sharing section) — the journal's job after rollup is to preserve the *reasoning* trail for recent trades, not to be the permanent full ledger.

## Frontmatter (on file creation)

```
---
name: trading-journal
description: Structured log of every trade executed through Trade Guardian — entry/exit, Guardian verdict at the time, warnings raised, and user notes. Read by Performance Analytics as enrichment context; exchange trade history remains the source of truth for hard numbers.
sources: [sentinel-skill]
---
```
