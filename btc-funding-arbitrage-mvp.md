# BTC Funding Fee Arbitrage - MVP Plan

*Created: 2026-02-11*

## Overview

A minimal viable product for capturing funding rate arbitrage on Bitcoin perpetual futures. This is a focused subset of the [broader trading system architecture](https://orbb.li/blog/20260106-trading-system-architecture/) discussed previously.

## Strategy Summary

**Funding Fee Arbitrage** exploits the periodic funding payments in perpetual futures markets:
- Open opposing positions: long spot + short perpetual (or vice versa)
- Capture funding rate payments (typically every 8 hours)
- Maintain delta-neutral exposure to price movements
- Profit from the funding rate differential

## MVP Scope

### What's Included
- **Single asset**: BTC only
- **Two exchanges**: One for spot, one for perpetual futures (e.g., Coinbase spot + Binance/Bybit perpetuals)
- **Manual position sizing**: Start with fixed size positions
- **Basic monitoring**: Funding rates, position sync, P&L tracking
- **Simple risk checks**: Position limits, max leverage, balance checks

### What's Excluded (for now)
- Multi-asset support
- Automated rebalancing
- Complex execution algorithms (TWAP/VWAP)
- Cross-exchange arbitrage optimization
- Machine learning for funding rate prediction
- Full OMS/PMS infrastructure

## Technical Architecture

Simplified 3-layer approach for MVP:

### Layer 1: Market Access
- Exchange API clients (REST + WebSocket)
- Real-time funding rate monitoring
- Order execution and fills tracking
- Balance/position queries

### Layer 2: Strategy Logic
- Funding rate threshold detection (e.g., trigger when rate > 0.01%)
- Position opening/closing logic
- Delta neutrality maintenance
- P&L calculation

### Layer 3: Risk Management
- Pre-trade checks: max position size, leverage limits
- Post-trade monitoring: balance thresholds, divergence alerts
- Emergency shutdown conditions

## Implementation Components

### 1. Data Collection
**Goal**: Understand funding rate patterns before trading

- Script to fetch historical funding rates (past 3-6 months)
- Calculate basic statistics: mean, median, extremes
- Identify profitable threshold ranges
- Output: CSV/JSON dataset for analysis

**Deliverables**:
- `data_fetcher.py`: API client for historical data
- `funding_analysis.ipynb`: Jupyter notebook with visualizations
- `config.json`: Exchange credentials, API endpoints

### 2. Monitoring Dashboard
**Goal**: Real-time visibility before automation

- Live funding rate display for BTC perpetuals
- Position tracking (spot + perp)
- Delta exposure calculation
- Funding payment history

**Deliverables**:
- `monitor.py`: WebSocket connections to exchanges
- Simple web UI or terminal dashboard
- Alert system (Telegram/email) for threshold crossings

### 3. Manual Execution Tools
**Goal**: Assisted trading with safety checks

- CLI tool to execute paired trades
- Pre-flight checks: balances, size limits, rate verification
- Post-trade confirmation and logging
- Position sync validation

**Deliverables**:
- `trade_executor.py`: Order placement with confirmations
- `risk_checker.py`: Validation rules
- Trade log database (SQLite)

### 4. Basic Automation
**Goal**: Automated entry/exit at thresholds

- Automatic position opening when funding rate crosses threshold
- Automatic closing at target profit or time limit
- Continuous delta monitoring with rebalancing alerts
- Kill switch for emergency stops

**Deliverables**:
- `strategy_runner.py`: Main automation loop
- `position_manager.py`: Delta tracking and rebalancing
- Enhanced monitoring with auto-trade logs

## Success Metrics

### Technical Metrics
- API uptime > 99%
- Order fill rate > 95%
- Delta drift < 2% (maintain near-neutral)
- No missed funding payments

### Financial Metrics
- Positive net funding income after fees
- Sharpe ratio > 1.0 (after accounting for collateral cost)
- Max drawdown < 5%
- APR > 10% (stretch goal: 20%)

## Risk Considerations

### Technical Risks
- Exchange API downtime during funding time
- WebSocket disconnections losing position sync
- Race conditions in order execution
- Rate limit throttling

**Mitigations**:
- Backup API endpoints
- Position reconciliation checks every 5 minutes
- Order state machine with retry logic
- Request queuing with rate limit awareness

### Financial Risks
- Extreme price movements causing liquidation
- Funding rate flipping unexpectedly
- Withdrawal/transfer delays between exchanges
- Insufficient margin on perp exchange

**Mitigations**:
- Conservative leverage (2-3x max)
- Stop-loss at 5% portfolio drawdown
- Maintain 30%+ margin buffer
- Daily balance reconciliation

## Exchange Selection

### Recommended Pairs

**Option A: Coinbase + Binance**
- Spot: Coinbase Pro (good liquidity, USD pairs)
- Perp: Binance (highest volume, reliable funding)
- Pros: Established, good APIs
- Cons: Binance US restrictions

**Option B: Kraken + Bybit**
- Spot: Kraken (USD/EUR pairs, margin available)
- Perp: Bybit (competitive funding rates)
- Pros: Bybit strong API, Kraken compliance
- Cons: Lower volume on Kraken

**Option C: Single exchange (Bybit/OKX)**
- Both spot + perp on same platform
- Pros: Simpler balance management, instant transfers
- Cons: Counterparty concentration risk

## Development Roadmap

### Phase 1: Research
- Historical data analysis
- Funding rate pattern identification
- Exchange API testing
- Paper trading simulation

### Phase 2: Semi-Manual
- Monitoring tools
- Assisted execution
- Manual position management
- Performance tracking

### Phase 3: Automated MVP
- Threshold-based automation
- Risk management integration
- 24/7 monitoring
- Alerting system

### Phase 4: Refinement
- Strategy parameter optimization
- Transaction cost analysis
- Multi-timeframe analysis
- Scaling to larger positions

## Tech Stack

### Core
- **Language**: Python 3.11+
- **Exchange APIs**: `ccxt` library (unified interface)
- **Data storage**: SQLite (MVP), PostgreSQL (later)
- **Real-time**: `asyncio` + `websockets`

### Supporting
- **Monitoring**: Jupyter notebooks, `plotly` for dashboards
- **Alerts**: `python-telegram-bot` or email (SMTP)
- **Config**: YAML/JSON with environment variables
- **Logging**: `structlog` with JSON output

### Infrastructure (Minimal)
- **Hosting**: Single VPS (AWS/DigitalOcean)
- **OS**: Ubuntu 22.04 LTS
- **Process management**: `systemd` or `supervisor`
- **Secrets**: `.env` files (MVP), Vault (later)

## Next Steps

1. **Exchange account setup**: Open accounts, complete KYC, fund with test capital ($1k-5k)
2. **API key generation**: Create keys with trade + read permissions
3. **Data collection**: Run funding rate scraper to gather sufficient historical data
4. **Paper trading**: Simulate strategy on historical data
5. **Live testing**: Start with minimal capital ($500) to validate strategy
6. **Scale gradually**: Increase position size as confidence grows

## Open Questions

- [ ] Which exchange pair to start with?
- [ ] What's the minimum profitable funding rate threshold?
- [ ] How often to rebalance delta exposure?
- [ ] Should we use margin on spot side or just perps?
- [ ] Tax implications of frequent funding payments?

## Future Enhancements (Post-MVP)

- Multi-asset support (ETH, SOL, etc.)
- Cross-exchange optimization (triangular arb)
- Dynamic position sizing based on volatility
- Funding rate prediction models
- Automated rebalancing with execution algos
- Integration with full trading system architecture
- Risk dashboard with real-time portfolio analytics

---

*This MVP focuses on proving the strategy works profitably before building complex infrastructure. Start small, measure everything, scale gradually.*
