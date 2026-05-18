---
name: campaign-selection
version: 1.0.0
description: "Create and update campaign selection pools (HOTEL / FLIGHT / TRAIN / Bundle) via the campaign-selection MCP. Use this when the user asks to '建选品池 / 创建选品池 / 配置选品池 / 选品规则 / selection pool / hotel pool / flight pool / 招商池 / 好货池 / 手动上传选品 / manual upload pool', or asks to filter products by tags like country / city / star / zone / SSDR / class type / route / airline. Wraps the proven payload patterns and avoids the trial-and-error loop on misleading 500 errors."
---

# campaign-selection

Wrapper for the `campaign-selection` MCP tools. Encodes the working payload shapes for HOTEL and FLIGHT pools learned through hard trial-and-error, plus the error → cause map. Use this BEFORE invoking `mcp__campaign_selection__*` directly.

Reference doc: https://www.feishu.cn/docx/CDY5dIrO1oG6fRxG1DEc14u4nRg (Campaign Selection MCP — Hotel & Flight Pool Creation Issues & Playbook)

---

## 0. Required context to collect first

Before any upsert, confirm with the user:

| Field | Example | Notes |
|---|---|---|
| `mainCampaignId` | `"M5475"` | Main campaign ID |
| `subCampaignId` | `34644` | Sub campaign (numeric) |
| `productLine` | `HOTEL` / `FLIGHT` / `HOTEL_PACKAGE` / `TRAIN` | |
| `campaignPermissionHead.teams` | `["HK"]` | Team code |
| `campaignPermissionHead.themes` | `["HK_Double_day_local_mega"]` | **Required**, NEVER empty |
| `eid` (FLIGHT Manual Upload only) | `"TR039628"` | Operator EID |

If the user only mentions team but not theme, call `listSelectPoolInfo` first to discover existing themes; if still ambiguous, ASK.

---

## 1. HOTEL pool — `upsertHotelSelectionSchema`

### 1.1 Working payload

```json
{
  "mainCampaignId": "M5475",
  "subCampaignId": 34644,
  "hotelProductLine": "HOTEL",
  "schemaName": "<pool name>",
  "selectType": 1,
  "campaignPermissionHead": {"teams": ["HK"], "themes": ["HK_Double_day_local_mega"]},
  "launchConfig": {
    "groupType": 0,
    "groupSortType": 0,
    "intentionSort": 0,
    "productSortType": 1
  },
  "tabInfoConfigList": [{
    "tabName": "Rule 1",
    "dataSourceConfig": {"dataSourceType": 6, "hotelList": [], "sourcingRuleIdList": []},
    "selectionConfig": {
      "indicatorRuleList": [
        {"indicatorKey": "country_id", "operatorKey": 4, "operatorValue": "78"},
        {"indicatorKey": "city_id",    "operatorKey": 4, "operatorValue": "228,334"},
        {"indicatorKey": "star",       "operatorKey": 4, "operatorValue": "5"}
      ]
    }
  }]
}
```

`selectType`: 1 = hotel, 2 = room.

### 1.2 Critical do/don't

- DO wrap `dataSourceConfig` + `selectionConfig` inside each `tabInfoConfigList[i]`. Do NOT put them at the top level — save will report 200 but tags will not show in the UI.
- DO use `launchConfig.groupType` at the **top level**, not nested under `launchConfig.groupConfig`.
- DO include `intentionSort: 0` and (when relevant) `checkDateConfig` to avoid 500.

### 1.3 Indicator keys

- `country_id`, `city_id`, `star`
- `ssdr_before_subsidy`, `beat_before_subsidy`
- `tcom_review_type`, `tcom_product_score`
- `is_ppp`, `is_prebuy_prpt`
- `zone_id` — **the only accepted key for hotelzone**. NOT `hotelzone` / `zone_name` / `biz_zone_id`. The numeric ID (e.g. 658) is the value.

### 1.4 Operator keys

| Code | Op |
|---|---|
| 1 | `=` |
| 2 | `!=` |
| 3 | range |
| 4 | `in` |
| 5 | `not in` |
| 6 | `contain` |
| 7 | `not contain` |
| 8 | `like` |
| 9 | `>` |
| 10 | `>=` |
| 11 | `<` |
| 12 | `<=` |

Example — SSDR ≤ 20%: `{"indicatorKey":"ssdr_before_subsidy","operatorKey":12,"operatorValue":"0.2"}`

---

## 2. FLIGHT pool — `upsertSelectionPoolForApi`

### 2.1 Source → dataSourceType

| Source | dataSourceType |
|---|---|
| Sourcing Rule | 301 |
| Supplementary Pool | 302 |
| **Manual Upload** | **300** |

### 2.2 marketingScene values

`NORMAL` and `GENERAL_DEALS` work. Lowercase variants are rejected.

### 2.3 Manual Upload payload (verified)

```json
{
  "mainCampaignId": "M5475",
  "subCampaignId": 34644,
  "productLine": "FLIGHT",
  "marketingScene": "NORMAL",
  "poolName": "<pool name>",
  "eid": "TR039628",
  "campaignPermissionHead": {"teams": ["HK"], "themes": ["HK_Double_day_local_mega"]},
  "launchConfig": {
    "flightLaunchConfig": {
      "needKnock": true,
      "priceRefreshModel": 1,
      "priceRefreshStartTime": 1778774400000,
      "priceRefreshEndTime":   1778860800000
    },
    "groupConfig":    {"groupType": 6, "groupSortType": 0},
    "subGroupConfig": {"groupType": 7, "groupSortType": 0},
    "resourceConfig": {"productSortType": 5},
    "regionList": ["HK"]
  },
  "tabInfoConfigList": [{
    "tabName": "Rule 1",
    "dataSourceConfig": {
      "dataSourceType": 300,
      "flightUploadDataList": [
        {
          "regions": "HK",
          "departureCityCode": "HKG",
          "arrivalCityCode": "ATH",
          "tripType": "RT",
          "travelStartTime": "2026-05-15",
          "travelEndTime":   "2026-08-31",
          "maxIntervalDays": 30,
          "minIntervalDays": 2,
          "isDirect": 0,
          "airlineCode": "TK",
          "classType": "C"
        }
      ]
    }
  }]
}
```

### 2.4 Manual Upload row required fields

- `regions`, `departureCityCode`, `arrivalCityCode`, `tripType` (`RT` / `OW`)
- `travelStartTime` — must be today or future
- `travelEndTime`
- `maxIntervalDays`, `minIntervalDays`
- `isDirect` (0 = any, 1 = direct), `airlineCode`, `classType` (Y / S / C / F)

### 2.5 Critical do/don't (FLIGHT)

- DO include `eid` for Manual Upload. Without it the API returns the misleading `operator can not be null!` error indefinitely.
- DO include the full `flightLaunchConfig` object (`needKnock`, `priceRefreshModel`, refresh times). The same misleading error appears if you omit it.
- DO use `marketingScene` upper-case (`NORMAL` / `GENERAL_DEALS`).
- For non-Manual sources, route filtering uses indicator `departure_arrival_city_id_pair` with NUMERIC city_id pairs in `dep-arr,dep-arr,...` form (e.g. `73-1`). IATA codes are NOT accepted on the indicator path — resolve to city_id first or fall back to Manual Upload (300).

### 2.6 Group type integers (FLIGHT)

| Setting | Value | Meaning |
|---|---|---|
| `groupConfig.groupType` | 6 | Arrival Country (primary) |
| `subGroupConfig.groupType` | 7 | Arrival City (secondary) |

The MD reference does not enumerate the full mapping; ask the user when other dimensions are needed.

---

## 3. Error → cause map

| Error message | Real root cause |
|---|---|
| `campaign permission check params are absent.` (10100) | Missing `themes` in `campaignPermissionHead` |
| `Non-Bundle scenario requires dataSourceConfig` (40000) | `dataSourceConfig` placed at wrong level — must live inside each tab in `tabInfoConfigList` |
| `operator can not be null!` (500) | Missing `eid` (Manual Upload) OR missing `flightLaunchConfig` OR wrong-cased marketingScene. NOT about `operatorKey`. |
| `Wrong Invalid Departure Start Date: Data row 1` | `travelStartTime` is in the past |
| `Wrong MaxintervalDays required:Data Row 1` | `maxIntervalDays` missing in upload row |
| `The indicator type does not match the group type` | Wrong indicator key for the dimension (e.g. `hotelzone` instead of `zone_id`) |
| `Unsupport data source value: 304` | Invalid dataSourceType integer for that product line |

---

## 4. Recommended workflow

1. **Discover** — call `mcp__campaign_selection__listSelectPoolInfo` to confirm campaign, team, and existing themes.
2. **Build payload** from §1.1 (HOTEL) or §2.3 (FLIGHT). Resolve any IATA → city_id mappings up front.
3. **Upsert** — call `mcp__campaign_selection__upsertHotelSelectionSchema` (HOTEL) or `mcp__campaign_selection__upsertSelectionPoolForApi` (FLIGHT/TRAIN/Bundle).
4. **Verify** — call `mcp__campaign_selection__querySelectionPoolDetailInfo` and confirm tags, group settings, and data source took effect.
5. **On 500 error** — consult §3 first; do NOT cycle through `operatorKey` permutations.

---

## 5. Things to ASK rather than guess

- The `eid` for the operator (no way to derive)
- The exact theme name (case-sensitive, may be empty for read but required for write)
- IATA → city_id mappings if the user wants indicator-based filtering instead of Manual Upload
- Group type integers for dimensions other than Arrival Country / Arrival City
