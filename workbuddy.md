# WorkBuddy 开发记录

> 本文件记录使用 WorkBuddy 开发「麦麦惊喜策划师」Skill 的完整过程。
> 参加 WorkBuddy 专项奖励（3000 积分）时必须提交本文件。

## 开发环境

- 开发日期：2026-10-12
- 技能名称：mcd-surprise-planner（麦麦惊喜策划师）
- MCP 连接器：mcd-mcp（Token 通过 open.mcd.cn 手机登录激活后，在 WorkBuddy→→中接入并启用）

## 对话记录

### 1. SKILL.md 文档审查与重写

用户：我想做一个麦当劳惊喜策划 Skill，帮我看看 SKILL.md 有没有要改的

WorkBuddy 重写了 SKILL.md，核心改动：

**结构层（P0，决定能不能被识别）**
- 补上 YAML frontmatter：`name: mcd-surprise-planner`、完整 description（含「做什么 + 何时用」）、`agent_created: true`
- 把原来正文里的「名称 / 一句话描述 / 触发条件」三节合并进 frontmatter，正文只留流程
- 指出 Skill 必须安装在合规目录（用户级 `~/.workbuddy/skills/` 或项目级 `.workbuddy/skills/`），放在项目根目录不会被加载

**内容层（P1，影响实际效果）**
- 第 1 步加了字段提取表，明确「哪些能默认、哪些必须追问」（默认不追问，避免一次抛一堆问题）
- 新增异常与降级章节：门店未营业/不配送、菜单价格拿不到、券失效、用户改主意，各有明确处理，堵住「现场编造」
- 补齐过敏原与饮食禁忌提醒、预算超支必须摊开说、不得静默超支
- 工具表从 9 个扩到覆盖 mcd-mcp 全套：`auto-bind-coupons`（自动领券）、`query-promotions`、`delivery-query-addresses` / `delivery-create-address`、`list-nutrition-foods`（成分核对）、`query-my-account`、积分商城系列、`draw-lottery`（互动彩蛋）、`party-order-create`（团建包场）
- 输出卡片模板补了「人均 / 原价→券后价 / 履约方式」，利于用户直接决策
- 文风统一改为祈使句，符合规范

### 2. Skill 打包安装

用户：打包安装到.workbuddy\skills 目录

WorkBuddy 按官方 Skill 流程完成安装和打包：
1. 用 `init_skill.py` 在项目级目录初始化 skill 骨架（名称校验为 hyphen-case：mcd-surprise-planner）
2. 写入正式 SKILL.md（含 frontmatter：name / description / agent_created: true）
3. 删除生成的示例脚手架（`scripts/example.py`、`references/api_reference.md`、`assets/example_asset.txt` 及空目录）——本 skill 不需要捆绑资源
4. `quick_validate.py` 校验 → Skill is valid!
5. `package_skill.py` 打包 → 校验通过并生成 zip（约 3.1 KB）

产物：已安装 Skill（项目级 `.workbuddy/skills/mcd-surprise-planner/SKILL.md`）+ 打包产物 `dist/mcd-surprise-planner.zip`。Skill 需要重启会话（或重新进入项目）后才会被识别加载。

### 3. 场景测试一：生日惊喜（预算 ¥100，默认 2 人）

用户：模拟一下：女朋友这周天生日，预算100，跑一遍完整流程

WorkBuddy 执行了真实 MCP 调用的完整流程，止步在下单确认前（按 SKILL 边界，未确认不下单）：

- **需求理解**：场合生日｜对象女朋友｜预算 ¥100｜人数默认 2 人；「这周天」换算为 2026-10-18（周日）；履约方式未说，按「到店自取/外带」跑（示例假设）
- **账户与券**：自动领取 3 张券（免费脆薯饼、人气麦旋风买一送一、9.9 元中杯冰美式）
- **门店**：麦当劳上海黄浦华旭国际大厦餐厅（storeCode 1450713，西藏中路 336 号，距示例点 144m，24h 营业）
- **当天活动**：10-18 是韩式蘸酱上新，「鸡薯双全盒 15.9 元起」——直接选进方案
- **试算**：`calculate-price` → ¥95.8，预算内
- **惊喜卡片**：生成了祝福文案（生日快乐！今天你是唯一的主角……）+ 餐品搭配（龙焰芝士棒鸡腿堡三件套、板烧鸡腿堡三件套、"我就喜欢"鸡薯双全盒、经典麦旋风）+ 门店取餐信息，等待用户确认是否下单

### 4. 场景测试二：团建凑单（6 人，人均 ¥50）

用户：团建6个人，人均50，试试凑单逻辑

WorkBuddy 跑完凑单全流程，同样止步在下单确认前：

- **凑单结果**：原价 ¥306.0 → 券后 ¥299.9（省 ¥6.1），实际人均 ¥49.98，差 2 分钱卡满预算
- **凑单四步法**（可复用）：①打底：主餐按人数配，人手一份（6 × 三件套 ≈ ¥161）②抬高：加分享装把客单从 161 拉到 250+（薄皮V翅双拼十翅 + BFF四宫格小食盘）③用券：全场只叠一张最优券（王牌小食拼省 ¥6.1）④收尾：用小食/甜品补差价，卡到预算线（派 ×2 → ¥299.9）
- **另一条路**：6 人团建其实更适合走企业团餐（beType=6），那边有满减/满折（`query-promotions`），凑单逻辑会变成「凑门槛」而不是「凑尾数」

## 遇到的问题与解决

两轮真实模拟共挖出 3 条 SKILL 需要补的规则：

| # | 现象 | 结论 |
|---|------|------|
| 1 | 薯条买一送一、王牌小食拼的券只到 10-16（周一~周五），周日压根用不上；周日唯一有效的是「9.9 元中杯冰美式」 | 券有时效，方案必须按**实际日期**过滤可用券 |
| 2 | 麦旋风买一送一在 `calculate-price` 报「当前优惠券不可用」 | 已领取 ≠ 当前门店可用，必须用 `query-store-coupons` 复核 |
| 3 | 第一版想同时叠两张券，`calculate-price` 报错 `code 600022 — 暂不支持多张券使用` | 一单只能用一张券，凑单时先定主餐总量，再选一张最优券补位 |

## 备注

- 本文件中不包含真实 MCP Token、手机号等隐私信息
- 对话记录来自 WorkBuddy 分享页（2026-10-12 导出），本地路径已做脱敏处理
