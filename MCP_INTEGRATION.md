# MCP 集成说明

## 使用的 MCP Server

- **名称**：麦当劳中国 MCP Server（`mcd-mcp`）
- **接入地址**：`https://mcp.mcd.cn`
- **传输协议**：Streamable HTTP
- **认证方式**：请求头 `Authorization: Bearer <MCP Token>`
- **Token 申请**：https://open.mcd.cn/mcp ，手机号登录 → 控制台 → 激活
- **官方文档**：https://github.com/m-china/mcd-mcp-server

## 使用的 Tool 及调用流程

一次完整的「惊喜策划」调用链：

```
用户：下周五女朋友生日，预算100，整点有仪式感的
  │
  ├─ now-time-info → 确认今天日期，算出"下周五"是几号
  ├─ query-nearby-stores → 按用户位置找最近的门店（或问清配送地址→ delivery-query-stores）
  ├─ query-meals → 获取该门店可售菜单
  ├─ query-meal-detail → 确认候选套餐的组成
  ├─ query-my-coupons / query-store-coupons → 查可用优惠券
  ├─ calculate-price → 商品 + 券 + 配送费，算出应付价
  ├─ campaign-calendar → 看看近期有没有生日/节日相关活动可蹭
  ├─ （用户确认后）create-order → 下单，返回支付链接
  └─ 输出惊喜卡片（文案 + 餐品 + 价格 + 门店/订单信息）
```

### Tool 明细

| Tool | 在本项目中的作用 |
| ---- | ---------------- |
| `now-time-info` | 获取当前日期时间，推算"下周五""七夕"等相对日期，判断早餐/正餐/夜宵时段 |
| `query-nearby-stores` | 按用户位置查询附近门店，用于到店取餐场景 |
| `delivery-query-stores` | 按配送地址查询可配送门店，用于外送场景 |
| `query-meals` | 获取门店可售菜单（分类、餐品编码），是搭配餐品的数据基础 |
| `query-meal-detail` | 按餐品编码查详情、套餐组成和可替换项 |
| `query-my-coupons` | 查用户账户下可用优惠券 |
| `query-store-coupons` | 查当前门店可用优惠券 |
| `calculate-price` | 按商品列表 + 优惠券计算应付金额，"省了多少钱"是方案亮点之一 |
| `campaign-calendar` | 查当月营销活动日历，让惊喜方案蹭上节日热点 |
| `create-order` | 用户确认后创建订单 |
| `query-order` | 下单后可查订单状态 |
| `cancel-order` | 用户反悔时取消订单 |

## 业务价值

1. **把点餐变成情感消费**：传统点餐工具解决"吃什么"，本 Skill 解决"怎么制造惊喜"——文案 + 餐品 + 下单一气呵成，情绪价值拉满。
2. **省钱可视化**：自动叠券并展示"原价 → 券后价"，让用户直观感受到 AI 在帮他省钱。
3. **零学习成本**：用户不需要懂菜单、不需要比价，一句话需求直达结果。
4. **传播性**：惊喜卡片天然适合截图发朋友圈 / 发给朋友，是 Star 增长的自然动力。
