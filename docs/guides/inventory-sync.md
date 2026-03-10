# Inventory Sync

The seller agent needs an inventory catalog to serve buyer queries, generate
pricing, and create deals. Inventory can come from a live ad server or from
built-in mock data for development.

---

## How It Works

The `ProductSetupFlow` runs at startup and performs these steps:

1. Initialize the seller organization (from `SELLER_ORGANIZATION_ID` / `SELLER_ORGANIZATION_NAME`)
2. Check for ad server credentials
3. **If GAM credentials exist** -- sync inventory via the GAM REST API
4. **If no credentials** -- create mock inventory packages for development
5. Create default product definitions with pricing and deal types

---

## Option A: Google Ad Manager (GAM)

### Prerequisites

- A GAM network code (the numeric ID for your GAM network)
- A service account JSON key with GAM API access
- The service account must have the `https://www.googleapis.com/auth/admanager` scope

### Environment Variables

```bash
AD_SERVER_TYPE=google_ad_manager
GAM_ENABLED=true
GAM_NETWORK_CODE=12345678
GAM_JSON_KEY_PATH=/path/to/service-account.json
GAM_APPLICATION_NAME=AdSellerSystem   # Optional, default: AdSellerSystem
GAM_API_VERSION=v202411               # Optional, default: v202411
```

### Sync Process

When GAM credentials are configured, the flow:

1. Creates a `GAMRestClient` and connects using the service account
2. Calls `list_inventory()` to fetch all ad units from GAM
3. Classifies each ad unit by inventory type (see table below)
4. Groups ad units by type and creates **Layer 1 (synced) packages**
5. Assigns estimated base CPMs per inventory type
6. Sets floor prices at 70% of base CPM
7. Stores packages in the storage backend

### Inventory Classification

Ad unit names are matched against keywords to determine inventory type:

| Inventory Type | Keyword Matches | Ad Formats | Device Types | Base CPM |
|---------------|----------------|------------|-------------|----------|
| `display` | *(default -- no other match)* | `banner` | PC, Phone, Tablet | **$12.00** |
| `video` | `video`, `preroll`, `midroll` | `video` | PC, Phone, Tablet | **$25.00** |
| `ctv` | `ctv`, `ott`, `connected` | `video` | CTV, Set Top Box | **$35.00** |
| `native` | `native`, `feed` | `native` | PC, Phone, Tablet | **$10.00** |
| `mobile_app` | `app`, `mobile` | `banner`, `video` | Phone, Tablet | **$18.00** |
| `linear_tv` | `linear`, `broadcast`, `tv `, `cable` | `video` | CTV, Set Top Box | **$40.00** |

Classification is case-insensitive and based on the ad unit's `name` field.

### Example

An ad unit named `"Premium CTV Streaming - Living Room"` would:

- Match keyword `ctv` -> classified as `ctv`
- Get ad format `video`, device types CTV + Set Top Box
- Base CPM of **$35.00**, floor price of **$24.50** (70% of base)

---

## Option B: Mock Inventory (Development)

When no ad server credentials are configured (`GAM_NETWORK_CODE` is unset and
`FREEWHEEL_API_URL` is unset), the flow creates 4 mock packages:

| Package | Type | Base CPM | Floor CPM | Device Types | Featured |
|---------|------|----------|-----------|-------------|----------|
| Display Network Bundle | display | $12.00 | $8.00 | PC, Phone, Tablet | No |
| Video Suite | video | $25.00 | $18.00 | PC, Phone, Tablet | No |
| CTV Premium Bundle | ctv | $35.00 | $28.00 | CTV, Set Top Box | Yes |
| NBCU Linear TV Broadcast Bundle | linear_tv | $40.00 | $28.00 | CTV, Set Top Box | Yes |

Each mock package includes realistic placements, IAB content categories, audience
segments, and geo targets (US).

The flow also creates default product definitions for finer-grained inventory:

| Product | Type | Base CPM | Floor CPM |
|---------|------|----------|-----------|
| Premium Display - Homepage | display | $15.00 | $10.00 |
| Standard Display - ROS | display | $8.00 | $5.00 |
| Pre-Roll Video | video | $25.00 | $18.00 |
| CTV Premium Streaming | ctv | $35.00 | $28.00 |
| Mobile App Rewarded Video | mobile_app | $20.00 | $15.00 |
| Native In-Feed | native | $12.00 | $8.00 |
| NBC Primetime :30 | linear_tv | $55.00 | $40.00 |
| NBCU Cable Network :30 | linear_tv | $22.00 | $15.00 |
| Telemundo Primetime :30 | linear_tv | $18.00 | $12.00 |
| Comcast Local Avails -- Top 10 DMAs | linear_tv | $15.00 | $8.00 |
| Comcast Addressable Linear -- National | linear_tv | $55.00 | $40.00 |
| Programmatic Linear Reach -- A25-54 | linear_tv | $30.00 | $20.00 |

---

## Option C: FreeWheel (Planned)

!!! warning "Planned Feature"
    FreeWheel integration (seller-dcd) is in the roadmap. The `FREEWHEEL_API_URL`
    and `FREEWHEEL_API_KEY` settings are declared in configuration but the
    integration client is not yet implemented. Currently only GAM is supported
    for live inventory sync.

---

## Manual Package Sync

You can trigger an inventory re-sync at any time via the API:

```bash
curl -X POST http://localhost:8000/packages/sync
```

This re-runs the `ProductSetupFlow.sync_from_ad_server()` step, fetching fresh
data from your configured ad server (or regenerating mock data if none is configured).

---

## Current Limitations

!!! note "Sync Limitations"
    The current inventory sync has the following constraints:

    - **No scheduled re-sync** -- Sync runs at flow startup or on manual trigger.
      There is no built-in cron or periodic sync.
    - **Automatic classification only** -- Ad unit type is inferred from the name.
      There is no manual mapping or override mechanism.
    - **Estimated CPMs** -- Base CPMs are estimated by inventory type, not pulled
      from GAM rate cards or historical data.
    - **No incremental sync** -- Each sync is a full re-import. Changed or
      removed ad units are not detected differentially.
    - **Floor prices are computed** -- Floor = 70% of base CPM. There is no
      per-ad-unit floor configuration.

    Planned improvements:

    - Scheduled periodic sync (configurable interval)
    - Manual inventory type mapping / override API
    - Rate card integration for accurate base CPMs
    - Incremental sync with change detection
    - FreeWheel ad server support
