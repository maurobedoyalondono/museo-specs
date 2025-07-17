# Geo-Location Environment Variables Setup

## API Environment Variables

Add the following to your API `.env` file:

```bash
# Geo-Location Service
GEO_LOCATION_ENABLED=true
IP_API_URL=http://ip-api.com/json
IP_API_TIMEOUT=5000
```

## Web Environment Variables

Add the following to your web `.env.local` file:

```bash
# Geo-Location
NEXT_PUBLIC_GEO_LOCATION_ENABLED=true
NEXT_PUBLIC_GEO_CACHE_DURATION=2592000000  # 30 days in ms (1 month)
```

## Database Migrations

To run the database migrations for LATAM currency support:

```bash
cd infrastructure
npm run migrate
```

This will run the following migrations:
1. `1752000000001_add_latam_exchange_rates.ts` - Adds exchange rates for 14+ LATAM currencies (safely handles existing rates)
2. `1752000000002_create_regional_pricebooks.ts` - Creates regional pricebooks for 19 LATAM countries
3. `1752000000003_add_regional_price_points.ts` - Adds initial price points with currency conversion

**Note**: The exchange rates migration has been updated to handle existing rates gracefully. It will:
- Deactivate any existing rates for the currency pairs being added
- Insert new rates with current timestamp
- Refresh the materialized view for current exchange rates

## Verification

After setting up the environment variables and running migrations:

1. Start the API server:
   ```bash
   cd api
   npm run dev
   ```

2. Start the web server:
   ```bash
   cd web
   npm run dev
   ```

3. Open browser DevTools and check the console for:
   - "GeoInitializer: Starting geo-location initialization..."
   - "GeoInitializer: Geo-location detection completed"

4. Check localStorage for the `geo-marketing-store` key with detected country/currency data

5. Test the API endpoint directly:
   ```bash
   curl http://localhost:8000/api/v1/geo/detect
   ```

## Supported LATAM Countries

The following countries are supported with their currencies:

- 🇨🇴 Colombia (COP)
- 🇲🇽 Mexico (MXN)
- 🇧🇷 Brazil (BRL)
- 🇦🇷 Argentina (ARS)
- 🇨🇱 Chile (CLP)
- 🇵🇪 Peru (PEN)
- 🇺🇾 Uruguay (UYU)
- 🇵🇾 Paraguay (PYG)
- 🇧🇴 Bolivia (BOB)
- 🇨🇷 Costa Rica (CRC)
- 🇬🇹 Guatemala (GTQ)
- 🇭🇳 Honduras (HNL)
- 🇳🇮 Nicaragua (NIO)
- 🇩🇴 Dominican Republic (DOP)
- 🇪🇨 Ecuador (USD)
- 🇻🇪 Venezuela (USD)
- 🇵🇦 Panama (USD)
- 🇸🇻 El Salvador (USD)
- 🇬🇾 Guyana (GYD)
- 🇸🇷 Suriname (SRD)
- 🇧🇿 Belize (BZD)
- 🇨🇺 Cuba (CUP)
- 🇵🇷 Puerto Rico (USD)

Plus support for:
- 🇺🇸 United States (USD)
- 🇨🇦 Canada (CAD)
- 🇬🇧 United Kingdom (GBP)
- 🇪🇺 European Union (EUR)