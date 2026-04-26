# 🤖 n8n Trading Bots — Autonomous ISA & Swing Trading

A collection of fully autonomous trading bots built with n8n, Ollama (local AI), and the Trading 212 API. These bots run 24/7 on a self-hosted n8n instance and make trading decisions using local LLMs — no cloud AI costs.

> ⚠️ **Disclaimer:** These bots are for educational purposes. Trading involves risk. Never invest money you cannot afford to lose. This is not financial advice.

---

## 📦 Workflows

### 1. `trading212-isa-bot.json` — ISA Buy Bot
Autonomous UK Stocks & Shares ISA portfolio manager targeting 15% annual growth.

**How it works:**
- Runs at 8:30am and 4pm weekdays
- Fetches live portfolio and cash balance from Trading 212
- Runs 4x Tavily research searches (ETFs, growth stocks, market outlook, dips)
- Scores market risk with a bubble detector (0-15 scale)
- Checks economic calendar for high-impact events
- Sends all context to Ollama (qwen2.5:14b) for trade decisions
- Places fractional share orders via T212 API
- Logs all trades to CSV on Your-NAS NAS
- Posts trade notifications to Discord via Del-Boy bot

**Allocation targets:**
- 20% VUAGl_EQ (S&P 500 Accumulating)
- 10% ISFl_EQ (FTSE 100)
- 10% SWLDl_EQ (MSCI World)
- 15% EQQQl_EQ (Nasdaq 100)
- 10% VFEMl_EQ (Emerging Markets)
- 5% VERXl_EQ (Europe ex-UK)
- 30% Individual growth stocks (PLTR, AMD, SOFI, IONQ, RKLB etc)

---

### 2. `swing-bot.json` — Swing Trading Bot (Demo Account)
Hourly dip scanner running on a T212 Demo account with £5000 virtual cash.

**How it works:**
- Runs every hour 9am-3pm weekdays
- Fetches dynamic watchlist from T212 instruments API (50+ volatile stocks)
- Pulls 30-day price history from Yahoo Finance for each stock
- Calculates RSI, 5-day change, volume spike, and dip score
- Runs sentiment check via Yahoo Finance RSS headlines
- Scores each stock 0-100 — only buys above threshold
- Bubble detector adjusts position sizes based on market risk
- Circuit breaker halts trading if daily limits hit
- 4% profit target / 2.5% stop loss
- EOD P&L report to Discord at 4pm daily
- Weekly performance report every Friday 5pm

---

### 3. `weekly-rebalancer.json` — Weekly Portfolio Rebalancer
Runs every Monday at 9am to review and rebalance the ISA portfolio.

**How it works:**
- Reviews all open positions vs target allocations
- Sells underperformers (down >15% with weak fundamentals)
- Buys underweighted positions
- Posts weekly review to Discord

---

### 4. `market-picker.json` — Global Market Scanner
Hourly global market scanner that posts top opportunities to Discord — information only, no trades placed.

**How it works:**
- Runs every hour 9am-9pm weekdays
- Scans US stocks, UK stocks, Europe/Asia, crypto via Yahoo Finance RSS and CoinGecko
- Gemma3:4b analyses all headlines and identifies top 10 opportunities
- Posts ranked opportunities to Discord with risk level and suggested action

---

## 🛠️ Setup

### Prerequisites
- n8n self-hosted (v2.12+)
- Ollama with `qwen2.5:14b` and `gemma3:4b` models
- Trading 212 account with API access
- Discord bot token
- Tavily API key (free tier: 1000 searches/month)

### Credentials needed
After importing each workflow, set up these credentials in n8n:

| Credential | Used by | Where to get |
|-----------|---------|--------------|
| Trading 212 API | All trading bots | T212 app → Settings → API |
| Ollama API | All AI nodes | Your Ollama host IP |
| Discord Bot | All Discord nodes | Discord Developer Portal |
| Tavily API | Research nodes | app.tavily.com |

### Environment variables
Set in your n8n `.env` or docker-compose:
```
NODES_EXCLUDE=[]
N8N_RUNNERS_ENABLED=true
```

### File paths
The bots write logs to:
```
/home/your-username/mounts/your-nas-trading/isa_trades.csv
/home/your-username/mounts/your-nas-trading/swing_trades.csv
/home/your-username/mounts/your-nas-trading/watchlist.json
```
Update these paths in the Execute Command nodes to match your setup.

---

## 📊 Architecture

```
Trading 212 API
    ↓
n8n Workflow
    ↓
Tavily / Yahoo Finance RSS (market research)
    ↓
Ollama Local LLM (trade decisions)
    ↓
Trading 212 API (place orders)
    ↓
Discord (notifications via Del-Boy bot)
    ↓
Your-NAS NAS (CSV trade logs)
```

---

## 🔧 Key nodes explained

### Build Ollama Prompt
Packages all market research, portfolio data, cash balance and risk assessment into a structured prompt for the LLM.

### Parse Trades
Extracts the JSON trade decisions from Ollama's response, validates tickers against a whitelist, and filters out any trades exceeding available cash.

### Circuit Breaker
Reads the trade log CSV and halts trading if:
- Daily trade limit exceeded (4 buys/day for ISA, configurable for swing)
- Cash balance below minimum threshold

### Bubble Detector
Fetches VIX/market data and scores current market risk 0-15. Adjusts position sizes:
- Score 0-4: Normal (100% position size)
- Score 5-7: Caution (75%)
- Score 8-9: Elevated (50%)
- Score 10-12: Euphoria (25%)
- Score 13-15: Critical (0% — no trades)

---

## 📈 Performance tracking

Trade logs are stored as CSV with columns:
```
Date, Ticker, Action, Price, Amount, Quantity
```

The Weekly Report node reads this CSV and calculates:
- Total P&L
- Win rate
- Profit factor
- Sharpe ratio estimate

---

## ⚙️ Customisation

### Change target growth rate
In `Build Ollama Prompt`, update the system prompt:
```
targeting 15% annual growth
```

### Add/remove tickers
In `Filter Volatile Instruments` (swing bot), update `volatileKeywords` array.

In `Build Ollama Prompt` (ISA bot), update the `VERIFIED T212 UK TICKERS` list.

### Adjust risk thresholds
In `Calculate Dip Score` (swing bot), update:
```javascript
const DIP_THRESHOLD = 35; // RSI threshold
const MIN_DROP = 0.05;    // 5% minimum drop
```

---

## 🤝 Credits
Built by [@your-github-username](https://github.com/your-github-username) using:
- [n8n](https://n8n.io) — workflow automation
- [Ollama](https://ollama.ai) — local LLM inference
- [Trading 212 API](https://t212public-api-docs.redoc.ly/) — broker
- [Tavily](https://tavily.com) — AI search
- [Yahoo Finance](https://finance.yahoo.com) — market data
