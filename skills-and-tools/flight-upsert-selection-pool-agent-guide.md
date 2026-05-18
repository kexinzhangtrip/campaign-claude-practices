# upsertSelectionPool 机票选品池入参指南

本文档用于指导 Agent 调用 `upsertSelectionPool` / `upsertSelectionPoolForApi` 创建或更新机票选品池时构造 `UpsertSelectionPoolRequest`。

本文重点覆盖 `productLine=FLIGHT`。顶层 `productLine` 不要传 `HOTEL`；当前接口的通用入口不支持用 `HOTEL` 创建选品池。

## 构造原则

Agent 构造机票选品池入参时，按以下顺序组织字段：

1. 填写活动与权限信息：`campaignPermissionHead`、`mainCampaignId`、`subCampaignId`。
2. 固定产线：`productLine` 传 `FLIGHT`。
3. 填写选品池基本信息：`poolName`、`marketingScene`、可选的 `eid`、`schemaId`。
4. 按数据来源构造 `tabInfoConfigList`。
5. 按投放要求构造 `launchConfig`。
6. 如需要给选品池打标签，再传 `themeTags`、`customerTags`。

新建时不要传 `schemaId`，或传 `null`。更新时必须传已有 `schemaId`，并为已有 tab 传对应 `tabId`。

## 顶层字段

### `campaignPermissionHead`

权限归属信息。创建选品池时建议必传。

```json
{
  "themes": ["MY_Double_date_local_mega"],
  "teams": ["XX"]
}
```

传参逻辑：

- `themes`：主题权限列表，至少传 1 个。
- `teams`：团队权限列表，至少传 1 个。
- Agent 不知道具体值时，应从用户上下文、活动配置或已有线上报文中获取，不要编造。

### `eid`

操作者员工号。可选，但建议 Agent 显式传。

传参逻辑：

- 已知操作者时传真实 `eid`。
- 不传时服务端会尝试从当前 SSO 上下文获取。
- 自动任务、Agent 任务或非交互式调用建议传，避免取不到登录态。

### `schemaId`

选品池 ID。

传参逻辑：

- 新建：不传或传 `null`。
- 更新：传已有选品池 ID。
- 更新时 `tabInfoConfigList` 需要包含所有要保留的 tab；未传的旧 tab 可能被认为需要删除。

### `mainCampaignId` / `subCampaignId`

活动 ID 信息。

传参逻辑：

- `mainCampaignId`：主活动 ID，字符串，例如 `"M5664"`。
- `subCampaignId`：子活动 ID，数字，例如 `20707`。
- 有活动上下文时建议都传。

### `productLine`

固定传：

```json
"productLine": "FLIGHT"
```

### `poolName`

选品池名称。

传参逻辑：

- 必传。
- 不能为空。
- 最长 50 字符。
- 建议用能表达活动、场景、规则目的的名称。

### `marketingScene`

营销场景。

传参逻辑：

- 常见值：`NORMAL`。
- 若用户明确指定弱营销、特殊营销等场景，再按业务场景传对应值。
- 机票上传类数据在部分特殊场景下可能触发额外校验。

### `aiSourceFlag`

AI 来源标记。可选。

传参逻辑：

- Agent 创建的选品池如需标识来源，可传约定值。
- 没有明确约定时可省略。

## `tabInfoConfigList`

`tabInfoConfigList` 是选品规则的核心。机票选品池通常至少传 1 个 tab。

每个 tab 的通用结构：

```json
{
  "tabId": null,
  "tabName": "Rule 1",
  "dataSourceConfig": {
    "dataSourceType": 302
  },
  "selectionConfig": {
    "indicatorRuleList": []
  }
}
```

字段传参逻辑：

- `tabId`：新建 tab 不传或传 `null`；更新已有 tab 时传旧 tab ID。
- `tabName`：必传，不能为空，最长 50 字符。
- `dataSourceConfig`：必传，用于声明数据源类型。
- `selectionConfig.indicatorRuleList`：筛选规则列表；BI 规则池通常必须传。

### 机票常用 `dataSourceType`

- `300`：`FLIGHT_UPLOAD`，机票人工上传。使用 `flightUploadDataList` 提供航线/商品明细。
- `301`：`FLIGHT_MANUAL_SOURCE`，机票招商规则。通常使用 `sourcingRuleIdList`，也可叠加 `selectionConfig`。
- `302`：`FLIGHT_BI_SUPPORT`，机票 BI 数据源。常见线上规则池只传 `dataSourceType`，筛选条件放在 `selectionConfig.indicatorRuleList`。

Agent 选择逻辑：

- 用户给的是规则条件、指标筛选、BI 筛选：优先用 `302`。
- 用户给的是人工整理的航线/商品清单：用 `300`。
- 用户给的是招商规则 ID：用 `301` 并传 `sourcingRuleIdList`。

## `selectionConfig.indicatorRuleList`

指标规则结构：

```json
{
  "indicatorKey": "trip_type",
  "operatorKey": 1,
  "operatorValue": "RT"
}
```

字段传参逻辑：

- `indicatorKey`：指标 key。应使用系统已有 key，例如 `trip_type`、`class_type`、`departure_country_id`。
- `operatorKey`：操作符 key。不要猜测含义，优先复用已有报文或接口返回的操作符。
- `operatorValue`：操作值，统一按字符串传；多值通常用英文逗号拼接，例如 `"Y,S,C,F"`。
- `aggregateValueList`：仅级联地理类指标需要。普通指标不要传。

常见机票指标示例：

```json
[
  {
    "indicatorKey": "is_direct",
    "operatorKey": 1,
    "operatorValue": "1"
  },
  {
    "indicatorKey": "trip_type",
    "operatorKey": 1,
    "operatorValue": "RT"
  },
  {
    "indicatorKey": "class_type",
    "operatorKey": 4,
    "operatorValue": "Y,S,C,F"
  },
  {
    "indicatorKey": "departure_country_id",
    "operatorKey": 4,
    "operatorValue": "3"
  },
  {
    "indicatorKey": "arrival_country_id",
    "operatorKey": 4,
    "operatorValue": "1"
  }
]
```

使用建议：

- 不确定指标 key 时，先查询可用指标，或参考同活动/同产线已有线上报文。
- 不要把自然语言条件直接塞进 `operatorValue`；需要先转换成系统指标值。
- 多个规则之间默认按系统规则组合，Agent 只负责构造规则数组。

## `dataSourceConfig`

### BI 规则池：`dataSourceType=302`

适用于大多数规则筛选型机票选品池。

```json
{
  "dataSourceType": 302
}
```

传参逻辑：

- 只传 `dataSourceType` 即可。
- 筛选条件放到同一个 tab 的 `selectionConfig.indicatorRuleList`。
- 不需要传 `sourcingRuleIdList` 或上传明细。

### 招商规则池：`dataSourceType=301`

```json
{
  "dataSourceType": 301,
  "sourcingRuleIdList": [10001, 10002]
}
```

传参逻辑：

- 用户明确提供招商规则 ID 时使用。
- `sourcingRuleIdList` 传 long 数组。
- 如需叠加筛选，可同时传 `selectionConfig.indicatorRuleList`。

### 人工上传池：`dataSourceType=300`

```json
{
  "dataSourceType": 300,
  "flightUploadDataList": [
    {
      "regions": "US",
      "departureCityCode": "SHA",
      "arrivalCityCode": "BKK",
      "tripType": "OW",
      "travelStartTime": "2026-06-01",
      "travelEndTime": "2026-06-30",
      "maxIntervalDays": 30,
      "minIntervalDays": 0,
      "airlineCode": "MU",
      "isDirect": 1,
      "classType": "Y",
      "departureAirportCode": "PVG",
      "arriveAirportCode": "BKK"
    }
  ]
}
```

传参逻辑：

- 用户提供具体航线/商品明细时使用。
- `flightUploadDataList` 至少传 1 条。
- `departureCityCode`、`arrivalCityCode`、`tripType`、`travelStartTime`、`travelEndTime`、`classType` 等字段应来自用户或上游数据。

可选字段：

- `bogoPriceType`
- `instantDiscountConfig`
- `baggageTypeConfig`
- `specialPriceConfig`
- `transferCityCode`
- `transferAirportCode`
- `hotelId`
- `roomId`
- `checkInDate`
- `checkOutDate`
- `nightsOfRoom`
- `passengerAccount`
- `uspLabel`

## `launchConfig`

`launchConfig` 控制投放分组、排序、区域和机票敲价配置。没有复杂要求时也建议传基础结构。

完整结构：

```json
{
  "groupConfig": {
    "groupType": 31,
    "additionGroupType": ["China-HK", "China-Macao", "China-TW"],
    "groupSortType": 4
  },
  "subGroupConfig": {
    "groupType": 32,
    "groupSortType": 6
  },
  "resourceConfig": {
    "productSortType": 1
  },
  "regionList": ["NP", "AE", "MX", "PK"],
  "flightLaunchConfig": {
    "needKnock": true,
    "priceRefreshModel": 1,
    "priceRefreshStartTime": 1779120000000,
    "priceRefreshEndTime": 1779724800000
  }
}
```

### `groupConfig`

一级分组配置。

字段传参逻辑：

- `groupType`：一级分组类型。按业务已有枚举值传，例如线上报文中的 `31`。
- `groupSortType`：一级分组排序方式。按业务已有枚举值传。
- `additionGroupType`：附加分组配置，可选。常见值为 `China-HK`、`China-Macao`、`China-TW`。
- `intentionSort`：当前接口转换逻辑未正确从 request 赋值到领域 DTO，Agent 创建机票规则池时可不传。

简单无分组场景可以传：

```json
{
  "groupType": 0,
  "groupSortType": 0
}
```

### `subGroupConfig`

二级分组配置。

字段传参逻辑：

- 没有二级分组要求时，可传 `groupType=0`、`groupSortType=0`。
- 有二级分组要求时，按业务已有枚举传 `groupType` 和 `groupSortType`。
- 如二级分组也有附加分组，可传 `additionGroupType`。

### `resourceConfig`

资源排序配置。

```json
{
  "productSortType": 1
}
```

传参逻辑：

- `productSortType` 按投放排序要求传。
- 没有明确要求时，可参考同场景线上报文；简单场景常见传 `0` 或业务默认值。

### `regionList`

投放区域列表。

```json
"regionList": ["NP", "AE", "MX", "PK"]
```

传参逻辑：

- 传 region/locale code 数组。
- 单区域也用数组，例如 `["HK"]`。
- 多区域按用户指定或活动配置传。

### `flightLaunchConfig`

机票敲价与价格刷新配置。

```json
{
  "needKnock": true,
  "priceRefreshModel": 1,
  "priceRefreshStartTime": 1779120000000,
  "priceRefreshEndTime": 1779724800000
}
```

字段传参逻辑：

- `needKnock`：是否需要敲价。
- `priceRefreshModel`：
  - `0`：no refresh
  - `1`：normal
  - `2`：flash sale
  - `3`：key campaign
- `priceRefreshStartTime` / `priceRefreshEndTime`：毫秒时间戳。
- 不需要敲价或刷新时，可传 `needKnock=false`、`priceRefreshModel=0`，时间传 `null` 或省略。

## `themeTags` / `customerTags`

选品池标签。可选。

```json
{
  "themeTags": [
    {
      "tagId": 123
    }
  ],
  "customerTags": [
    {
      "tagId": 456
    }
  ]
}
```

传参逻辑：

- 只需要传 `tagId`。
- `themeTags` 表示主题标签。
- `customerTags` 表示人群标签。
- 没有标签需求时可不传或传空数组。

## 推荐报文模板

下面是 Agent 创建机票 BI 规则选品池时可参考的完整模板。按用户需求替换活动、权限、规则、投放配置即可。

```json
{
  "campaignPermissionHead": {
    "themes": ["MY_Double_date_local_mega"],
    "teams": ["XX"]
  },
  "eid": "operator-eid",
  "mainCampaignId": "M5664",
  "subCampaignId": 20707,
  "productLine": "FLIGHT",
  "poolName": "poolName",
  "marketingScene": "NORMAL",
  "tabInfoConfigList": [
    {
      "tabName": "Rule 1",
      "dataSourceConfig": {
        "dataSourceType": 302
      },
      "selectionConfig": {
        "indicatorRuleList": [
          {
            "indicatorKey": "is_direct",
            "operatorKey": 1,
            "operatorValue": "1"
          },
          {
            "indicatorKey": "trip_type",
            "operatorKey": 1,
            "operatorValue": "RT"
          },
          {
            "indicatorKey": "class_type",
            "operatorKey": 4,
            "operatorValue": "Y,S,C,F"
          },
          {
            "indicatorKey": "departure_country_id",
            "operatorKey": 4,
            "operatorValue": "3"
          },
          {
            "indicatorKey": "arrival_country_id",
            "operatorKey": 4,
            "operatorValue": "1"
          }
        ]
      }
    }
  ],
  "launchConfig": {
    "groupConfig": {
      "groupType": 31,
      "additionGroupType": ["China-HK", "China-Macao", "China-TW"],
      "groupSortType": 4
    },
    "subGroupConfig": {
      "groupType": 32,
      "groupSortType": 6
    },
    "resourceConfig": {
      "productSortType": 1
    },
    "regionList": ["NP", "AE", "MX", "PK"],
    "flightLaunchConfig": {
      "needKnock": true,
      "priceRefreshModel": 1,
      "priceRefreshStartTime": 1779120000000,
      "priceRefreshEndTime": 1779724800000
    }
  },
  "themeTags": [],
  "customerTags": []
}
```

## Agent 检查清单

调用前逐项检查：

- `productLine` 是否固定为 `FLIGHT`。
- 新建时是否未传 `schemaId`；更新时是否传了已有 `schemaId`。
- `poolName` 是否非空且不超过 50 字符。
- `campaignPermissionHead.themes` 和 `campaignPermissionHead.teams` 是否至少各有 1 个值。
- `tabInfoConfigList` 是否至少有 1 个 tab。
- 每个 tab 是否有非空 `tabName` 和 `dataSourceConfig.dataSourceType`。
- `dataSourceType=302` 时，是否已把规则放入 `selectionConfig.indicatorRuleList`。
- `dataSourceType=300` 时，是否已传 `flightUploadDataList`。
- `dataSourceType=301` 时，是否已传 `sourcingRuleIdList`。
- `launchConfig.groupConfig`、`resourceConfig`、`regionList` 是否按投放要求填写。
- `flightLaunchConfig` 的时间是否为毫秒时间戳。
- 不确定枚举值时，是否参考了同活动、同 region、同场景的线上报文。

## 常见错误

- 顶层 `productLine` 传了 `HOTEL`：当前入口不支持。
- `poolName` 或 `tabName` 为空。
- `poolName` 或 `tabName` 超过 50 字符。
- 非 BUNDLE 场景漏传 `dataSourceConfig`。
- `dataSourceType` 不是服务端支持的值。
- 指标 key、操作符或值不是系统支持的组合。
- 人工上传场景漏传 `flightUploadDataList`。
- 更新时漏传需要保留的旧 tab。
