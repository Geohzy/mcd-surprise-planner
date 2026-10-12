# 麦麦惊喜策划师 · McDonald's Surprise Planner

> 一句话，把麦当劳变成你的惊喜快递员。

麦麦惊喜策划师是一个基于麦当劳中国 MCP 能力的 WorkBuddy Skill。
你只需要用一句话描述**场合**（生日 / 表白 / 道歉 / 毕业 / 节日…）、**对象**和**预算**，
它就会自动完成：写祝福文案 → 搭配合适的餐品 → 算价叠券 → 一键下单，
最后生成一张可以直接发给对方的「惊喜卡片」。

## 效果示例

你说：

> 下周五是我女朋友生日，预算 100 块，整点有仪式感的

它会返回：

```
🎂 生日惊喜方案 · for 她

💌 祝福卡片
「愿你的新一岁，像麦旋风一样甜，像薯条一样被偏爱。
生日快乐，我的女孩！」

🍔 推荐搭配（2 人份 · 预估 ¥96.5）
- 双层深夜小龙虾堡 ×2
- 脆薯条（大）×1
- 麦旋风（奥利奥）×2
- 已叠加优惠券：满 99 减 15

📍 附近门店：XX 路餐厅（距你 800m，营业至 23:00）
🛒 确认后一键下单 → [去支付]
```

## 快速开始

### 1. 申请麦当劳 MCP Token

1. 打开 https://open.mcd.cn/mcp ，点击右上角
2. 使用手机号验证登录，登录成功后右上角变为
3. 点击申请 MCP Token，一键复制保存

### 2. 在 WorkBuddy 中配置

打开 WorkBuddy → 左侧边栏→→
右上角→，填入：

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

记得把 `YOUR_MCP_TOKEN` 换成你自己的 Token，点击保存后启用 `mcd-mcp`。

### 3. 导入本 Skill

把本仓库的 `SKILL.md` 导入 WorkBuddy（或放在 Skill 目录下），然后直接对话：

> 帮我策划一个表白惊喜，预算 80，送到 XX 小区

## 支持的场合

| 场合 | 示例说法 |
| ---- | -------- |
| 生日 | 女朋友下周五生日，预算 100 |
| 表白 | 想跟喜欢的人表白，预算 80，要浪漫一点 |
| 道歉 | 惹女朋友生气了，买点好吃的哄哄她 |
| 毕业 | 室友毕业了，宿舍 4 个人庆祝一下 |
| 节日 | 七夕 / 圣诞 / 1024 程序员节整点有意思的 |
| 日常 | 加班到深夜，给自己点顿好的 |

没列出来的场合直接说，策划师会自己理解。

## 工作原理

```
一句话需求
↓
① 理解场合、对象、预算、人数
↓
② query-meals / query-meal-detail 浏览菜单
↓
③ 生成个性化祝福文案（按场合调性）
↓
④ calculate-price 算价 + query-my-coupons / query-store-coupons 叠券
↓
⑤ query-nearby-stores 找最近的门店
↓
⑥ 用户确认 → create-order 下单
↓
一张惊喜卡片（文案 + 餐品 + 价格 + 取餐/配送信息）
```

详细调用链见 [MCP_INTEGRATION.md](./MCP_INTEGRATION.md)。

## 目标用户

- 想制造小惊喜但没时间的打工人
- 不会写祝福语的直男 / 直女
- 宿舍、办公室团建点餐的"点单担当"

## 文件说明

| 文件 | 说明 |
| ---- | ---- |
| `SKILL.md` | Skill 定义，可直接导入 WorkBuddy |
| `MCP_INTEGRATION.md` | MCP 调用说明 |
| `mcp-config.example.json` | MCP 配置示例（占位符） |
| `workbuddy.md` | WorkBuddy 开发记录 |
| `CONTEST_DECLARATION.md` | 参赛声明（官方原文件，请勿修改） |

## 参赛信息

本项目为「麦当劳 × WorkBuddy 1024 程序员节创意开发大赛」参赛作品，
由参赛者独立开发，非麦当劳官方产品。
