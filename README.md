# n8n Trading Workflows

Automated trading and market analysis workflows for n8n, using Trading 212, Ollama (local LLM), and Discord.

## Workflows

### 1. `trading212-autonomous-trader-v2.json`
Fully autonomous AI trading system for a UK Stocks & Shares ISA via Trading 212.
- Polls market data and analyses with Ollama (`qwen2.5:14b` / `32b`)
- Uses Tavily for live news search
- Sends trade signals + Discord approval requests before executing
- Weekly portfolio rebalancer + monitor

### 2. `swing-bot-v2-t212-demo.json`
Swing trading bot targeting 20-day positions on Trading 212 (demo account mode).
- Entry/exit signal generation via local LLM
- Position sizing based on portfolio value
- Discord notifications for all trade events

### 3. `global-market-picker.json`
Multi-market scanner running 3× daily during market hours (09:00, 13:00, 17:00 ET, Mon–Fri).
- Scans US stocks, UK stocks, Europe/Asia ETFs, catalysts, and crypto
- Feeds headlines into Ollama (`gemma4`) for opportunity ranking
- Posts top 10 opportunities to Discord `#market-scanner` channel

## Setup

### Prerequisites
- [n8n](https://n8n.io) (self-hosted)
- [Ollama](https://ollama.ai) running locally with `qwen2.5:14b` or `qwen2.5:32b` and `gemma4`
- Trading 212 account with API access
- Discord server with bot and webhook configured
- Tavily API key (for news search in autonomous trader)

### Import Steps
1. In n8n, go to **Workflows → Import from file**
2. Select the JSON file
3. Replace all `REPLACE_WITH_CREDENTIAL_ID` placeholders with your actual n8n credential IDs
4. Replace all `YOUR_DISCORD_GUILDID` / `YOUR_DISCORD_CHANNELID` values
5. Update `REPLACE_WITH_WEBHOOK_ID` with your Discord webhook ID
6. Activate the workflow

### Credential Placeholders
All credentials have been stripped for safe sharing. You will need to re-add:

| Placeholder | What it is |
|---|---|
| `REPLACE_WITH_CREDENTIAL_ID` | Your n8n credential ID |
| `REPLACE_WITH_CREDENTIAL_NAME` | Your n8n credential name |
| `REPLACE_WITH_WEBHOOK_ID` | Discord webhook ID |
| `YOUR_DISCORD_GUILDID` | Your Discord server (guild) ID |
| `YOUR_DISCORD_CHANNELID` | Your Discord channel ID |
| `REPLACE_WITH_INSTANCE_ID` | Your n8n instance ID |

## Disclaimer
These workflows are for **educational purposes**. Automated trading carries significant financial risk. Not financial advice.
