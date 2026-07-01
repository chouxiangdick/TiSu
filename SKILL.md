---
name: personal-coach-skill-design
description: 设计/升级"长期陪伴型"个人 coach 类 skill 的完整方法论——健身/学习/营养/写作教练。把 v1 风格（被动答 + 机械出 plan）升级到 v2 风格（主动跟踪 + 状态关注 + 周期化 + 抗幻觉）。当用户说"升级我的 X skill"、"我的 skill 老是出幻觉/不主动"、"X 教练 skill 加主动跟踪"时用。**核心 4 大支柱 + 4 个 cron 协同 + 4 条抗幻觉硬规则 + cron-sync-checklist** 防 cross-file drift。
version: 1.1.0
author: Community (auto-generated from fitness-coach v1→v2 实战)
license: MIT
metadata:
  hermes:
    tags: [skill-design, coach, proactive, anti-hallucination, cron, state-tracking, mesocycle, cron-sync, fitness, learning, nutrition]
    related_skills: [hermes-agent-skill-authoring, hermes-deployment-guide, gateway-zombie-recovery]
---

# Personal Coach Skill Design（长期陪伴型 skill 设计方法论）

把"用户问 → LLM 答"的工具型 skill 升级为"主动跟踪 + 状态关注 + 周期化 + 抗幻觉"的私教型 skill。**所有方法来自 fitness-coach v1 → v2 实战**（社区案例，可复用方法论）。

## 适用场景

| 场景 | 描述 |
|------|------|
| **健身/营养教练** | 训练计划 + 饮食 + 体重/体脂跟踪 + 周期化 |
| **学习/考试教练** | 学习计划 + 进度跟踪 + 知识点测试 + 周期化复习 |
| **写作/创作教练** | 写作计划 + 字数跟踪 + 灵感/状态 + 周期化产出 |
| **投资/理财教练** | 资产跟踪 + 风险评估 + 周期化复盘 |
| **冥想/心理教练** | 练习计划 + 心情跟踪 + 周期化调整 |
| **习惯养成教练** | 习惯跟踪 + 周期化坚持 + 抗中断机制 |

**任何需要"长期陪伴 + 数据跟踪 + 主动调整 + 状态关注"的 skill 都适用**。

## 不适用场景

- 一次性任务型 skill（"翻译"、"总结"、"查资料"）—— 不需要主动跟踪
- 工具型 skill（"天气查询"、"日历"）—— 不需要状态关注
- 短期项目 skill（"3 天内完成 X"）—— 不需要周期化
- 没有 state.md 数据存储 —— 必须有
- 只用 CLI 不接 cron —— v2 必须接 cron

## 首次加载行为（v2 强制的 Step 0）

**当这个 skill 被第一次加载到 Hermes Agent 时**，Agent 必须在执行任何 cron 或回答任何训练问题之前，主动采集用户画像：

```
🏋️ 欢迎使用 [X] 教练！

先让我了解你，以便制定个性化方案：

性别：？（男 / 女）
年龄：？
身高（cm）：？
体重（kg）：？
训练经验：？（新手 / 初中级 / 中高级 / 高级）
当前目标：？（增肌 / 减脂 / 增力 / 保持）
每周能练几天：？
有无伤病或限制：？
```

**规则**：
- 只问一次。用户回答后存入 `state.md` 用户画像区
- 用户事后可以随时说"更新我的数据"来修改
- 用户画像区是 **所有 cron 和算法的前置依赖** — 没有用户画像就不该出训练计划
- 如果用户跳过（"先不填"），Agent 应说明"没有基础数据我无法制定安全有效的计划"，暂不启用 cron，等用户填完

## 4 大支柱（**v2 核心**）

v1 → v2 的本质差异 = 4 大支柱的落实程度。

### 支柱 1：主动跟踪（Proactive Tracking）

**v1 风格**：
```
用户: "我今天练什么？"
LLM:  [读 skill] → [出训练计划] → [回]
```

**v2 风格**：
```
[cron 9:30 触发]
↓
LLM: [读 state.md] → [检查今日字段] → [问 6 问]
用户: "79.5 | 7h | 5/10 | 8/10 | 6/10 | 无"
↓
LLM: [patch state.md] → [python 算 EWMA + Recovery] → [按状态出 plan]
```

**关键设计**：
- 4 个 cron 角色分工（详见下方）
- state.md 单一权威数据源
- 所有"主观报告" = 信号，"数据" = 真相
- 没数据就明说"无数据"，**不编**

### 支柱 2：状态关注（State Monitoring）

**v1 风格**：
- 按 cycle_index 机械出 plan
- 不看用户状态

**v2 风格**：
- Recovery Score 决定加码/减量/休息
- 4 维度（生理/心理/社会/训练）综合判断
- 心理信号（心情 ≤ 3）优先于训练信号
- 社会压力（加班/应酬）灵活调整

**关键设计**：
- Recovery Score 算法（详见 references/algorithm-templates.md §R1）
- 状态评估算法（生理/心理/社会/训练 4 维度）
- "先看人，再看 plan"（原则 5：状态 > 计划）

### 支柱 3：周期化编程（Mesocycle Programming）

**v1 风格**：
- 按周循环（Push/Pull/Legs 简单循环）
- 无容量渐进
- 无 deload 计划

**v2 风格**：
- 4 周 hypertrophy + 1 周 deload（mesocycle）
- 容量渐进：W1 baseline → W2 +1组 → W3 +2组 → W4 维持 → W5 deload
- 4 个 deload 自动触发条件（计划/力量/状态/疲劳）
- 减脂期 vs 增肌期切换判断

**关键设计**：
- 周期结构（详见 references/algorithm-templates.md §R3）
- deload 触发条件
- mesocycle 进度 date math

### 支柱 4：抗幻觉保障（Anti-Hallucination Hard Rules）

**v1 风格**：
- LLM 编合理解释（如"健腹轮跪姿要股四头肌"）
- 引用编造的论文（如"RP 2019 paper"）
- 心算复杂数字
- 编时间线（"再过 X 周你就能到 Y kg"）

**v2 风格**：
- **4 条硬规则**（v2 强制，不可破）

详见 §抗幻觉硬规则。

## 4 个 cron 角色分工

**核心思想**：不同频率采集不同粒度数据，分离关注点。

| cron | 频率 | 角色 | 输入 | 输出 |
|------|------|------|------|------|
| `morning-X-plan` | 每天 9:30 | **日问 + 日 plan** | state.md 今日字段 | N 问 + 当日 plan |
| `evening-X-reminder` | 每天 18:30 | **日复盘 + 决策** | 训练后 RPE/不适 | autoregulation 决策 + 重量 patch |
| `weekly-X-checkin` 🆕 | 每周一 9:30 | **周复盘 + 趋势** | 最近 7 天数据 | 周报 + 力量 trend + 下周建议 |
| `monthly-X-report` 🆕 | 每月 1 号 9:30 | **月报 + 战略** | 过去 30 天数据 | 月报 + mesocycle 进度 + 下月建议 |

**设计要点**：
- 4 个 cron **都必读 state.md**
- 4 个 cron **第一步都做"今日字段过期检查"**（v2.1 必查）
- 4 个 cron **共享同一套抗幻觉硬规则**
- 4 个 cron **触发后都 patch state.md**

## 4 条抗幻觉硬规则（**v2 强制**）

**这是 v2 最重要的部分**。每条都对应实战事故教训。

### 规则 1：用户报告异常时

- ❌ 听到"X 不对"立刻给合理解释 → **先 read_file 查源**（state.md / cron prompt / 当前 SKILL.md）再答
- ❌ 编"健腹轮跪姿要股四头肌"这种合理但没验证的解释
- ✅ 查不到就说"我去查一下"，然后 read_file/grep
- ✅ 如果是 cross-file drift（如 cron prompt 跟 SKILL.md 不一致），**先修源文件**，再答用户
- ✅ 修完要明确告诉用户"找到 root cause 了"

### 规则 2：引用数据时

- ❌ 引用没验证的论文 / URL → 写"共识知识"就行，别编具体 paper 名
- ❌ 编"研究显示 X 的人比 Y 的人 Z%"这种无来源数字
- ✅ 数字/Paper/URL 只引用 references 里有的，没记录就说"没记录，建议查 [具体来源]"
- ✅ 引用**体系名**（RP / Mike Israetel 共识）可以，但编"RP 2019 paper" 这种具体引用 = 幻觉

### 规则 3：计算数字时

- ❌ 心算复杂数字（EWMA / 1RM / 比例）
- ✅ 用工具算（python 表达式 / `python3 -c "..."`）
- ✅ 显示计算过程给用户看

### 规则 4：推断未来时

- ❌ "再过 X 周你就能到 Y kg"（编时间线）
- ✅ "按当前 EWMA -0.5 kg/周 推算，约 X-Y 周，**前提是坚持当前节奏**"
- ✅ 给范围，不给具体日期

## 🧠 心理建设机制（鼓励系统）

**健身教练不只是算数据和出计划**。在用户不想动的时候推一把，在用户累的时候给空间。金句挂钩实际表现，不尬吹不廉价。

### 触发场景 + Agent 行为

| 场景 | 检测条件 | Agent 说的 | 意图 |
|------|---------|-----------|------|
| 连续训练 3 天+ | streak ≥ 3 | 🔥 **勤奋者他日不留悔恨** — 今天状态一般？减量但别停 | 鼓励坚持 |
| 偷懒 1 天 | 昨日无记录，非休息日 | 💪 一天不练影响不大，**但别让明天再写悲歌** | 防滑坡 |
| PR 破纪录 | 检测到 PR 标记 | 🏆 **纪录就是用来破的**。下次目标 +2.5kg | 趁热打铁 |
| 减脂平台期 | 体重连续 7 天不变 | 📉 体重没动？围度呢？照片呢？**数据不只看一个维度** | 打破焦虑 |
| 受伤/生病休息 | 用户报告伤病 | 🛌 **休息也是训练的一部分**。恢复好了再冲 | 给许可 |
| 连续偷懒 3 天+ | streak 中断 ≥ 3 天 | ⚠️ **勤奋者他日不留悔恨，懒惰者明日再写悲歌** | 敲警钟 |

### 设计规则

1. **不廉价** — 每句金句跟实际表现挂钩，不天天说"你是最棒的"
2. **不尬吹** — PR 才庆祝，连续训练才表扬，空话不说
3. **分场景** — 鼓励（连续训练）/ 敲打（连续偷懒）/ 安慰（受伤休息）三类分开
4. **可配置** — 金句放在 references/quotes.md，用户可自定义

## state.md 数据 schema 模板

**所有 cron 必读的单一权威数据源**。schema 必须包含：

```yaml
# 用户画像（首次加载时采集，之后只更新）
## 基本信息
- 性别: 男/女
- 年龄: N
- 身高: N cm
- 当前体重: N kg（每日更新）
- 训练经验: 新手 / 初中级 / 中高级 / 高级
- 当前目标: 增肌 / 减脂 / 增力 / 保持
- 每周训练天数: N
- 伤病/限制: 无 / 具体描述
- 目标备注: 自由文本

# 状态（cron 启动时必读）

## 当前状态（updated YYYY-MM-DD HH:MM）
- 上次 X：YYYY-MM-DD（X 类型）
- 连续 Y 天数：N 天
- 今日 (YYYY-MM-DD)：**X 类型**（如 Push / 休息日 / 写作日）

## 每日指标（cron 主动问 → 用户答 → 自动 patch）
- YYYY-MM-DD HH:MM: 指标 1 | 指标 2 | 指标 3 | ...

## 周维度指标（每周一 patch）
- YYYY-MM-DD HH:MM: 指标 1 | 指标 2 | 指标 3 | 指标 4

## EWMA / 趋势字段（cron 自动算）
- 7-day: X
- 14-day: X
- 30-day: X

## X 历史（最后 5 次）
- X: [a, b, c, d, e]

## 周期状态
- 当前 mesocycle: N (类型)
- 当前 week: WX
- 开始日期: YYYY-MM-DD
- 预计 deload: YYYY-MM-DD
```

**关键设计**：
- 今日字段**必须带日期**（cron 才能检查是否过期）
- 每次 patch 必更新"当前状态 (updated 时间戳)"
- 所有数字字段都用 python 算，不心算

## 抗 cross-file drift：cron-sync-checklist

**v2.1 实战发现**：SKILL.md Recovery 公式 40/30/30，cron prompt 用 33/33/34——**两个文件不一致**。LLM 主动检测到 + 提议修。

**防 drift 流程**（详见 references/cron-sync-checklist.md）：

```
1. 改 SKILL.md → 列出影响的 cron
2. 对每个 cron `hermes cron edit` 同步 prompt
3. 跑 `hermes cron run <id>` 测试
4. 对比 SKILL.md 和 cron prompt 的关键段落
5. 不一致 → 修 + 测
```

**v2 强制：每次自进化流程必须包含 Step 7 = cron-sync-check**。

## 完整设计 checklist（升级现有 skill 时跑）

```markdown
## Personal Coach Skill Design · Checklist

### 1. 数据 schema
- [ ] state.md 有 **用户画像** 区（性别/年龄/身高/体重/经验/目标）
- [ ] state.md 有"今日 (YYYY-MM-DD)"字段（带日期）
- [ ] 每日/每周/每月数据 sections
- [ ] EWMA / 趋势字段（cron 自动算）

### 2. 4 个 cron
- [ ] morning-X-plan（每天 9:30，N 问 + 当日 plan）
- [ ] evening-X-reminder（每天 18:30，autoregulation 决策）
- [ ] weekly-X-checkin（每周一 9:30，周报）
- [ ] monthly-X-report（每月 1 号 9:30，月报）

### 3. 4 条抗幻觉硬规则
- [ ] 用户报告异常时先 read_file
- [ ] 引用只引 references 里的
- [ ] 计算用 python 工具
- [ ] 推断给范围不给日期

### 4. 抗 cross-file drift
- [ ] SKILL.md 改了 → 4 个 cron prompt 同步
- [ ] cron prompt 改了 → SKILL.md 同步
- [ ] 跑 cron run 测试

### 5. 测试
- [ ] 模拟用户报告异常 → LLM 先 read_file 不编
- [ ] 模拟数据缺失 → LLM 明说"无数据"不编
- [ ] 模拟 cross-file drift → LLM 主动检测 + 提议修
```

## 实现路径

**fitness-coach v1 → v2 改造用了 11 步**（约 90 分钟）：

0. **设计 state.md 用户画像**（性别/年龄/身高/体重/经验/目标）— 首次加载 skill 时 Agent 自动采集，存为 state.md 基础数据
1. 大改 SKILL.md v2（加 4 大支柱）
2. 新增 7 个 references（每个支柱一个）
3. 改 morning-X-plan cron prompt（加今日推进 + EWMA + Recovery）
4. 改 evening-X-reminder cron prompt（加 autoregulation）
5. 新增 weekly-X-checkin cron（周报）
6. 新增 monthly-X-report cron（月报）
7. **用 `hermes -z` 验证**（详见下方"测试技术"段）
8. **发现 state.md 今日字段过期 bug**（`hermes -z` 跑暴露）
9. 加"今日推进检查"硬规则到 4 个 cron + SKILL.md + daily-metrics-tracking.md + cron-sync-checklist.md
10. 重跑 `hermes -z` 验证：主动 patch + 主动发现 Recovery 公式 drift
11. 修 SKILL.md Recovery 公式（跟 cron prompt 一致）

**实战教训**：
- 第 1 步别一次写完所有——分阶段改 + 每步验证
- 第 7-10 步（验证 + 发现 bug + 修）是 v2 真正的价值——v1 是"写完就用"，v2 是"边用边改边对账"
- **LLM 主动发现 drift**（第 10 步）证明 v2 抗幻觉硬规则实战生效

### 测试技术：`hermes --profile X -z` 不真触发的演练

**问题**：`hermes cron run <id>` 会 schedule 一次但实际跑要等 ticker tick（几分钟到几小时）；直接 `systemctl restart` + 等 9:30 太慢。

**解决**：用 `hermes --profile <profile> -z "<prompt>"` 模拟一次 prompt 输入，**不真触发 schedule**，**不真发飞书**，**几秒内拿到 LLM 输出**：

```bash
# 模拟 morning-X-plan 跑一次，看输出格式对不对
hermes --profile <profile> -z "按 morning-fitness-plan v2.1 prompt 跑一遍今日（<日期>）推送"

# 模拟 weekly-checkin 看周报格式
hermes --profile <profile> -z "按 weekly-checkin v2.1 prompt 跑一遍本周周报"
```

**优点**：
- 几秒出结果（vs 等 ticker 几分钟）
- 不真发平台消息（v2 演练安全）
- 输出在 stdout，可 grep / pipe / 完整 review

**用法模板**：

```bash
hermes --profile <profile> -z "按 <cron-name> v<X> prompt 跑一遍 <场景>。

执行流程：
1. cat <state.md 路径>
2. 提取今日字段日期，对比今天
3. [具体的 v2 逻辑]
4. 算 <算法>（python 算）
5. 出 <输出>

重点验证：<要验证的硬规则>"
```

**v2.1 实战用法**（脱敏示例）：
- Step 7：演练 v2 → 发现 state.md 今日过期
- Step 10：演练 v2.1 → LLM 主动发现 Recovery 公式 drift
- ✅ v2 改完第一时间 `hermes -z` 演练，发现问题立刻改，不要等"明早 9:30"

## 实战案例

- **fitness-coach v1 → v2**（健身教练 skill 实战案例）
  - 详细见 references/case-study-fitness-coach.md
  - 11 步改造，发现 2 个 cross-file drift（state.md 今日过期 + Recovery 公式）
  - 4 个 cron 全部加上"今日推进"和抗幻觉硬规则

## 关键设计原则

1. **数据 > 主观** — EWMA 比单日体重更可靠
2. **状态 > plan** — Recovery Score 决定训练量
3. **主动 > 被动** — cron 问，不等用户说
4. **抗幻觉 > 流畅** — 不编解释，不编数字，不编 paper
5. **同步 > 独立** — SKILL.md 和 cron prompt 必须保持一致（cron-sync-checklist）
6. **算法 > 经验** — python 算，不心算，不"差不多"
7. **范围 > 具体** — 推断给范围（X-Y 周），不给日期

## 已知坑

- ❌ v1 风格的"用户问 LLM 答" — 不需要这个 skill
- ❌ 一次性任务型 skill — 不适用
- ❌ 短周期项目（< 1 个月）— 不需要 mesocycle
- ❌ 没有 state.md — 必须有
- ❌ 只用 CLI 不接 cron — v2 必须接 cron
- ❌ 心算不写代码 — 必须用 python
- ❌ 改 SKILL.md 不改 cron prompt — cross-file drift 风险
- ❌ 算法公式在两个文件各写一份 — drift 真事故
- ❌ 将用户真实名称/个人标识写入公开发布的内容 — 示例对话、README、案例研究中的用户名/昵称/机器名/IP 等必须脱敏替换为通用占位符。所有公开文本中的示例必须用中性称呼，不可出现真实用户名

## README 设计要点（面向公开仓库）

当 coach skill 设计完成后发布到 GitHub 等公开平台，README 直接影响项目印象。

### 语言切换栏

GitHub 不自动检测读者语言。通用做法：

- README.md = 中文内容（GitHub 默认渲染）
- README_EN.md = 英文内容

两个文件顶部各放一个切换栏：

```markdown
<!-- README.md 顶部 -->
<p align="center">
  <a href="README_EN.md">:flag-us: English</a> · <b>:flag-cn: 中文</b>
</p>

<!-- README_EN.md 顶部 -->
<p align="center">
  <b>:flag-us: English</b> · <a href="README.md">:flag-cn: 中文</a>
</p>
```

### 高质量 README 要素

1. Badge 徽章行 — 版本、License、框架、平台等徽章（shields.io），整齐一行居中
2. **架构图 — 用 Mermaid 代码块（GitHub 原生渲染），不要用 ASCII 手画。推荐 Claude 风格配色：**
   ```mermaid
   %%{init: {'theme': 'base', 'themeVariables': {
     'background': '#fcf9f7',
     'primaryColor': '#6B4FBB',
     'primaryTextColor': '#fff',
     'primaryBorderColor': '#5a3fa8',
     'lineColor': '#D4A574',
     'secondaryColor': '#f5f0eb',
     'clusterBkg': '#fffcf9',
     'clusterBorder': '#e8ddd0',
     'nodeBorder': '#d4c5b8'
   }}}%%
   ```
   底色暖白 #fcf9f7、主色 Claude 紫 #6B4FBB、线条暖琥珀 #D4A574、边框柔米色。
3. 对比表格 — v1 vs v2（改造前 vs 改造后）的差异用表格呈现，比大段文字直观

### 健身领域特别注意事项

- 示例对话必须脱敏：展示用户与 Agent 的对话时用中性称呼（你），不可用真实用户名
- 数据示例用合理但不真实的数字：体重/训练量等示例数字要看起来合理但无法追溯到真人
- 突出通讯软件集成：明确写出通过飞书/微信/Telegram 自动推送，这是区别于传统健身 App 的核心卖点

## Reference Files

- `references/algorithm-templates.md` — 6 个核心 Python 算法（EWMA / 1RM / Recovery / Mesocycle / Autoregulation / 今日字段）
- `references/cron-sync-checklist.md` — 跨文件同步验证流程（防 cross-file drift）
- `references/case-study-fitness-coach.md` — 健身教练 skill v1→v2 完整实战案例
- `references/pre-publish-privacy-check.md` — 发布到公开仓库前的隐私扫描脚本 + 脱敏清单
