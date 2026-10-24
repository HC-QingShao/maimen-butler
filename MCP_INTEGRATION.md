# MCP_INTEGRATION.md — 麦当劳 MCP 集成说明

## 1. 使用的 MCP Server

| 项 | 值 |
|---|---|
| Server | 麦当劳中国官方 MCP Server（mcd-mcp v1.0.0，服务版本 v1.0.9） |
| Endpoint | https://mcp.mcd.cn |
| 传输协议 | Streamable HTTP（无状态，单次 JSON-RPC 即可完成调用，无需 initialize 握手） |
| 认证 | Authorization: Bearer <MCP_TOKEN>，Token 由 open.mcd.cn/mcp 控制台激活获得 |
| 限流 | 每 Token 600 次/分钟（超限 429）；Token 无效 401 |
| 服务范围 | 中国大陆地区（不含港澳台） |

接入配置（环境变量占位符，不含真实凭证）：

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer $MCD_MCP_TOKEN"
      }
    }
  }
}
```

本项目同时内置零依赖 Python 客户端 skill/scripts/mcd.py 作为备用通道：从环境变量 MCD_MCP_TOKEN 读取 Token，直接对 Endpoint 发起 tools/call JSON-RPC 请求，兼容返回纯 JSON 或 SSE 两种响应形态。

## 2. 使用的 Tool（共 33 个，全量接入）

### 点餐与订单

| Tool | 用途 | 在 Skill 中的角色 |
|---|---|---|
| query-meals | 门店菜单 | 点餐链路的选品入口，按分类/标签（早餐/新品/人气）筛选 |
| query-meal-detail | 餐品详情/套餐组成 | 确认套餐默认项，支撑换商品与特制需求 |
| query-store-coupons | 门店可用券 | 省钱军师：为当前门店匹配最优券 |
| calculate-price | 算价 | 下单前必经，输出小计/优惠/配送费/应付明细 |
| create-order | 下单 | 返回 payH5Url 与取餐信息（含取餐柜二维码） |
| cancel-order | 取消订单 | 按 cancelReasonCode（1改主意/2重复下单/3点错/4地址电话错/5时间错/-1其它）取消 |
| query-order | 订单详情/进度 | 下单后进度跟踪 |
| order-list | 历史订单 | "上次点了啥"场景（不含商城订单） |

### 地址与门店

| Tool | 用途 | 在 Skill 中的角色 |
|---|---|---|
| delivery-query-addresses | 配送地址列表 | 外送链路第一步 |
| delivery-create-address | 新增地址 | 用户无地址时补建（需用户确认） |
| delivery-query-stores | 可配送门店 | 外送场景选店 |
| query-nearby-stores | 附近门店 | 到店自取/得来速场景选店 |
| query-meal-assistance | 助餐服务（团餐） | 企业团餐算价前置查询 |

### 活动与券

| Tool | 用途 | 在 Skill 中的角色 |
|---|---|---|
| campaign-calendar | 活动日历 | "今天有什么活动"推荐场景 |
| available-coupons / auto-bind-coupons / query-my-coupons | 麦麦省领券三件套 | 薅羊毛链路：查询、一键领取、库存盘点 |
| query-lottery-info / draw-lottery / query-my-prizes | 积分抽奖三件套 | 先展示奖品池与消耗规则，用户同意后才抽奖 |

### 积分商城

| Tool | 用途 | 在 Skill 中的角色 |
|---|---|---|
| query-my-account | 积分账户 | 余额与即将过期积分提醒 |
| mall-points-products / mall-product-detail | 商城商品 | 积分兑换选品 |
| mall-create-order | 兑换下单 | 扣积分发券码/实物，强制用户确认 |
| mall-order-list / mall-order-detail | 商城订单 | 兑换记录查询 |

### 主题活动与通用

| Tool | 用途 | 在 Skill 中的角色 |
|---|---|---|
| query-party-city / query-party-store / query-partystore-date / query-partystore-session / party-order-create | 主题活动五级漏斗 | 生日派对/品鉴会/麦麦体验营的 city→store→date→session→order 链路 |
| now-time-info | 当前时间 | 判断早餐时段（10:30 前）、解析"今天/明天"预约日期 |
| list-nutrition-foods | 营养数据（TOON 紧凑格式） | 解析能量/蛋白质/脂肪/碳水/钠/钙，按热量预算配餐 |

## 3. 核心调用流程

### 流程 A：外送点餐（含省钱比价）

```
now-time-info（定日期）
  → delivery-query-addresses（选地址，无地址则 delivery-create-address）
  → delivery-query-stores(beType=2)（选门店，得 storeCode/beCode）
  → query-meals（菜单选品）+ query-meal-detail（套餐/特制）
  → query-store-coupons（匹配门店可用券，给出最省组合）
  → calculate-price（报小计/优惠/配送费/应付，用户确认）
  → create-order（返回 payH5Url + 订单号）
  → query-order（进度）/ cancel-order（反悔时按原因码取消）
```

### 流程 B：营养配餐

```
list-nutrition-foods（TOON 表格解析）
  → 按用户热量预算过滤组合 → 输出推荐 + 营养明细（附"仅供参考"提示）
```

### 流程 C：薅羊毛

```
auto-bind-coupons（一键领券）→ query-my-coupons（库存）
  → query-lottery-info（抽奖划算度评估，用户同意后 draw-lottery，再 query-my-prizes）
  → query-my-account（提醒即将过期积分）
```

### 流程 D：主题活动预约

```
campaign-calendar / 用户指定（得 spuId）
  → query-party-city → query-party-store → query-partystore-date
  → query-partystore-session（用户确认场次）→ party-order-create
```

## 4. 业务价值

1. 消灭点餐摩擦：把"打开 App→选地址→翻菜单→凑券→比价→下单"的 6 步压缩成一句话，且 AI 主动匹配优惠券，平均每单可省数元。
2. 让沉睡资产苏醒：大量用户的积分、待领券、抽奖次数处于闲置状态，Skill 主动盘点并提醒，提高会员权益利用率。
3. 健康决策辅助：官方营养数据 + 热量预算组合推荐，服务健身与控盐人群。
4. 线下场景数字化：主题活动预约链路（派对/品鉴会）此前只能到店咨询，现在对话即可完成五级筛选与下单。
5. 安全可控：所有涉及金钱与积分的写操作均强制人工确认，支付永远由用户本人完成，符合账号安全最佳实践。

## 5. 合规说明

- 项目不含任何真实 Token/密钥/账号凭证，配置仅使用环境变量占位符；
- 仅调用麦当劳官方 MCP Server 的公开能力，无逆向、无爬虫、无绕过限流；
- 输出信息（价格、营养、活动）以麦当劳官方渠道实时结果为准。
