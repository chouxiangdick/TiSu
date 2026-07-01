---
name: cron-sync-checklist
description: 跨文件同步验证流程——SKILL.md 和 cron prompt 不一致导致 cross-file drift（v2.1 实战发现 2 个：state.md 今日过期 + Recovery 公式）。**v2 强制每次改 SKILL.md 必查**。
---

# Cron Sync Checklist（跨文件同步验证）

**v2.1 实战发现**：LLM 主动检测到 SKILL.md Recovery 公式 40/30/30 ≠ cron prompt 33/33/34，主动提议修。这就是 cron-sync-checklist 流程的价值。

## 为什么要做这个

**4 个 cron + 1 个 SKILL.md = 5 个文件描述同一套行为**。任何一个不一致 = 用户体验分裂。

**真实事故**（fitness-coach 案例）：

1. **Push 日 cron 错带腿部动作**（实战事故 #1）
   - SKILL.md Push 块：上肢 + 核心（**无腿部**）
   - morning-X-plan prompt：Push 块**写了腿部动作**
   - 后果：用户问"push 为什么带腿" → LLM 硬解释"健腹轮跪姿股四头"

2. **cron 不读 user profile**（实战事故 #2）
   - SKILL.md：4 场景 reference
   - cron prompt：**不读状态源**（按 cycle_index 算）
   - 后果：休息日还推训练内容，用户质疑

3. **state.md 今日字段过期**（v2.1 真事故）
   - state.md "今日" 字段停在过去的日期
   - cron prompt 说"严格按 state.md 今日字段"
   - 后果：推送时跟 cycle 推算冲突

4. **Recovery 公式不一致**（v2.1 真事故）
   - SKILL.md 公式：`(sleep/8)*40 + (10-fatigue)*30 + mood*30`（v1 错误，sum 超 100）
   - cron prompt 公式：`(sleep/8)*33 + (10-fatigue)/9*33 + mood/10*34`（v1.1 正确）
   - 后果：cron 用 33/33/34 算出来 65 分正常，SKILL.md 用 40/30/30 算出来 362.5 明显错

## 同步验证流程（5 步）

### Step 1：列出当前 cron

```bash
hermes --profile <profile> cron list
```

确认 4 个 cron 都存在（morning / evening / weekly / monthly）。

### Step 2：读每个 cron 的 prompt

```bash
# cron jobs.json 在 <hermes-home>/profiles/<profile>/cron/jobs.json
python3 -c "
import json
with open('<hermes-home>/profiles/<profile>/cron/jobs.json') as f:
    data = json.load(f)
jobs = data if isinstance(data, list) else data.get('jobs', [])
for j in jobs:
    print('=== ' + j.get('name', '?') + ' ===')
    print(j.get('prompt', '(no prompt)'))
    print()
" > /tmp/cron-prompts.txt
```

### Step 3：对比 SKILL.md 关键段落

**v2 强制对比表**：

| SKILL.md 段落 | 必须同步的 cron |
|--------------|----------------|
| **Core Principles 原则 7（抗幻觉）** | 所有 4 个 cron |
| **Verification 段（今日字段检查）** | 所有 4 个 cron |
| **Data Collection §DC.1（6 问）** | morning-X-plan |
| **Data Collection §DC.2（3 问）** | evening-X-reminder |
| **Data Collection §DC.3（4 问）** | weekly-X-checkin |
| **Data Collection §DC.4（月报）** | monthly-X-report |
| **Trend Analysis §TA.1-4（算法）** | morning-X-plan + weekly-X-checkin + monthly-X-report |
| **Trend Analysis §TA.4 Recovery Score 公式** | 所有用到 Recovery 的 cron（**重点对账**）|
| **Mesocycle Programming §MP.3（Deload 触发）** | morning-X-plan |
| **Autoregulation Protocol** | evening-X-reminder |
| **State Monitoring 综合决策** | morning-X-plan |
| **Pitfalls 抗幻觉** | 所有 4 个 cron |
| **DM.0 今日字段规范** | 所有 4 个 cron |

### Step 4：手动更新不一致的 cron

```bash
# 用 hermes cron edit 更新 prompt
hermes --profile <profile> cron edit <cron_id> --prompt "..."
```

### Step 5：测试 cron 立即跑一次

```bash
# 用 hermes -z 模拟一次输出（不真触发飞书推送）
hermes --profile <profile> -z "按 <cron-name> prompt 跑一遍今日推送"
```

看输出是否符合 SKILL.md 最新规则。

#### Step 5.1：测试技术详解（`hermes -z` 演练模式）

`hermes --profile <profile> -z "<prompt>"` 是 v2 验证的关键工具。**不真触发 schedule**、**不真发飞书**、**几秒拿到 LLM 输出**——这是 v2.1 实战发现 2 个 cross-file drift 的核心方法。

**为什么需要演练模式**：
- `hermes cron run <id>` 会 schedule 一次但实际跑要等 ticker tick（几分钟到几小时）
- `systemctl restart` + 等 9:30 太慢
- v2 改完必须立即验证，不能等

**v2.1 实战用法**：

```bash
# Step 7 演练 v2 → 发现 state.md 今日过期
hermes --profile <profile> -z "按 morning-fitness-plan v2 prompt 跑一遍今日推送"

# 验证 v2.1
hermes --profile <profile> -z "按 morning-fitness-plan v2.1 prompt 跑一遍今日推送"
```

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

**Pitfall**：
- ❌ `hermes -z` 不带 `--profile` → 走主实例，不是 profile 的 gateway 上下文
- ❌ 把 `hermes -z` 输出当最终输出（演练 vs 真 cron 输出略有差异——LLM 温度、上下文窗口）
- ✅ v2 改完**第一时间** `hermes -z` 演练，发现问题立刻改，**不要等"明早 9:30"**
- ✅ 把 v2.1 实战用法写到 cron prompt 第一段（演练模板）—— 让 LLM 自己会演练

## 自动化对账脚本（可选）

```bash
#!/bin/bash
# sync_check.sh - 检查 SKILL.md 和 cron prompt 一致性

PROFILE=${1:-default}
SKILL="<hermes-home>/profiles/${PROFILE}/skills/<your-skill>/SKILL.md"
CRON_JOBS="<hermes-home>/profiles/${PROFILE}/cron/jobs.json"

echo "=== 1. 提取 SKILL.md 关键段落 ==="
grep -A 5 "Recovery Score\|mesocycle\|EWMA\|deload" "$SKILL" > /tmp/skill-key.txt

echo "=== 2. 提取所有 cron prompt ==="
python3 -c "
import json
with open('$CRON_JOBS') as f:
    data = json.load(f)
jobs = data if isinstance(data, list) else data.get('jobs', [])
for j in jobs:
    print(f'=== {j.get(\"name\")} ===')
    print(j.get('prompt', ''))
    print()
" > /tmp/cron-prompts.txt

echo "=== 3. 对比 Recovery 公式 ==="
echo "SKILL.md:"
grep -A 3 "Recovery = " "$SKILL" | head -5
echo ""
echo "Cron prompts:"
grep -A 3 "recovery_score\|Recovery = " /tmp/cron-prompts.txt | head -20

echo "=== 4. 对比 mesocycle ==="
echo "SKILL.md:"
grep -A 5 "W1\|W5\|Deload" "$SKILL" | head -10
echo ""
echo "Cron prompts:"
grep -A 5 "W1\|W5\|Deload" /tmp/cron-prompts.txt | head -20
```

## v2 强制：每次自进化的 Step 7

SKILL.md 的 `Skill Maintenance` 段落 Step 7 必须包含 cron-sync-check：

```
7. **cron 同步检查**（v2 强制）：
   - 列出当前 cron（`hermes cron list`）
   - 对每个 cron cat 它的 prompt
   - 跟 SKILL.md 关键段落对比（用上面的对比表）
   - 不一致 → `hermes cron edit` 同步
   - 测试：cron run 看输出
   - 跑 sync_check.sh 自动化对账
```

## 实战案例

### 案例 1：Push 日 cron 错带腿部动作

- **症状**：用户问"X 训练为什么带 Y"
- **诊断**：LLM grep SKILL.md 和 cron prompt 对比，发现不一致
- **修法**：`hermes cron edit morning-X-plan`，删除 X 块里错误的 Y 动作
- **验证**：cron run 一次，确认 X 不含 Y

### 案例 2：cron 不读 user profile

- **症状**：休息日还推训练内容
- **诊断**：cron prompt 没"必读 state.md"指令
- **修法**：cron prompt 加"【必做第一步】读 state.md"
- **验证**：cron run 休息日，确认静默

### 案例 3：state.md 今日字段过期

- **症状**：推送时，state.md 今日字段是过去的日期
- **诊断**：cron prompt 没说"今日字段过期要主动 patch"
- **修法**：4 个 cron prompt + SKILL.md Verification 段 + daily-metrics-tracking.md + cron-sync-checklist.md 全部加"今日推进"逻辑
- **验证**：cron run v2.1，输出 "⚠️ state.md 今日字段过时 X 天，已按 cycle 推算 + 主动 patch"

### 案例 4：Recovery 公式不一致

- **症状**：LLM 主动检测到 SKILL.md 公式 40/30/30 vs cron 33/33/34
- **诊断**：v1 公式（错误，sum 超 100）vs v1.1 修正版
- **修法**：patch SKILL.md 段用 v1.1 公式（跟 cron 一致）
- **验证**：两个文件 grep 公式，确认一致

## Pitfalls

- ❌ 改 SKILL.md 不改 cron prompt（最大风险）
- ❌ 改 cron prompt 不测（必须 cron run 验证）
- ❌ 新加 reference 不加 cron 主动采集
- ❌ 新加 cron 不在 SKILL.md 列出
- ❌ 不同步 .v1.bak（出问题时没法回滚）
- ❌ 手动编辑 jobs.json（必须用 `hermes cron edit`）
- ❌ 算法公式 SKILL.md 和 cron prompt 各写一份（**v2.1 真事故**）
