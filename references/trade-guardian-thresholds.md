# Trade Guardian — Thresholds

Numeric limits Trade Guardian uses to decide whether a trade is **simple**, **light Guardian**, or **full Guardian**. Edit the numbers here freely — this file doesn't affect the workflow logic in `references/trade-guardian.md`.

If any one threshold below crosses into "light" or "full," the whole trade escalates to that tier — tiers don't average across categories.

## 1. Position size vs account equity
| Range | Tier |
|---|---|
| < 5% of equity | Simple |
| 5–15% of equity | Light Guardian |
| > 15% of equity | Full Guardian |

## 2. Leverage (futures)
| Range | Tier |
|---|---|
| ≤ 3x | Simple |
| 4x – 10x | Light Guardian |
| > 10x | Full Guardian (always flag as high-risk regardless of size) |

## 3. Funding rate (futures, per 8h)
| Range | Treatment |
|---|---|
| -0.03% to 0.03% | Normal — don't mention |
| > 0.03% or < -0.03% (up to ±0.1%) | Elevated — raise in Devil's Advocate |
| > 0.1% or < -0.1% | Extreme — explicit warning, expensive to hold |

## 4. Position reduction / close size
| Range | Tier |
|---|---|
| < 30% of holding in that symbol | Simple |
| 30% – 70% | Light Guardian |
| > 70% (or closing to zero) | Full Guardian |

## 5. Concentration
- Single asset > 40% of total equity → flag concentration risk
- Correlated group (e.g. BTC + ETH + SOL as "crypto beta") > 60% of total equity → flag even if no single asset is individually over 40%

## 6. Momentum (Devil's Advocate input)
- 24h price move > ±7% in the *same direction* as the requested trade → flag as "chasing momentum"

## Defaults / fallback
- Default account base currency: USDT
- Default market: USDⓈ-M Futures (not COIN-M) for futures trades
- Default margin type: Cross
- If equity/portfolio data can't be retrieved, treat the trade as Light Guardian minimum — don't silently execute without exposure context.

## Notes
These are starting-point defaults, not fixed rules — revisit as account size, trading style, or risk tolerance changes.
