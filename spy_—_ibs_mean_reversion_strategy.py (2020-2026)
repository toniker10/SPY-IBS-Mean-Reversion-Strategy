# -*- coding: utf-8 -*-
import yfinance as yf
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
from datetime import datetime
import warnings
warnings.filterwarnings("ignore")

TICKER = "SPY"
START_DATE = "2020-01-01"
INITIAL_CAP = 10000.0
IBS_ENTRY = 0.20
IBS_EXIT = 0.80

# Load data
print(f"Downloading {TICKER} data from {START_DATE}...")
raw = yf.download(TICKER, start=START_DATE, auto_adjust=True, progress=False)
if isinstance(raw.columns, pd.MultiIndex):
    raw.columns = raw.columns.get_level_values(0)

data = raw[["Open", "High", "Low", "Close", "Volume"]].dropna().copy()

# Calculate IBS and Returns
rng = (data["High"] - data["Low"]).replace(0, np.nan)
data["IBS"] = ((data["Close"] - data["Low"]) / rng).fillna(0.5)
data["DailyReturn"] = data["Close"].pct_change().fillna(0)

# Backtest State Machine (Avoiding look-ahead bias)
n = len(data)
raw_signal = np.zeros(n, dtype=int)
in_trade = False
entry_arr = (data["IBS"] < IBS_ENTRY).values
exit_arr = (data["IBS"] > IBS_EXIT).values
signal_change_idx = []

for i in range(n):
    prev_state = in_trade
    if not in_trade:
        if entry_arr[i]:
            in_trade = True
    else:
        if exit_arr[i]:
            in_trade = False
    if in_trade != prev_state:
        signal_change_idx.append(i)
    raw_signal[i] = 1 if in_trade else 0

raw_position = pd.Series(raw_signal, index=data.index)
position = raw_position.shift(1).fillna(0)
strat_ret = position * data["DailyReturn"]
equity = (1 + strat_ret).cumprod()

bh_ret = data["DailyReturn"]
bh_equity = (1 + bh_ret).cumprod()

# Performance Metrics Function
def compute_metrics(df, position, strat_ret, equity):
    days_span = (df.index[-1] - df.index[0]).days
    years = days_span / 365.25 if days_span > 0 else (1 / 365.25)
    final_equity = equity.iloc[-1]
    cagr = (final_equity ** (1 / years) - 1) * 100 if final_equity > 0 else -100.0
    exposure = position.mean() * 100
    running_max = equity.cummax()
    drawdown = (equity - running_max) / running_max
    max_dd = drawdown.min() * 100
    ann_vol = strat_ret.std() * np.sqrt(252)
    sharpe = (strat_ret.mean() * 252) / ann_vol if ann_vol > 0 else 0.0

    trades = []
    in_trade_ = False
    trade_ret = 1.0
    pos_vals = position.values
    ret_vals = strat_ret.values
    for i in range(len(df)):
        if pos_vals[i] == 1:
            if not in_trade_:
                in_trade_ = True
                trade_ret = 1.0
            trade_ret *= (1 + ret_vals[i])
        else:
            if in_trade_:
                trades.append(trade_ret - 1)
                in_trade_ = False
    if in_trade_:
        trades.append(trade_ret - 1)
    trades = np.array(trades)
    
    n_trades = len(trades)
    if n_trades > 0:
        win_rate = (trades > 0).mean() * 100
        gross_profit = trades[trades > 0].sum()
        gross_loss = -trades[trades < 0].sum()
        profit_factor = gross_profit / gross_loss if gross_loss > 0 else np.inf
    else:
        win_rate, profit_factor = 0.0, 0.0

    return {
        "CAGR (%)": cagr, "Exposure (%)": exposure, "MaxDD (%)": max_dd,
        "Sharpe": sharpe, "WinRate (%)": win_rate, "ProfitFactor": profit_factor,
        "NumTrades": n_trades, "TotalReturn (%)": (final_equity - 1) * 100
    }, drawdown

strat_metrics, strat_dd = compute_metrics(data, position, strat_ret, equity)
bh_position = pd.Series(1, index=data.index)
bh_metrics, bh_dd = compute_metrics(data, bh_position, bh_ret, bh_equity)

# Print Results
print("=" * 60)
print(f"BACKTEST RESULTS ({data.index[0].date()} to {data.index[-1].date()})")
print("=" * 60)
for k in ["TotalReturn (%)", "CAGR (%)", "MaxDD (%)", "Sharpe", "WinRate (%)", "NumTrades"]:
    print(f"{k:>16}: {strat_metrics[k]:,.2f}")
print("=" * 60)

# Plotting
fig, axes = plt.subplots(2, 1, figsize=(12, 8), sharex=True)
axes[0].plot(data.index, equity * INITIAL_CAP, label="IBS Strategy", color="blue", linewidth=1.2)
axes[0].plot(data.index, bh_equity * INITIAL_CAP, label="SPY Buy & Hold", color="gray", alpha=0.7)
axes[0].set_title("Strategy Equity Curve vs SPY")
axes[0].set_ylabel("Portfolio Value ($)")
axes[0].legend()
axes[0].grid(True, alpha=0.3)

axes[1].fill_between(data.index, strat_dd * 100, 0, color="red", alpha=0.4, label="Drawdown")
axes[1].set_ylabel("Drawdown (%)")
axes[1].set_xlabel("Date")
axes[1].legend()
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
