# live-traded-MQL5-EA-WannaBeQuant-Strategy-Selection

## WannaBeQuant Strategy Selection EA

**WannaBeQuant Strategy Selection EA** is a multi-strategy automated trading Expert Advisor built for **MetaTrader 5 / MQL5**.

Original MT5 product: https://www.mql5.com/en/market/product/137847?source=Site+Profile+Seller

This EA has been live-traded for 1+ year by the author and is designed as an example of a structured, production-style MQL5 trading system. It combines multiple configurable crypto and stock strategies, database-backed trade tracking, equity monitoring, drawdown protection, and timer-based execution.

> **Risk Disclaimer**  
> This project is for educational and research purposes only. Automated trading involves substantial risk. Past performance, including live trading history, does not guarantee future results. Always backtest, forward-test, and understand the code before using it on a live account.

---

## Overview

At a high level, this EA:

- Loads several independent trading strategies
- Supports crypto and stock trading logic
- Uses technical indicators such as MACD, Stochastic, ATR, Ichimoku, Bollinger Band Width, ROC, ADX, and price envelopes
- Runs on a timer instead of every market tick
- Tracks equity and drawdown
- Opens trades when strategy conditions are met
- Stores and manages trade metadata using a local database
- Closes positions based on strategy exits, stop-loss/take-profit rules, expiry logic, or drawdown conditions

In plain English:

> This is a multi-strategy automated trading EA that trades crypto and stocks, sizes positions relative to account settings, records its own trades in a database, monitors account drawdown, and opens/closes trades based on technical indicator signals.

---

## Key Features

### Multi-Strategy Trading Engine

The EA includes several configurable strategies, including:

| Strategy | Market | Type | Default Status |
|---|---:|---|---|
| Strategy 8 | Crypto | Volatility / Mean Reversion | Enabled |
| Strategy 13 | Crypto | Ichimoku Trend | Enabled |
| Strategy 11 | Crypto | Turtle / Breakout Trend | Enabled |
| Strategy 4 | Stocks | MACD / Stochastic / EMA | Enabled |
| Strategy 5 | Stocks | MACD / ATR / Stochastic | Enabled |
| Strategy 3 | Stocks | ROC / ATR | Disabled by default |

Each strategy has its own inputs, symbol list, max trade limits, spread threshold, indicator parameters, and trade comment.

---

## How It Works

### 1. Startup Validation

When the EA starts, `OnInit()` performs several checks before allowing the system to run.

It validates:

- General EA settings
- Strategy-specific inputs
- Magic number
- Recording interval
- Drawdown settings
- Max order limits
- Symbol configuration
- Database position tracking

If the EA detects a serious mismatch between live open positions and the internal database records, it refuses to start. This helps prevent unmanaged or orphaned trades from being ignored.

---

### 2. Timer-Based Execution

The EA does not rely on `OnTick()` for its main trading loop.

Instead, it uses a timer-based system with:

```mql5
EventSetTimer(RecordingInterval);
```

The default recording interval is commonly set to 60 seconds. This means the EA periodically checks market conditions, account equity, strategy signals, and open trades rather than reacting to every tick.

This can make the EA easier to control and less noisy than tick-by-tick systems.

---

### 3. Equity and Drawdown Monitoring

The EA records account equity and compares current equity against recent maximum equity.

It uses drawdown-related thresholds such as:

```mql5
drawdownThreshold_percentage
drawdownThreshold_percentage_OPEN
```

These controls help determine whether the EA should:

- Continue opening trades
- Stop opening new trades
- Trigger protective close logic
- Mark drawdown conditions using internal flags

Important internal flags include:

```mql5
equityDDlow
equityDD_hit_close
```

This allows the EA to reduce trading activity when equity conditions become unsafe.

---

### 4. Trade Entry Logic

Trade execution is handled through a dedicated trade entry function.

Before opening a trade, the EA checks:

- Strategy conditions
- Spread limits
- Max trade limits
- Lot size calculation
- Account margin
- Symbol availability
- Risk-related settings

When a trade is opened, the EA stores important metadata so that the trade can be managed later.

Stored trade information includes:

- Position ticket ID
- Symbol
- Strategy number
- Order number
- Side: buy or sell
- Entry price
- Take-profit
- Stop-loss
- Expiry date
- Lot size
- Additional numeric strategy data

---

### 5. Database-Backed Position Management

The EA uses its own local storage/database logic to track active trades.

This helps the EA know:

- Which strategy opened the trade
- Which symbol the trade belongs to
- What exit rules apply
- Whether the trade is still valid
- Whether a position has been manually closed
- Whether the EA database and live MT5 account are still in sync

This is one of the major design features of the system.

---

### 6. Trade Exit Logic

Positions can be closed under several conditions:

- Stop-loss condition
- Take-profit condition
- Strategy-specific exit rule
- Timer/countdown expiry
- Drawdown protection trigger
- Manual-close reset logic
- Database mismatch handling

After a trade is closed, the EA removes or updates the related database records so the internal state stays clean.

---

## Main Technologies

- **MQL5**
- **MetaTrader 5**
- **CTrade**
- **PositionInfo**
- **SQLite-style local trade tracking**
- **Timer-based EA architecture**
- **Multi-strategy technical indicator framework**

---

## Risk Controls

The EA includes several risk and safety mechanisms:

- Max total order limit
- Strategy-level max trade limits
- Spread threshold filters
- Relative lot sizing
- Drawdown-based trade blocking
- Drawdown-based close logic
- Database validation on startup
- Position ID reconciliation
- Magic number filtering
- Input validation before trading starts

---

## Example General Inputs

Some important general settings include:

```mql5
baseOrderSizeMultiplyer = 4.0
max_orders = 100
drawdownThreshold_percentage = 0.96
drawdownThreshold_percentage_OPEN = 0.97
MAGIC_NUMBER = 64537
RecordingInterval = 60
```

These values should be reviewed carefully before use.

---

## Example Strategy Inputs

Each strategy has its own configuration block.

Example strategy-level settings include:

```mql5
crypto_volatility_8 = true
crypto_strat_13_bool = true
crypto_turtle_11 = true
STOCKS_ROC_ATR_strat_4 = true
STOCKS_MACD_ATR_strat_5 = true
STOCKS_ROC_ATR_strat_3 = false
```

---

