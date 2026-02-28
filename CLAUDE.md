# BK P2P Aggregator - Phase 1 MVP

Read REQUIREMENTS.md for full specs. Build Phase 1: Monitoring Dashboard.

## What to Build

A web dashboard that aggregates P2P crypto exchange rates (JPY ↔ USDT/BTC/ETH) from multiple exchanges into one screen.

### Backend (Node.js + TypeScript)
1. **P2P Rate Fetcher** - Fetch P2P order book data from these 3 exchanges first:
   - **Bybit P2P**: `https://api2.bybit.com/fiat/otc/item/online` (POST, JSON body with tokenId, currencyId=JPY, side=1 for buy/0 for sell)
   - **OKX P2P**: `https://www.okx.com/v3/c2c/tradingOrders/books` (GET, params: quoteCurrency=JPY, baseCurrency=USDT, side=buy/sell)
   - **Binance P2P**: `https://p2p.binance.com/bapi/c2c/v2/friendly/c2c/adv/search` (POST, JSON body with fiat=JPY, tradeType=BUY/SELL, asset=USDT)
   
   For each exchange, fetch:
   - Price (rate)
   - Available quantity
   - Min/Max order limits
   - Merchant info (name, completion rate, order count, online status)
   - Payment methods
   
2. **Express API Server** (port 3003):
   - `GET /api/rates` - All exchanges' current rates
   - `GET /api/rates/:exchange` - Specific exchange rates  
   - `GET /api/best` - Best buy/sell rates across all exchanges
   - `GET /api/spread` - Spread comparison
   - `GET /api/arbitrage` - Arbitrage opportunities
   - `GET /api/status` - System health
   
3. **Rate Update Scheduler** - Fetch rates every 30 seconds, store in memory + SQLite for history

4. **SQLite DB** for rate history and later features

### Frontend (Single-page, served by Express)
Build with vanilla HTML/CSS/JS (no framework needed for MVP). Dark theme.

Main dashboard showing:
- **Rate Ranking Table**: All exchanges sorted by best rate (buy and sell tabs)
  - Columns: Rank, Exchange, Rate, Available Amount, Merchant Completion Rate, Payment Methods
- **Spread Comparison Bar**: Visual comparison of spreads
- **Arbitrage Alert Box**: If price difference > ¥1 between exchanges
- **Auto-refresh**: Every 30 seconds with countdown timer
- **Currency Tabs**: USDT / BTC / ETH toggle
- **Filters**: Min amount, payment method (bank transfer only), merchant rating

### Key Technical Notes
- Use `node-fetch` or `axios` for API calls
- Some P2P APIs may need specific headers (User-Agent, etc.) - handle gracefully
- Rate limit: Don't hammer APIs, respect intervals
- Error handling: If one exchange is down, still show others
- All times in JST (Asia/Tokyo)

### File Structure
```
bk-p2p-aggregator/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts              # Express server entry
│   ├── config.ts             # Configuration
│   ├── fetchers/
│   │   ├── base.ts           # Base fetcher interface
│   │   ├── bybit.ts          # Bybit P2P fetcher
│   │   ├── okx.ts            # OKX P2P fetcher
│   │   └── binance.ts        # Binance P2P fetcher
│   ├── services/
│   │   ├── aggregator.ts     # Rate aggregation logic
│   │   ├── scheduler.ts      # Rate update scheduler
│   │   └── db.ts             # SQLite operations
│   ├── routes/
│   │   └── api.ts            # API routes
│   └── types.ts              # TypeScript types
├── public/
│   ├── index.html            # Dashboard
│   ├── style.css             # Dark theme styles
│   └── app.js                # Frontend JS
├── data/                     # SQLite DB storage
└── REQUIREMENTS.md
```

Build everything, make it run with `npm run dev`. Make it look professional with a dark trading terminal aesthetic.

When completely finished, run: openclaw system event --text "Done: BK P2P Aggregator Phase 1 MVP built - dashboard with Bybit/OKX/Binance P2P rate monitoring" --mode now
