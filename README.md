# Crypto Watcher + Lovable UI

A real-time cryptocurrency price monitoring system built with Make.com 
for automation and Lovable for the frontend dashboard. The system fetches 
live crypto data from the CoinGecko public API, applies alert threshold 
logic, delivers notifications via Telegram and Slack, logs data to Google 
Sheets and displays everything on a responsive live dashboard.

---

## Live Demo

🌐 [View Live Dashboard](https://coin-hearth-monitor.lovable.app)

---

## System Architecture
Webhook (cron-job.org trigger)
├── Branch 1 — Price Monitoring
│   → CoinGecko /coins/markets
│   → Iterator (tracked coins)
│   → Sleep (rate limit handling)
│   → CoinGecko /coins/{id} (metadata)
│   → Google Sheets (Latest + History tabs)
│   → Filter (±5% threshold)
│   → Telegram Alert
│
└── Branch 2 — Trending Coins
→ CoinGecko /search/trending
→ Iterator (trending coins)
→ Slack Alert
→ Google Sheets (Latest + History tabs)

---

## Features

- **Live price monitoring** — tracks Bitcoin, Ethereum and Solana in real time
- **Price alert system** — notifies when a coin moves beyond a configurable threshold
- **Trending coin detection** — independently monitors globally trending coins on CoinGecko
- **Multi-channel notifications** — delivers alerts via Telegram and Slack
- **Google Sheets logging** — maintains a Latest tab for the dashboard and a History tab for audit trail
- **Lovable dashboard** — responsive frontend displaying monitored coins, trending coins, price change chart and alert panel
- **Webhook trigger** — scenario can be triggered on a schedule via cron-job.org or manually via the dashboard button
- **Auto refresh** — dashboard reads from Google Sheets CSV every 60 seconds

---

## Tech Stack

| Tool | Role |
|------|------|
| Make.com | Automation workflow |
| CoinGecko API | Live crypto market data |
| Google Sheets | Data storage and dashboard bridge |
| Telegram Bot | Price alert notifications |
| Slack | Trending coin notifications |
| Lovable | Frontend dashboard |
| cron-job.org | Webhook scheduler |
| Postman | API testing |

---

## Make.com Scenario Setup

### Prerequisites
- Make.com account (free tier)
- CoinGecko public API (no key required)
- Google Sheets with two tabs: **Latest** and **History**
- Telegram Bot token and Chat ID (via BotFather)
- Slack workspace with a connected channel
- cron-job.org account for scheduling

### Webhook Setup
1. Create a new Make.com scenario
2. Add **Webhooks → Custom webhook** as the first module
3. Copy the generated webhook URL
4. Register it on cron-job.org as a POST request on your desired schedule

### Branch 1 — Price Monitoring
1. Add **HTTP → Make a request** module
   - URL: `https://api.coingecko.com/api/v3/coins/markets`
   - Method: GET
   - Parameters: `vs_currency=usd`, `ids=bitcoin,ethereum,solana`, 
     `order=market_cap_desc`, `price_change_percentage=24h`
2. Add **Iterator** — point to `Data[]` from HTTP response
3. Add **Tools → Sleep** — set to 2000ms to avoid rate limits
4. Add second **HTTP module** — `https://api.coingecko.com/api/v3/coins/{{id}}`
5. Add **Google Sheets → Add a Row** — write to Latest and History tabs
6. Add **Filter** — `price_change_percentage_24h > 5 OR < -5`
7. Add **Telegram Bot → Send a Message** with coin data mapped

### Branch 2 — Trending Coins
1. Add **HTTP → Make a request** module
   - URL: `https://api.coingecko.com/api/v3/search/trending`
   - Method: GET
2. Add **Iterator** — point to `coins[]` from response
3. Add **Slack → Send a Message** with trending coin data
4. Add **Google Sheets → Add a Row** — write to Latest and History tabs

---

## Google Sheets Structure

### Latest Tab (read by dashboard)
Cleared and rewritten on every scenario run.

| Coin Name | Symbol | Price (USD) | 24h Change % | Market Cap | Volume | Last Updated | Alert Type |
|-----------|--------|-------------|--------------|------------|--------|--------------|------------|

### History Tab (audit log)
Appended on every scenario run — never cleared.

Same column structure as Latest tab.

### Publishing CSV for Lovable
1. File → Share → Publish to web
2. Select **Latest** tab
3. Select **CSV** format
4. Copy the published URL and use it in Lovable as the data source

---

## Lovable Dashboard

The frontend dashboard was built using [Lovable](https://lovable.dev) and 
reads data from the Google Sheets published CSV URL every 60 seconds.

### Dashboard Features
- **Header** — total market cap and last updated timestamp
- **Price Alerts panel** — highlights coins exceeding threshold with adjustable slider (1%–20%)
- **24h Price Change chart** — green/red bar chart toggleable between change % and volume
- **Monitored Coins** — cards for all Price Alert rows from Google Sheets
- **Trending Coins** — cards for all Trending Alert rows from Google Sheets
- **Run Monitor button** — triggers Make.com webhook on demand

---

## Environment Variables

No environment variables required. All connections are configured 
directly within Make.com's module settings.

---

## How to Run

1. Clone or fork this repository
2. Set up your Make.com scenario following the setup instructions above
3. Publish your Google Sheets Latest tab as CSV
4. Deploy the Lovable dashboard with your Google Sheets CSV URL and 
   Make.com webhook URL
5. Register your webhook URL on cron-job.org
6. Trigger the webhook manually or wait for the scheduled run

---

## Known Limitations

- CoinGecko free tier allows ~10-30 API calls per minute — Sleep module 
  mitigates rate limiting but tracking more than 5 coins may cause errors
- Make.com free plan provides 1,000 operations per month
- Google Sheets published CSV has up to 5 minutes cache delay

---

## Future Improvements

- Add SMS notifications via Twilio for urgent high-threshold alerts
- Integrate paid CoinGecko API key for higher rate limits
- Replace Google Sheets with Supabase for real-time dashboard updates
- Add historical price sparkline charts
- Add user authentication to the dashboard for personalised watchlists

---

## Author

**Okpara Kenneth Chinonso**  
[LinkedIn](https://linkedin.com/in/okparakenneth) | 
[GitHub](https://github.com/nonsosky)

---

## License

This project is open source and available under the [MIT License](LICENSE).
