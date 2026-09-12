# SPY IBS Mean Reversion Strategy

A daily-bar Monte Carlo–free, rules-based mean reversion backtest on SPY (1993–Present), built to be leak-free: every trade uses only information available at the time it would have been placed. Includes a matching TradingView Pine Script strategy so results can be cross-checked on an independent platform.

## Strategy Rules

**IBS (Internal Bar Strength)** = `(Close - Low) / (High - Low)` — measures where the close sits within the day's range (0 = closed at the low, 1 = closed at the high).

| Signal | Condition | Action |
|---|---|---|
| Entry | IBS < 0.20 | Buy at next bar's open |
| Exit  | IBS > 0.80 | Close position at next bar's open |

The idea: a day that closes near its low (IBS near 0) after a selloff tends to mean-revert; a day that closes near its high (IBS near 1) signals the reversion has played out.

## No Look-Ahead Bias

Signals are computed from bar *t*'s close, but the position is shifted forward one day (`position.shift(1)`) before being applied to returns — so a signal generated today can only affect tomorrow's trade, never today's. The companion Pine Script strategy fills orders on the bar *after* the signal bar by default, which is the same execution convention. This is verified to produce matching logic on both platforms.

## Results

Backtest period: **1993-01-29 to 2026-09-11** (33+ years, full SPY history).

| Metric | IBS Strategy | SPY Buy & Hold |
|---|---|---|
| Total Return (%) | 5,415.08 | 3,069.58 |
| CAGR (%) | 12.67 | 10.83 |
| Max Drawdown (%) | -26.06 | -55.19 |
| Sharpe Ratio | 0.97 | 0.65 |
| Win Rate (%) | 69.26 | N/A¹ |
| Profit Factor | 1.93 | N/A¹ |
| Number of Trades | 989 | 1 |

¹ *Win Rate and Profit Factor aren't meaningful for Buy & Hold, since holding for 33 years counts as a single "trade" by definition.*

**Key takeaway:** the IBS strategy delivers a higher CAGR than passive Buy & Hold while cutting max drawdown roughly in half (-26% vs -55%) and nearly 1.5x the Sharpe ratio — i.e. similar-or-better returns for meaningfully less risk, at the cost of being out of the market part of the time (see Limitations).

## Installation

```bash
git clone https://github.com/toniker10/SPY-IBS-Mean-Reversion-Strategy.git
cd SPY-IBS-Mean-Reversion-Strategy
pip install -r requirements.txt
python spy_ibs_mean_reversion_strategy.py
```

## TradingView Version

The `Pine Script` file replicates the exact same entry/exit logic for use directly on TradingView's SPY daily chart, so the strategy can be validated against a second, independent backtesting engine.

## Limitations

- Backtested on adjusted daily close/OHLC data from Yahoo Finance; real-world fills, slippage, and commissions will differ from this simplified model (commission is set to 0 by default in the Pine Script — adjust to your broker's actual rate before treating results as realistic).
- Sharpe ratio is computed without subtracting a risk-free rate (simplified/"raw" Sharpe).
- Single-asset, single-parameter-set backtest — no walk-forward validation or out-of-sample testing, so results may be overfit to this specific historical window.
- Past performance does not indicate future results.

## Disclaimer

This project is for educational and research purposes only. It is not investment, trading, or financial advice.

## Author

toniker10
