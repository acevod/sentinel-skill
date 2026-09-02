# Sentinel

A Claude skill that turns Claude into a full trading copilot for [Binance Agent OS](https://binance.com/agent-os) (Binance's MCP server for AI applications) — not just an order executor, but a system that watches your portfolio, challenges risky trades, reads market conditions, remembers every trade, and learns from your track record over time.

Built for Binance's Agent OS Mini Hackathon (Track A).

## Why "Sentinel"

The skill started as a single risk-check layer for trade execution. It grew into five modules covering the full loop a trader actually goes through: check state → get context → decide → act → record → learn. "Sentinel" is the umbrella name for that whole system.

## The five modules

| Module | Triggers on | What it does |
|---|---|---|
| **Portfolio Analysis** | "what's my balance", "how's my portfolio" | Full breakdown of holdings, exposure, P&L, and risk flags — never just a raw number |
| **Trade Guardian** | "buy $X of Y", "open a 5x long", "close my position" | Risk-aware execution: checks market + portfolio context, plays devil's advocate on risky trades, confirms before executing |
| **Market Intelligence** | "how's the market today", "worth trading today" | Broad market read — movers, funding rate landscape, overall risk-on/risk-off mood — independent of your own positions |
| **Trading Journal** | (automatic, after every executed trade) | Logs a structured, numbered entry — what executed, Guardian's verdict at the time, warnings raised — persisted in memory across sessions |
| **Performance Analytics** | "what's my win rate", "why do I keep losing" | Reads exchange trade history (source of truth for hard numbers) plus the journal (context for *why*) to surface real patterns, honestly — including saying "not enough data" when that's true |

`SKILL.md` is the router: it reads the user's request, decides which module(s) apply, and loads only the relevant reference file(s) — keeping each request fast instead of loading all five modules' logic every time.

## How Trade Guardian works

**Classify → Identify → Market check → Portfolio check → Devil's Advocate + Bull Case → Confirm → Execute → Verify → Receipt → hand off to Journal**

Every trade gets tiered — Simple, Light Guardian, or Full Guardian — based on thresholds you control (position size vs. equity, leverage, funding rate, position reduction %, concentration, momentum). Small, low-risk trades execute with no friction. Trades that cross a threshold get an explicit risk analysis and require confirmation before anything executes.

## Structure

```
sentinel/
├── SKILL.md                          # Router — decides which module(s) a request needs
├── LICENSE
├── README.md
└── references/
    ├── portfolio-analysis.md         # Module 1
    ├── trade-guardian.md             # Module 2 — workflow logic
    ├── trade-guardian-thresholds.md  #   ↳ numeric risk limits (edit to fit your risk tolerance)
    ├── trade-guardian-output.md      #   ↳ output formatting per tier
    ├── market-intelligence.md        # Module 3
    ├── trading-journal.md            # Module 4 — writes to /areas/trading-journal.md in memory
    └── performance-analytics.md      # Module 5 — reads journal + exchange history
```

## Requirements

- A Claude.ai account (Free, Pro, Max, Team, or Enterprise) with **Code execution and file creation** enabled in Settings → Capabilities
- **Memory** ("Generate memory from chats") enabled in Settings — the Trading Journal module depends on this to persist entries across sessions
- The [Binance Agent OS](https://binance.com/agent-os) MCP connector connected to your account

## Installation

1. Download or clone this repo
2. Zip the `sentinel/` folder (the folder itself as the root of the zip)
3. In Claude.ai, go to **Settings → Customize → Skills**
4. Click **"+"** → **"+ Create skill"** → **"Upload a skill"**
5. Upload the zip file, then toggle the skill on

No coding, no separate server, no API key management — Sentinel runs entirely inside Claude.ai on your existing plan.

## Customizing risk thresholds

Your risk tolerance isn't the same as anyone else's. Open `references/trade-guardian-thresholds.md` and adjust the numbers — position size %, leverage caps, funding rate bands, concentration limits — to match your own account size and trading style. `references/portfolio-analysis.md` reuses the same thresholds, so you only need to edit them in one place.

## A note on data sources

Sentinel deliberately does not treat the journal as its only record of your trading history. Hard numbers (win rate, realized P&L) always come from exchange trade history via the MCP — which is complete regardless of whether every trade went through Sentinel. The journal adds the reasoning layer exchange data can't provide: what was flagged, what you were warned about, what you noted yourself.

## Disclaimer

This skill is a workflow aid, not financial advice. It does not guarantee profitable trades, does not replace your own judgment, and its market/risk/performance analysis is only as good as the data available at the time of the request. Trading involves risk of loss, including the risk of total loss. Use at your own risk.

## License

[MIT](./LICENSE) — free to use, modify, and share.
