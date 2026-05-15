# GoldTrack — COMEX Gold Intelligence Module

Built by Daniel Anthony. Tracks COMEX warehouse stocks, delivery notices, open interest, ETF holdings, LBMA vault data, institutional flows, and DXY correlation for gold markets.

---

## What's in here

```
goldtrack/
├── backend/          # Rust (axum) API server — all gold data routes
│   └── src/
│       ├── main.rs   # Server entry point, route definitions, DB connection
│       ├── routes.rs # All API handlers
│       ├── sync.rs   # Data sync functions (CME, ETF, LBMA, OI, DXY)
│       └── db.rs     # DB schema init (creates all tables)
├── frontend/         # React + TypeScript components
│   └── src/
│       ├── App.tsx               # Root app (standalone)
│       └── components/           # All dashboard components
│           ├── WarehouseStocks.tsx
│           ├── PriceChart.tsx
│           ├── OpenInterest.tsx
│           ├── ETFHoldings.tsx
│           ├── LBMAVault.tsx
│           ├── FirmFlowHeatmap.tsx
│           ├── InstitutionalActivity.tsx
│           ├── NetPositioning.tsx
│           ├── DeliveryPace.tsx
│           ├── BasisSpread.tsx
│           ├── GoldVsDxy.tsx
│           ├── MarketSignal.tsx
│           ├── DailyBrief.tsx
│           ├── MetalsSummary.tsx
│           ├── HistoricalComparisonChart.tsx
│           ├── AlertBanner.tsx
│           └── Layout.tsx
└── scripts/
    └── cme-download.mjs  # Downloads CME XLS/PDF files
```

---

## Integrating into the trading-app platform

### Step 1 — Backend routes

Copy `goldtrack/backend/src/` files into your backend. The backend uses **axum** — if your backend is actix-web, the route handlers need adapting (same logic, different extractor syntax).

Add these routes to your router under `/api/gold/`:

```
GET /api/gold/cme/latest-stocks
GET /api/gold/cme/summary
GET /api/gold/cme/latest-notices
GET /api/gold/cme/vault-breakdown
GET /api/gold/cme/firm-flows
GET /api/gold/cme/sync          ← triggers CME data download
GET /api/gold/prices/latest
GET /api/gold/prices/sync
GET /api/gold/prices/signal-history
GET /api/gold/etf/holdings
GET /api/gold/etf/sync
GET /api/gold/lbma/latest
GET /api/gold/lbma/sync
GET /api/gold/oi/latest
GET /api/gold/oi/sync
GET /api/gold/dxy/latest
GET /api/gold/dxy/sync
GET /api/gold/cme/institutional/latest
GET /api/gold/cme/institutional/top-traders
GET /api/gold/cb/reserves
GET /api/gold/cb/sync
GET /api/gold/export/csv
```

### Step 2 — Database tables

Run `db::init_db(&pool).await` on startup — this creates all required tables automatically (warehouse_stocks, metals_summary, delivery_notices, vault_breakdown, price_history, etf_holdings, lbma_vault, open_interest, central_bank_reserves, dxy_history, institutional_positions).

All tables use PostgreSQL. Same Cloud SQL instance works fine.

### Step 3 — Environment variables

Add to your `.env`:
```
# Already set if you're on the same DB — just uses the same PGHOST/PGDATABASE/PGUSER/PGPASSWORD
```

No extra env vars needed — GoldTrack uses the same PostgreSQL credentials as the rest of the platform.

### Step 4 — Frontend

Copy `goldtrack/frontend/src/components/` into your frontend as `src/goldtrack/components/`.

Add a new tab to DashboardV2:
```tsx
// In the MENU_GROUPS definition, add a new group:
goldtrack: {
  label: 'Gold',
  tabs: [
    { id: 'goldtrack', label: 'Gold Dashboard' },
  ],
},

// Add the TabType:
type TabType = ... | 'goldtrack';

// Add the render:
{activeTab === 'goldtrack' && <GoldTrackDashboard apiBase="http://localhost:8080/api/gold" />}
```

Import Layout and App from `src/goldtrack/` — or use individual components directly inside a wrapper div.

### Step 5 — CME Data Sync

The sync script downloads XLS/PDF files from CME's FTP. Run from `goldtrack/scripts/`:
```bash
node cme-download.mjs
```

Or trigger via the API:
```
GET /api/gold/cme/sync
```

This fetches the latest warehouse stocks and delivery notices directly from CME.

---

## Data Sources

| Data | Source | Sync endpoint |
|------|--------|---------------|
| COMEX warehouse stocks | CME Group FTP | `/api/gold/cme/sync` |
| Delivery notices | CME Group | `/api/gold/cme/sync` |
| Gold/Silver prices | Alpha Vantage / Yahoo Finance | `/api/gold/prices/sync` |
| ETF holdings (GLD, IAU, PHYS) | ETF provider data | `/api/gold/etf/sync` |
| LBMA vault data | LBMA website | `/api/gold/lbma/sync` |
| Open interest | CME | `/api/gold/oi/sync` |
| DXY correlation | FRED / Yahoo Finance | `/api/gold/dxy/sync` |
| Central bank reserves | WGC / IMF | `/api/gold/cb/sync` |

---

## Questions

Contact Daniel Anthony.
