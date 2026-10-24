# workbuddy.md — WorkBuddy 开发记录

本项目全程使用 WorkBuddy（麦当劳大赛官方合作伙伴）完成创意构思、开发、联调与打包。以下为本项目的开发过程记录（对话上下文摘要，可按需导出完整对话用于核验）。

## 开发环境

- 工具：WorkBuddy（AI Agent 工作台，Agent 模式）
- 接入：通过自定义连接器 + 内置 Python 客户端双通道访问麦当劳官方 MCP Server（https://mcp.mcd.cn）

## 开发时间线

### 1. 需求理解与文档研读

- 抓取并解析麦当劳 MCP 官方文档（open.mcd.cn/mcp）与官方 README（v1.0.9 全量 33 工具表）；
- 梳理工具分组（点餐/地址门店/活动券/积分商城/主题活动/通用）与 beType、orderType 等枚举语义；
- 提炼关键约束：600 次/分钟限流、TOON 紧凑响应格式、外送备注不超 50 字、抽奖 1 秒冷却等。

### 2. 连通性验证（真实 Token 实测）

- 使用 Streamable HTTP 直接发起 JSON-RPC initialize：返回 serverInfo: mcd-mcp v1.0.0；
- 验证无效 Token 返回 401，有效 Token 正常；
- 实测 now-time-info、query-my-account 等工具，确认无状态调用可行（无需维护会话）；
- tools/list 实际返回工具数多于文档，确认以运行时列表为权威。

### 3. Skill 架构设计与实现（WorkBuddy skill-creator 工作流）

- 采用渐进式披露结构：SKILL.md（场景路由+安全铁律，常驻轻量）→ references/tools.md（33 工具参数速查+链路+坑位，按需加载）→ scripts/mcd.py（零依赖 JSON-RPC 客户端，可执行而无需载入上下文）；
- 设计 11 条自然语言场景路由（外送/自取/得来速、省钱军师、营养配餐、薅羊毛、抽奖、积分商城、主题活动、订单管理、取消订单）；
- 制定安全铁律：下单/兑换强制用户确认、绝不代付（payH5Url 只转达）、扣积分先讲规则。

### 4. 调试与修复

- 发现并修复 token 文件路径解析缺陷（脚本子目录与技能根目录不一致）；
- 端到端回归：mcd.py list（全工具枚举）与 mcd.py call query-my-account（真实账户数据：可用积分 875.4）均通过。

### 5. 参赛化改造

- 依据大赛报名要求补齐 README.md、CONTEST_DECLARATION.md（官方原文）、MCP_INTEGRATION.md、workbuddy.md；
- 安全整改：从公开仓库中移除真实 Token，改用 MCD_MCP_TOKEN 环境变量占位符，并通过 .gitignore 排除 token.txt；
- 使用 WorkBuddy 的打包脚本对 Skill 完成校验（validate）与 zip 打包。

## 结论

本项目从文档研读到成品打包的全部编码、调试、文档工作均在 WorkBuddy 对话中完成，符合 WorkBuddy 联动奖励的核验要求。
