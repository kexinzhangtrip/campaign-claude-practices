# Skill: Build Airline Campaign Page

## 触发条件

当用户提到"搭建航司活动页面"、"创建航司页面"、"新建活动页面"或类似意图时，激活此 Skill。

---

## 执行流程

### Step 1 — 确认是否为航司活动页面

如果用户明确表示是航司活动页面，主动提示：

> 你可以提供一个已有的参考页面 URL（如 trip.com 落地页），我会参考其组件结构来搭建新页面。如果没有，可以直接跳过，我会使用标准航司活动模板。

等待用户回复（填 URL 或跳过）。如提供 URL，使用 `get_page_from_url` 获取组件结构，仅参考组件搭配方式，不复制数据。

---

### Step 2 — 收集必须信息

**以下 6 项为必须信息，缺任何一项都不能开始创建，需追问直到完整：**

| # | 字段 | 说明 |
|---|------|------|
| 1 | `promoId` | 活动项目 ID（数字） |
| 2 | `locale` | 页面支持的语言/地区，可多个（如 en-TH, zh-HK） |
| 3 | 页面名称 | 每个 locale 各自的页面名称 |
| 4 | 页面布局 | 自然语言描述（如"顶部 banner，分 Tab 展示欧洲/美洲机票"） |
| 5 | 货品信息 | 产线（FLIGHT / HOTEL / ANT）+ 对应选品池 ID（pool ID） |
| 6 | 页面文案 | 各模块的标题、副标题、正文等文案内容 |

> 用户表达不清晰时（如未说明几个 locale、哪个 pool 对应哪个 Tab），主动追问，不要猜测。

---

### Step 3 — 确定页面布局结构

#### 布局组装原则

**以基础 DSL 模板为起点，结合用户自然语言描述进行调整：**

- 用户未提及的部分 → 保留模板默认结构
- 用户明确要增加的楼层 → 在合适位置插入新组件，sort 值在相邻组件之间取中间值或递增
- 用户明确不需要的楼层 → 从模板中删除对应组件
- 用户描述了不同的顺序 → 调整对应组件的 sort 值

#### 默认产线楼层顺序

若页面包含多个产线，楼层顺序固定为：**机票（FLIGHT）→ 酒店（HOTEL）→ ANT**

#### 布局决策规则

| 用户描述 | 对应组件结构 |
|---------|-------------|
| 多个目的地分组展示（如欧洲/美洲） | `cloud-trip-common-tab` + `cloud-trip-common-tab-pane`，每个 pane 内放产品组件 |
| 顶部有可跳转导航 | `cloud-component-sales-tabs-navigation-bar-v2`，`linkedStructureID` 指向对应楼层 ID |
| 单一产线不分组 | 直接放产品组件，无需 Tab 包裹（模板默认） |
| 有航司介绍图文内容 | `cloud-trip-common-photos-text` |
| 有"查看更多"广告位 | `cloud-component-sales-more-sales-banner`（模板默认包含） |

---

### Step 4 — 组件选型速查

直接使用以下映射，**无需查询知识库**即可完成选型。如需了解某组件完整 props，再查 campaign-component 知识库。

| 位置/功能 | 组件名称 |
|----------|---------|
| 页面根节点 | `@ctrip/sales-page` |
| 顶部 Banner 头图 | `@ctrip/cloud-component-trip-promo-banner` |
| 顶部导航跳转栏 | `@ctrip/cloud-component-sales-tabs-navigation-bar-v2` |
| 标题文字（h1/h2） | `@ctrip/cloud-component-trip-promo-headline` |
| 正文文字块 | `@ctrip/cloud-component-trip-promo-block-text` |
| Tab 容器 | `@ctrip/cloud-trip-common-tab` |
| Tab 分页 | `@ctrip/cloud-trip-common-tab-pane` |
| **机票产品楼层（FLIGHT）** | `@ctrip/trip-sales-component-flight-products` |
| **酒店产品楼层（HOTEL）** | `@ctrip/cloud-component-sales5-hotel-products` 或 `@ctrip/cloud-component-sales-hotel-hot-deal` |
| **ANT 产品楼层** | `@ctrip/trip-sales-component-ant-products` |
| 图文介绍块 | `@ctrip/cloud-trip-common-photos-text` |
| 更多活动广告位 | `@ctrip/cloud-component-sales-more-sales-banner` |
| 条款声明 | `@ctrip/cloud-component-promo-terms` |

#### 机票产品组件关键配置

- 选品池来源：`merchandiseSetting.sourceType = "pool"`，配置用户提供的 pool ID
- 卡片布局默认：`widgetLayout: "flightHotDeal"`
- 默认显示数量：`defaultVisibleCardCount: { mobile: 4, online: 8 }`
- 标题模式默认：`titleSubtitleMode: "default"`

---

### Step 5 — 组装页面 Schema 并创建

**以下为标准航司活动页面基础 DSL 模板。组装时以此为起点，结合用户自然语言描述进行增删调整：**

```json
{
  "id": "stru_page_tkairline",
  "label": "page",
  "name": "@ctrip/sales-page",
  "type": "react.component",
  "version": "",
  "props": {
    "background": { "color": "{{PAGE_BG_COLOR}}" }
  },
  "directive": {
    "tpl": "{{__templates:cont_E2qP26CsShge21i:schemas}}"
  },
  "extension": { "sort": 100 },
  "children": [
    {
      "id": "stru_banner_tk",
      "label": "banner",
      "name": "@ctrip/cloud-component-trip-promo-banner",
      "type": "react.component",
      "version": "",
      "directive": {},
      "extension": { "parentId": "stru_page_tkairline", "sort": 100 },
      "props": {
        "backgroundColor": "{{PAGE_BG_COLOR}}",
        "bannerHeight": 350,
        "bannerMobileHeight": 385,
        "bannerMobileURL": "{{BANNER_MOBILE_URL}}",
        "bannerMobileWidth": 750,
        "bannerURL": "{{BANNER_PC_URL}}",
        "curveEdge": false
      }
    },
    {
      "id": "stru_nav_tk",
      "label": "navigation",
      "name": "@ctrip/cloud-component-sales-tabs-navigation-bar-v2",
      "type": "react.component",
      "version": "",
      "directive": {},
      "extension": { "parentId": "stru_page_tkairline", "sort": 200 },
      "props": {
        "floorBackground": "{{PAGE_BG_COLOR}}",
        "tabTextColor": "#ffffff",
        "tabs": [
          {
            "id": "1",
            "linkedStructureID": "stru_headline2_tk",
            "tabLabel": "{{NAV_TAB_1_LABEL}}"
          }
        ]
      }
    },
    {
      "id": "stru_headline1_tk",
      "label": "intro headline",
      "name": "@ctrip/cloud-component-trip-promo-headline",
      "type": "react.component",
      "version": "",
      "directive": {},
      "extension": { "parentId": "stru_page_tkairline", "sort": 300 },
      "props": {
        "htmlTag": "h1",
        "textColor": "#ffffff",
        "title": "{{INTRO_HEADLINE}}"
      }
    },
    {
      "id": "stru_text_tk",
      "label": "intro text",
      "name": "@ctrip/cloud-component-trip-promo-block-text",
      "type": "react.component",
      "version": "",
      "directive": {},
      "extension": { "parentId": "stru_page_tkairline", "sort": 400 },
      "props": {
        "backgroundColor": "rgba(255,255,255,0)",
        "fontSize": 18,
        "fontSizeMobile": 13,
        "spacingBottom": 24,
        "spacingLeft": 16,
        "spacingRight": 16,
        "spacingTop": 0,
        "text": "{{INTRO_TEXT}}",
        "textColor": "#ffffff",
        "textLineHeight": 1.3
      }
    },
    {
      "id": "stru_headline2_tk",
      "label": "flight headline",
      "name": "@ctrip/cloud-component-trip-promo-headline",
      "type": "react.component",
      "version": "",
      "directive": {},
      "extension": { "parentId": "stru_page_tkairline", "sort": 500 },
      "props": {
        "htmlTag": "h2",
        "textColor": "#ffffff",
        "title": "{{FLIGHT_SECTION_TITLE}}"
      }
    },
    {
      "id": "stru_flight_europe",
      "label": "flight products",
      "name": "@ctrip/trip-sales-component-flight-products",
      "type": "react.component",
      "version": "",
      "directive": {},
      "extension": { "parentId": "stru_page_tkairline", "sort": 600 },
      "props": {
        "widgetLayout": "flightHotDeal",
        "merchandiseSetting": {
          "sourceType": "pool",
          "poolId": "{{POOL_ID_1}}"
        },
        "titleSubtitleMode": "default",
        "primaryTab": { "enable": true, "position": "center", "type": "text" },
        "title": { "enable": false, "text": "", "type": "text", "icon": { "enable": false } },
        "subtitle": { "enable": false, "text": "" },
        "defaultVisibleCardCount": { "mobile": 4, "online": 8 },
        "showCityImage": true,
        "hideAirline": false,
        "hideCheckedBaggageTag": false,
        "hideInstantDiscountTag": false,
        "hideAppOnlyDiscount": false
      }
    },
    {
      "id": "stru_headline_more_tk",
      "label": "more deals headline",
      "name": "@ctrip/cloud-component-trip-promo-headline",
      "type": "react.component",
      "version": "",
      "directive": {},
      "extension": { "parentId": "stru_page_tkairline", "sort": 1100 },
      "props": {
        "htmlTag": "h2",
        "textColor": "#ffffff",
        "title": "{{MORE_DEALS_TITLE}}"
      }
    },
    {
      "id": "stru_more_sales_tk",
      "label": "more-sales-banner",
      "name": "@ctrip/cloud-component-sales-more-sales-banner",
      "type": "react.component",
      "version": "",
      "directive": {},
      "extension": { "parentId": "stru_page_tkairline", "sort": 1200 },
      "props": {
        "excludePromoId": "",
        "priorityPromoIds": ""
      }
    },
    {
      "id": "stru_terms_tk",
      "label": "terms",
      "name": "@ctrip/cloud-component-promo-terms",
      "type": "react.component",
      "version": "",
      "directive": {},
      "extension": { "parentId": "stru_page_tkairline", "sort": 1300 },
      "props": {
        "fold": true,
        "terms": "{{TERMS_TEXT}}",
        "textColor": "#ffffff"
      }
    }
  ]
}
```

**模板变量说明：**

| 变量 | 来源 |
|------|------|
| `{{PAGE_BG_COLOR}}` | 用户提供，或默认 `#B50014`（TK 红） |
| `{{BANNER_PC_URL}}` / `{{BANNER_MOBILE_URL}}` | 用户提供，未提供则留空字符串 |
| `{{NAV_TAB_1_LABEL}}` | 导航栏 tab 文案，默认 "Flights" |
| `{{INTRO_HEADLINE}}` / `{{INTRO_TEXT}}` | 用户文案 |
| `{{FLIGHT_SECTION_TITLE}}` | 用户文案 |
| `{{POOL_ID_1}}` | 用户提供的 pool ID |
| `{{MORE_DEALS_TITLE}}` | 用户文案，默认 "View More Deals" |
| `{{TERMS_TEXT}}` | 用户提供，未提供则留空 |

**基于用户描述的调整规则（sort 700–1000 为可扩展区间）：**

- 需要分 Tab 展示（如欧洲/美洲） → 将 `stru_flight_europe` 替换为 `cloud-trip-common-tab` 包裹 N 个 `cloud-trip-common-tab-pane`，每个 pane 内放一个 `trip-sales-component-flight-products`
- 需要酒店楼层 → sort 700 插入 `cloud-component-sales5-hotel-products`
- 需要 ANT 楼层 → sort 800 插入 `trip-sales-component-ant-products`
- 需要图文介绍 → 在合适 sort 位置插入 `cloud-trip-common-photos-text`
- 用户不需要导航栏 → 删除 `stru_nav_tk`
- 用户不需要简介文字 → 删除 `stru_headline1_tk` 和 `stru_text_tk`

**每个 locale 单独调用 `create_page` 创建，参数：**

- `promoId`：用户提供的值
- `name`：对应 locale 的页面名称
- `locale`：对应语言代码
- `schemas`：填入变量后的完整 DSL 数组

API 报错或字段不确定时，**立即暂停并告知用户具体错误**，不继续其他 locale。

---

### Step 6 — 多语言文案检查

检查用户提供的文案是否覆盖所有 locale：

- **已覆盖**：全部创建完成，汇总返回所有 content ID
- **未覆盖**：暂停，询问用户：
  > 你提供的文案只有 [已有语言]，但页面还支持 [缺少语言]。是否需要将现有文案翻译成其他语言并更新对应页面？
  >
  > **注意：翻译内容为 AI 生成，需人工校对后再发布。**
  - 确认翻译 → 翻译后 `update_page` 更新对应 locale
  - 拒绝 → 告知用户哪些 locale 文案待补充

---

### Step 7 — 完成汇总

```
✅ 页面创建完成（未发布）

| Locale | 页面名称 | Content ID |
|--------|---------|------------|
| en-TH  | ...     | cont_xxx   |
| zh-HK  | ...     | cont_xxx   |

⚠️ 页面尚未发布，请在确认内容无误后手动发布。
```

---

## 错误处理原则

| 情况 | 处理方式 |
|------|---------|
| 必须信息缺失 | 追问，不继续执行 |
| 用户描述不清晰（locale / 分组等） | 追问澄清，不猜测 |
| API 报错 | 暂停，告知用户具体错误，等待指示 |
| 组件 props 不确定 | 查询 campaign-component 知识库，仍不确定则询问用户 |
| 翻译文案 | 明确告知为 AI 翻译，需人工校对 |

**所有页面默认不自动发布。**

---

## 依赖的 MCP 工具

- `campaign-page MCP`：`create_page`、`update_page`、`get_page_from_url`、`get_page_draft_data`、`publish_page`
- `campaign-component 知识库`：组件完整 props 说明，仅在模板变量不足以覆盖需求时查询
