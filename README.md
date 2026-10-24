# 麦门管家 MaiMen Butler

> 一句话吃透麦当劳 —— 基于麦当劳中国官方 MCP 的全能点餐 Skill
> 覆盖官方 MCP Server 全部 33 个工具：智能点餐 · 省钱军师 · 营养配餐 · 积分商城 · 抽奖 · 主题活动 · 订单管理

## 项目介绍

「麦门管家」是一个运行在 WorkBuddy 上的 Skill（技能包）。它把麦当劳中国官方 MCP 服务的 33 个工具编排成 11 条自然语言场景链路，让用户说人话就能完成从"我饿了"到"取餐码在手"的全过程：

| 场景 | 说一句话就够了 | 幕后编排 |
|---|---|---|
| 外送点餐 | "帮我点个麦辣鸡腿堡套餐送家里" | 选地址→选门店→菜单→算价→下单 |
| 到店/得来速 | "附近哪家麦当劳能买甜筒，我开车去" | 找店→菜单→算价→下单 |
| 省钱军师 | "用券给我配一单最便宜的" | 领券→门店可用券→算价比价 |
| 营养配餐 | "减脂期能吃什么，600 大卡以内帮我搭" | 营养表解析→热量筛选→组合推荐 |
| 薅羊毛 | "把能领的券都领了，积分别浪费" | 一键领券→查库存→提醒过期积分 |
| 积分抽奖 | "帮我抽一次奖，先看看划不划算" | 活动规则→消耗确认→抽奖→中奖播报 |
| 积分商城 | "我的积分能换啥" | 余额→兑换列表→详情→下单 |
| 主题活动 | "给孩子办个麦当劳生日派对" | 城市→门店→日期→场次→下单 |
| 订单管理 | "上次点了啥？" / "点错了帮我取消" | 历史/商城订单查询，按原因码取消 |

**设计亮点**

- 安全铁律：下单前强制展示商品清单与总价（含优惠明细）并等用户确认；payH5Url 支付链接只转达、绝不代付；扣积分的操作先讲规则再动手。
- 双通道调用：既可走 WorkBuddy 原生 MCP 连接器（mcd-mcp），也可用内置的零依赖 Python 客户端直连（无状态 JSON-RPC，无需握手），环境适配性拉满。
- 渐进式知识：SKILL.md 只装场景路由，33 个工具的参数表和坑位说明按需加载，省上下文。
- 真实数据实测：菜单、算价、积分账户等链路均用真实 Token 验证通过。

## 目标用户

- 打工人：中午 12 点纠结吃什么、想省钱又想吃好的外卖人群
- 健身党/家长：关心热量、钠含量，需要营养配餐建议的人群
- 麦门铁粉：积分多、爱抽奖、追活动、攒周边的资深会员
- 家长：想给孩子办生日派对、品鉴会、麦麦体验营的家庭

## 安装方法

### 0. 申请麦当劳 MCP Token（免费）

1. 访问 open.mcd.cn/mcp（https://open.mcd.cn/mcp），右上角【登录】（手机号验证码）
2. 登录后右上角【控制台】→【激活】→ 同意服务协议
3. 复制生成的 MCP Token

### 1. 配置 MCP 连接器（推荐，以 WorkBuddy 为例）

WorkBuddy：【专家·技能·连接器】→【连接器】→ 右上角【自定义连接器】→【配置MCP】，粘贴（替换 Token 后保存并启用）：

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer YOUR_MCP_TOKEN"
      }
    }
  }
}
```

Cursor / Cherry Studio / TRAE / Kiro / VSCode 均可按各自 MCP 配置方式接入同一段 JSON。

### 2. 安装 Skill

把本仓库 skill/ 目录复制到 WorkBuddy 用户技能目录：

```
~/.workbuddy/skills/maimen-butler/   # 即 C:\Users\<你>\.workbuddy\skills\maimen-butler\
```

### 3. 配置 Token（两种方式任选）

- 环境变量（推荐）：设置 MCD_MCP_TOKEN=<你的token>
- 本地文件：在技能根目录创建 token.txt 写入 Token（已被 .gitignore 排除，切勿提交到公开仓库）

### 4. 验证

```bash
python skill/scripts/mcd.py list
python skill/scripts/mcd.py call now-time-info
python skill/scripts/mcd.py call query-my-account '{}'
```

## 使用示例

安装后在 WorkBuddy 对话框里直接说：

```text
我：今天中午吃麦当劳，帮我用最优惠的方式点一个巨无霸套餐，外送到公司
AI：（选择地址→门店→菜单→匹配优惠券→算价）
    为你找到最优方案：巨无霸套餐 + 中可乐，商品小计 43.5 元，
    使用"麦麦省"券立减 8 元，配送费 6 元，应付 41.5 元。
    确认下单吗？
我：确认
AI：下单成功！订单号 MCD20261024xxxx，预计 12:45 前送达。
    支付链接：https://pay.mcd.cn/... （请在 15 分钟内完成支付）

我：我 875 积分快过期了，能换点啥？
AI：（查积分→列兑换清单）当月无过期积分；当前可兑换：板烧鸡腿堡券（xx积分）…
```

更多场景见 skill/SKILL.md 的「场景路由」。

## 项目结构

```
maimen-butler/
├── README.md               # 本文件
├── CONTEST_DECLARATION.md  # 参赛声明（官方原文，未改动）
├── MCP_INTEGRATION.md      # 麦当劳 MCP 集成说明
├── workbuddy.md            # WorkBuddy 开发记录
├── .gitignore              # 排除 token.txt 等敏感文件
└── skill/                  # WorkBuddy Skill 源码（可直接安装）
    ├── SKILL.md            # 技能主文件：场景路由 + 安全铁律
    ├── references/tools.md # 33 个工具参数速查 + 关键链路 + 坑位清单
    └── scripts/mcd.py      # 零依赖 JSON-RPC 客户端（备用调用通道）
```

## 免责声明

本项目为麦当劳程序员节创意开发大赛参赛作品，由参赛者独立开发，非麦当劳官方产品。项目输出仅供参考，不构成医疗、营养或其他专业建议；餐品信息、价格及供应状态以麦当劳官方渠道的实时结果为准。

## 参赛信息

- 赛事：麦当劳程序员创意开发大赛（https://github.com/M-China/mcd-developer-innovation-challenge）
- 基于：麦当劳中国官方 MCP 服务（https://open.mcd.cn/mcp ，v1.0.9，33 Tools）
- 开发工具：WorkBuddy（官方合作伙伴）
- 许可：仅供学习交流与非商业用途
