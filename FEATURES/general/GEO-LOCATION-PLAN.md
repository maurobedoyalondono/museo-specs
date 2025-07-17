# GEO-LOCATION-PLAN.md - Geo-Location Currency Detection Implementation

## Overview
This document tracks the implementation of a geo-location feature that detects user's country from IP address and determines the appropriate currency for pricing. The system integrates with the existing pricing infrastructure that already supports geo_regions and multiple currencies.

## Architecture Analysis Summary

### Existing Infrastructure
1. **Database**:
   - Pricebooks table has `geo_regions` field for regional pricing
   - Exchange rates exist for USD, EUR, GBP
   - Pricing schema supports multi-currency with automatic conversion

2. **API**:
   - `PricingService.resolvePricebookWithFallback()` accepts `geoLocation.country`
   - Product endpoints already accept `geo_location` parameter
   - Clean Architecture pattern established

3. **Web**:
   - Zustand stores with localStorage persistence (AuthStore pattern)
   - Services pass currency to API (currently hardcoded or from env)
   - i18n context for locale (separate from geo-marketing)

### Identified Gaps
- No IP-to-country detection service
- No LATAM currencies in exchange rates
- No country-to-currency mapping
- Web doesn't detect or send geo_location
- No persistent storage of detected location

## Implementation Status Tracker

### Phase 1: Database - LATAM Currency Support ✅ COMPLETED

#### 1.1 Add LATAM Exchange Rates ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1752000000001_add_latam_exchange_rates.ts`
- [x] Create migration file
- [x] Add exchange rates for LATAM currencies:
  - [x] USD → COP (Colombian Peso) ~4100
  - [x] USD → MXN (Mexican Peso) ~17.50
  - [x] USD → BRL (Brazilian Real) ~5.10
  - [x] USD → ARS (Argentine Peso) ~850
  - [x] USD → CLP (Chilean Peso) ~950
  - [x] USD → PEN (Peruvian Sol) ~3.80
  - [x] USD → UYU (Uruguayan Peso) ~40
  - [x] USD → PYG (Paraguayan Guarani) ~7500
  - [x] USD → BOB (Bolivian Boliviano) ~6.90
  - [x] USD → CRC (Costa Rican Colón) ~520
  - [x] USD → GTQ (Guatemalan Quetzal) ~7.80
  - [x] USD → HNL (Honduran Lempira) ~24.50
  - [x] USD → NIO (Nicaraguan Córdoba) ~36.50
  - [x] USD → DOP (Dominican Peso) ~60
- [x] Add reverse rates (COP → USD, etc.)
- [x] Also added EUR and GBP to LATAM rates
- [ ] Run migration

#### 1.2 Create Regional Pricebooks ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1752000000002_create_regional_pricebooks.ts`
- [x] Create migration file
- [x] Add regional pricebooks:
  ```sql
  -- Pattern for each country
  ('Colombia COP Pricing', 'regional', 'COP', ARRAY['CO'], 3, NOW(), TRUE)
  ('Mexico MXN Pricing', 'regional', 'MXN', ARRAY['MX'], 3, NOW(), TRUE)
  -- etc for all LATAM countries
  ```
- [x] Set fallback to default USD pricebook
- [x] Created pricebooks for 19 LATAM countries
- [x] Also created VIP/Wholesale pricebooks for major markets (CO, MX, BR)
- [ ] Run migration

#### 1.3 Add Initial Regional Pricing ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1752000000003_add_regional_price_points.ts`
- [x] Create price points for each regional pricebook
- [x] Use exchange rate conversion from USD base prices
- [x] Added variant-specific pricing for major markets (COP, MXN, BRL)
- [x] Created VIP and Wholesale pricing with appropriate discounts
- [ ] Run migration

### Phase 2: API - Geo-Location Module ✅ COMPLETED

#### 2.1 Domain Layer ✅ COMPLETED
**Structure**:
```
/api/src/domain/geo/
  models/
    GeoLocation.ts
  services/
    IGeoLocationService.ts
```

- [x] Create `GeoLocation.ts` model:
  ```typescript
  export interface GeoLocationInfo {
    country: string;      // ISO code (CO, MX, BR)
    countryName: string;
    region: string;
    city?: string;
    currency: string;
    timezone?: string;
  }
  ```
- [x] Create `IGeoLocationService.ts` interface
- [x] Added country-currency mapping for all LATAM countries
- [x] Added error handling types and enums

#### 2.2 Infrastructure Layer ✅ COMPLETED
**Structure**:
```
/api/src/infrastructure/geo/
  services/
    IpApiService.ts
    CountryCurrencyMapper.ts
  config/
    geoLocationConfig.ts
```

- [x] Implement `IpApiService.ts`:
  - [x] Integration with ip-api.com
  - [x] Error handling for API failures
  - [x] Rate limiting considerations
  - [x] IP validation and extraction from headers
- [x] Create `CountryCurrencyMapper.ts`:
  - [x] Country to currency mapping utilities
  - [x] Default fallback logic
  - [x] Helper methods for regions and countries
- [x] Create `geoLocationConfig.ts` configuration

#### 2.3 Application Layer ✅ COMPLETED
**Structure**:
```
/api/src/application/geo/
  dto/
    GeoLocationDTO.ts
  usecases/
    DetectLocationUseCase.ts
    GetCountryInfoUseCase.ts
```

- [x] Create DTOs:
  - [x] DetectLocationRequest/Response
  - [x] CountryCurrencyInfoRequest/Response
  - [x] SupportedCountries/CurrenciesResponse
  - [x] GeoLocationDTOMapper utility
- [x] Implement `DetectLocationUseCase.ts`:
  - [x] IP extraction from Express request
  - [x] Service orchestration with caching
  - [x] Manual country override support
- [x] Implement `GetCountryInfoUseCase.ts`:
  - [x] Country information queries
  - [x] Supported countries/currencies lists

#### 2.4 Presentation Layer ✅ COMPLETED
**Files**:
- `/api/src/api/controllers/GeoLocationController.ts`
- `/api/src/api/routes/geoRoutes.ts`
- `/api/src/infrastructure/config/geoLocationContainer.ts`

- [x] Create controller with endpoints:
  - [x] `GET /api/v1/geo/detect` - Detect location from IP
  - [x] `GET /api/v1/geo/countries` - List supported countries
  - [x] `GET /api/v1/geo/countries/:code` - Get country info
  - [x] `GET /api/v1/geo/currencies` - List supported currencies
- [x] Add validation middleware with express-validator
- [x] Register routes in main app
- [x] Create container registration for services
- [x] Integrate with server initialization

### Phase 3: Web - Geo-Marketing Integration 🟡 IN PROGRESS

#### 3.1 Geo-Marketing Store ✅ COMPLETED
**File**: `/web/src/shared/store/GeoMarketingStore.ts`

- [x] Create store with state:
  ```typescript
  interface GeoMarketingState {
    country: string | null;
    currency: string | null;
    countryName: string | null;
    detectedAt: Date | null;
    expiresAt: Date | null;
    isLoading: boolean;
    error: string | null;
  }
  ```
- [x] Implement actions:
  - [x] `detectLocation()` with deduplication
  - [x] `setGeoData()`
  - [x] `clearGeoData()`
  - [x] `getCurrency()` with USD fallback
  - [x] `isExpired()` check
- [x] Add localStorage persistence (1 month expiry)
- [x] Added convenient selectors and initialization helper

#### 3.2 Geo-Location Service ✅ COMPLETED
**File**: `/web/src/shared/services/GeoLocationService.ts`

- [x] Create service class extending BaseAPIService
- [x] Implement `detectLocation()` method
- [x] Add country info methods
- [x] Add supported countries/currencies methods
- [x] Add type definitions
- [x] Check if feature is enabled

#### 3.3 Update Existing Services ✅ COMPLETED

**BasketService.ts Updates**:
- [x] Import GeoMarketingStore
- [x] Update `addToBasket()` to use detected currency
- [x] Update `getCountry()` to check GeoMarketingStore first
- [x] Added `getCurrency()` method that retrieves from store
- [x] Ensure all methods respect geo-marketing data

**ProductService.ts Updates**:
- [x] Import GeoMarketingStore
- [x] Update all product queries to include:
  - [x] `currency` parameter from store
  - [x] `geo_location` parameter when available
- [x] Added `getGeoData()` helper method
- [x] Updated `getFeaturedProducts()`, `searchProducts()`, `getProductDetail()`, and `getVariantDetail()`

#### 3.4 App Initialization ✅ COMPLETED
**File**: Created `/web/src/shared/components/organisms/GeoInitializer/`

- [x] Created GeoInitializer component similar to AuthInitializer
- [x] Added to layout.tsx after AuthInitializer
- [x] Checks if geo-location feature is enabled
- [x] Calls `initializeGeoDetection()` on app start
- [x] Auto-checks localStorage and validates expiry
- [x] Triggers detection only if needed (expired or missing data)

### Phase 4: Testing & Documentation ❌ NOT STARTED

#### 4.1 Unit Tests ❌ NOT STARTED
- [ ] Test CountryCurrencyMapper
- [ ] Test GeoLocationUseCase
- [ ] Test GeoMarketingStore
- [ ] Test service integrations

#### 4.2 Integration Tests ❌ NOT STARTED
- [ ] Test full geo-detection flow
- [ ] Test pricebook selection with geo_regions
- [ ] Test currency conversion
- [ ] Test fallback scenarios

#### 4.3 Documentation ❌ NOT STARTED
- [ ] Update API documentation
- [ ] Add usage examples
- [ ] Document country-currency mappings
- [ ] Add troubleshooting guide

## Technical Specifications

### Country-Currency Mapping
```typescript
export const COUNTRY_CURRENCY_MAP: Record<string, string> = {
  // North America
  'US': 'USD',
  'CA': 'CAD',
  'MX': 'MXN',
  
  // South America
  'BR': 'BRL',
  'AR': 'ARS',
  'CO': 'COP',
  'CL': 'CLP',
  'PE': 'PEN',
  'UY': 'UYU',
  'PY': 'PYG',
  'BO': 'BOB',
  'EC': 'USD',  // Dollarized
  'VE': 'USD',  // De facto USD
  'PA': 'USD',  // Uses USD
  'GY': 'GYD',
  'SR': 'SRD',
  
  // Central America
  'CR': 'CRC',
  'GT': 'GTQ',
  'HN': 'HNL',
  'NI': 'NIO',
  'SV': 'USD',  // Dollarized
  'BZ': 'BZD',
  
  // Caribbean
  'DO': 'DOP',
  'CU': 'CUP',
  'PR': 'USD',  // US Territory
  
  // Europe (existing)
  'GB': 'GBP',
  'DE': 'EUR',
  'FR': 'EUR',
  'ES': 'EUR',
  'IT': 'EUR',
  
  // Default
  'DEFAULT': 'USD'
};
```

### API Response Format
```json
{
  "success": true,
  "data": {
    "country": "CO",
    "currency": "COP",
    "countryName": "Colombia",
    "region": "South America",
    "detectedFrom": "ip"
  },
  "meta": {
    "timestamp": "2025-01-14T12:00:00Z"
  }
}
```

### Local Storage Structure
```json
{
  "geo-marketing-store": {
    "state": {
      "country": "CO",
      "currency": "COP",
      "countryName": "Colombia",
      "detectedAt": "2025-01-14T12:00:00Z",
      "expiresAt": "2025-02-14T12:00:00Z"
    },
    "version": 0
  }
}
```

### Environment Variables

#### API (.env)
```bash
# Geo-Location Service
GEO_LOCATION_ENABLED=true
IP_API_URL=http://ip-api.com/json
IP_API_TIMEOUT=5000
```

#### Web (.env.local)
```bash
# Geo-Location
NEXT_PUBLIC_GEO_LOCATION_ENABLED=true
NEXT_PUBLIC_GEO_CACHE_DURATION=2592000000  # 30 days in ms
```

## Integration Points

### PricingService Integration
The existing `resolvePricebookWithFallback` method already supports geo-location:
```typescript
const applicablePricebooks = await this.pricebookRepository.findApplicablePricebooks({
  currency: context.currency,
  geoRegion: context.geoLocation?.country,  // ← Uses country code
  customerTier: context.customerTier,
  channel: context.channel
});
```

### Product Endpoints Integration
Product controllers already accept geo_location:
```typescript
geoLocation: req.query.geo_location ? 
  JSON.parse(req.query.geo_location as string) : 
  undefined
```

## Success Criteria
- [x] User's country detected from IP on first visit (GeoInitializer runs on app start)
- [x] Appropriate currency selected based on country mapping (COUNTRY_CURRENCY_MAP implemented)
- [x] LATAM currencies show correct exchange rates (migration created)
- [x] Regional pricebooks selected automatically (geo_regions support added)
- [x] Products display prices in detected currency (ProductService updated)
- [x] Basket calculations use detected currency (BasketService updated)
- [x] Data persists in localStorage for 1 month (GeoMarketingStore with zustand persist)
- [x] System falls back to USD on detection failure (implemented in store)
- [x] No impact on existing locale/i18n functionality (separate from i18n context)
- [ ] Performance: Detection completes in <2 seconds (needs testing)

## Risk Mitigation
1. **IP API Failure**: Fallback to USD, cache results aggressively
2. **Exchange Rate Issues**: Use last known rates, alert on staleness
3. **Performance**: Use Redis cache for IP lookups
4. **Privacy**: Only store country, no personal data
5. **Testing**: Comprehensive test coverage before production

## Notes
- Keep geo-marketing separate from i18n/locale
- Use existing pricing infrastructure, don't duplicate
- Follow Clean Architecture patterns from existing modules
- Consider adding analytics events for geo-detection
- Plan for future: saved user preferences override detection