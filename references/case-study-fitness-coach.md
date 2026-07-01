---
name: case-study-fitness-coach
description: fitness-coach v1 → v2 完整实战案例——健身教练 skill 升级 11 步流程。**两次 cross-file drift 事故 + LLM 主动发现 + 修复**全过程。**所有方法论的实战验证**。
---

# Case Study: fitness-coach v1 → v2（健身教练 skill 实战案例）

**时间跨度**：约 1 个月
**改造耗时**：约 90 分钟（11 步）
**结果**：v2 落地，4 个 cron 全部带"今日推进 + 抗幻觉"硬规则

## 背景

**v1 痛点**：
1. **"老出现幻觉"**（用户原话）
2. **机械出训练计划**——不关注用户状态
3. **不主动跟踪**——等用户主动报数据
4. **没有周期化**——只是简单动作循环
5. **没有抗幻觉保障**——LLM 编合理解释

**v2 目标**：
- 主动跟踪（cron 主动问 6 问）
- 状态关注（Recovery Score 驱动）
- 周期化编程（4 周 hypertrophy + 1 周 deload）
- 抗幻觉硬规则（4 条）

## 11 步改造流程

### Step 1: 大改 SKILL.md v2

**做了什么**：
- 加 4 大支柱（核心原则 5/6/7）
- 加 4 个 references 段落（Data Collection / Trend Analysis / Mesocycle Programming / Autoregulation）
- 加 4 条抗幻觉硬规则
- 加 Verification 段

**耗时**：30 分钟
**变化**：SKILL.md 从 327 行 → 640 行

### Step 2: 新增 7 个 references

**做了什么**：
- `daily-metrics-tracking.md`（DC.1-4 + 4 个 cron 主动问句格式）
- `trend-analysis.md`（EWMA / plateau / 1RM / Recovery 4 个算法）
- `mesocycle-programming.md`（4 周 hypertrophy + 1 周 deload）
- `autoregulation-protocol.md`（RPE 决策表）
- `state-monitoring.md`（4 维度状态判断）
- `plateau-intervention.md`（refeed / diet break 决策）
- `cron-sync-checklist.md`（跨文件同步）

**耗时**：30 分钟

### Step 3: 改 morning-X-plan cron prompt

**做了什么**：
- 5 步流程（必读 state.md / 6 问 / 算趋势 / 出 plan / 输出格式）
- 加 python 算 EWMA / Recovery / 1RM
- 加红旗症状检测

**耗时**：10 分钟

### Step 4: 改 evening-X-reminder cron prompt

**做了什么**：
- 训练日 18:30 提醒 + 训练后 3 问
- python 算 autoregulation（next_weight）
- 重量上报 patch

**耗时**：5 分钟

### Step 5: 新增 weekly-X-checkin cron

**做了什么**：
- 每周一 9:30
- 4 问（周维度指标 + 最大变化）
- 周报（7-day EWMA + 力量 trend + Recovery 均分 + mesocycle 进度 + 下周建议）

**耗时**：5 分钟

### Step 6: 新增 monthly-X-report cron

**做了什么**：
- 每月 1 号 9:30
- 不问，自动算月报
- 30-day EWMA + 力量 1RM 月度变化 + 目标差距 + 下月建议

**耗时**：3 分钟

### Step 7: 验证：演练一次

**做了什么**：
- 触发 v2 输出
- 检查 6 问格式 / EWMA 算法 / Recovery 计算

**结果**：✅ v2 跑通

**但发现一个隐藏问题**（Step 8）——

### Step 8: 发现 state.md 今日字段过期

**症状**：推送时，state.md "今日" 字段是过去的日期
**实际跑出来**：state.md "今日" 字段停在过去某天，但今天是新一天
**LLM 行为**：
- cycle 推算 = 今天 = 正确训练日
- 主动指出 state.md 状态过时
- 主动建议 patch state.md

**这是 v2 抗幻觉硬规则的胜利**——LLM 没硬按旧今日字段，没编合理解释，主动暴露数据不一致。

### Step 9: 加"今日推进检查"硬规则

**做了什么**：
- 4 个 cron prompt 全部加"今日字段过期检查 + 主动 patch"
- SKILL.md Verification 段加"今日字段检查"项
- daily-metrics-tracking.md 加 DM.0 段
- cron-sync-checklist.md 加"今日字段规范"项

**耗时**：15 分钟

### Step 10: 重跑验证 v2.1 今日推进生效

**做了什么**：
- 重跑 morning-X-plan
- LLM 输出：
  - ⚠️ 警告块（state.md 今日字段过时 N 天）
  - 主动 patch state.md
  - EWMA / Plateau / Strength Trend 全部"无数据"（不编）
  - Recovery Score 用 placeholder + 公式展示

**结果**：✅ v2.1 今日推进生效

**但 LLM 主动发现第二个 cross-file drift**（Step 11）——

### Step 11: 修复 Recovery 公式 cross-file drift

**症状**：LLM 主动检测到 SKILL.md Recovery 公式 `(sleep/8)*40 + (10-fatigue)*30 + mood*30`（v1 错误，sum 超 100）≠ cron prompt 公式 `(sleep/8)*33 + (10-fatigue)/9*33 + mood/10*34`（v1.1 正确）
**LLM 行为**：
- 主动发现 drift
- 主动提议修
- 暂时用 cron prompt 的 33/33/34 算

**修法**：
- patch SKILL.md Recovery 段用 v1.1 公式（跟 cron 一致）
- 验证两个文件 grep 公式一致

**结果**：✅ cross-file drift 修复

**这是 v2 cron-sync-checklist 实战胜利**——LLM 主动对账两个文件，发现不一致，提议修。

## 关键战绩

| 项 | v1 | v2 |
|---|---|---|
| 主动跟踪 | ❌ | ✅ 4 个 cron 主动问 6 问 |
| 状态关注 | ❌ | ✅ Recovery Score 驱动 |
| 周期化 | ❌ | ✅ 4 周 + 1 周 deload |
| 抗幻觉 | ❌ | ✅ 4 条硬规则 + 主动发现 drift |
| 今日推进 | ❌ | ✅ cron 启动时主动检查 + patch |
| cross-file drift 检测 | ❌ | ✅ LLM 主动发现 + 提议修 |

## 11 步流程的可复用性

**fitness-coach 的 11 步流程 = personal-coach-skill-design skill 的实战验证**。

任何长期陪伴型 skill 升级都可以套用：

1. **诊断 v1 痛点**（用户报告 + 调研）
2. **大改 SKILL.md v2**（加 4 大支柱）
3. **新增 references**（每个支柱 1-2 个）
4. **改 2 个现有 cron**（morning + evening）
5. **新增 2 个 cron**（weekly + monthly）
6. **验证：演练一次**
7. **发现隐藏问题**（v2.1 真事故：今日字段过期）
8. **加硬规则修问题**（今日推进 + 数据缺失处理）
9. **重跑验证生效**
10. **LLM 主动发现 cross-file drift**（v2.1 真事故：Recovery 公式）
11. **修 drift + 跑 sync_check**

**11 步是底线**，实战可能加更多步（每发现一个 bug 就多一步）。

## v2 需求分析

> "请你修复，我要的是绝对的顶级和专业，我还要它主动跟踪体脂率和体重！不是一味的只会给出机械的训练计划，还要关注我的状态！"

→ v2 需求 = 主动 + 状态 + 顶级专业
→ v2 4 大支柱 = 主动 + 状态 + 周期化 + 抗幻觉（顶级专业的技术保障）
→ v2.1 实战 + 2 个 cross-file drift 修复 = 顶级专业的工程严谨

## 给其他 skill 作者的教训

1. **改 SKILL.md 别忘了改 cron prompt**——cross-file drift 是最大风险
2. **首次演练必暴露隐藏问题**——v1 跑得好好的，v2 第一次跑就发现今日过期
3. **LLM 主动发现 drift 是抗幻觉硬规则的胜利**——v2 设计目标达成
4. **11 步流程别跳过任何一步**——v2.1 实战发现 2 个 bug，都是 v2 流程本身挖出来的
5. **算法公式别在两个文件各写一份**——v2.1 真事故，SKILL.md 和 cron prompt 的 Recovery 公式不一致

## Pitfalls（v1 → v2 升级时）

- ❌ 一次写完所有代码——分阶段 + 每步验证
- ❌ SKILL.md 改了 cron prompt 不改（**最大风险**）
- ❌ 演练只跑一次——v1 跑得好不代表 v2 没问题
- ❌ 算法公式两份文件各写（v2.1 真事故）
- ❌ 真人对话触发 patch 时忘了更新日期（v2.1 真事故）
- ❌ 抗幻觉硬规则不写到 cron prompt（只在 SKILL.md 写没用）

## Reference Files

- 主文件 `SKILL.md`（4 大支柱 + 4 条抗幻觉硬规则）
- `references/algorithm-templates.md`（6 个 Python 算法）
- `references/cron-sync-checklist.md`（跨文件同步）
- `references/case-study-fitness-coach.md`（本文件）
