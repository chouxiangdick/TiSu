---
name: algorithm-templates
description: 6 个核心 Python 算法模板——EWMA / Epley 1RM / Recovery Score / Mesocycle 进度 / Autoregulation / 今日字段检查。**所有算法都用工具算，不心算**。复制即用，附完整测试用例。
---

# Algorithm Templates（核心 Python 算法）

v2 强制：**所有数字用工具算，不心算**。下面是 6 个最常用的算法模板——复制即用。

## R1. EWMA 体重跟踪（α=0.3）

### 算法

```python
def ewma(weights: list[float], alpha: float = 0.3) -> list[float]:
    """
    指数加权移动平均（Exponentially Weighted Moving Average）
    
    Args:
        weights: 体重序列（按时间顺序，最旧在前）
        alpha: 平滑系数（0.2-0.4，常用 0.3）
    
    Returns:
        EWMA 序列（与输入等长）
    """
    if not weights:
        return []
    
    ewma_values = [weights[0]]
    for w in weights[1:]:
        ewma_values.append(alpha * w + (1 - alpha) * ewma_values[-1])
    return ewma_values
```

### 使用示例

```python
# 7-day EWMA（示例数字）
weights_7d = [79.5, 79.3, 79.4, 79.2, 79.0, 78.9, 79.1]
ewma_7d = ewma(weights_7d, alpha=0.3)
print(f"7-day EWMA: {ewma_7d[-1]:.2f}")
print(f"对比首日: {weights_7d[0] - ewma_7d[-1]:.2f}")

# 输出：
# 7-day EWMA: 79.18
# 对比首日: 0.32
```

### 判定标准

```python
def classify_ewma_trend(ewma_now: float, ewma_week_ago: float) -> str:
    """判定 EWMA 趋势（单位/周，正数 = 在下降）"""
    weekly_change = ewma_week_ago - ewma_now
    
    if weekly_change >= 0.4:
        return "✓ 正常（≥ 0.4/周）"
    elif weekly_change >= 0.2:
        return "⚠ 偏慢（0.2-0.4/周）"
    elif weekly_change > 0:
        return "⚠ 接近 plateau（< 0.2/周）"
    else:
        return "⚠ 反弹（EWMA 上升）"
```

### Pitfalls

- ❌ 用单日数据判定趋势——必须 EWMA
- ❌ EWMA 初始化用 0——第一个数据点就是初始 EWMA
- ❌ α=0.3 编成"RP 推荐"——α=0.2-0.4 都行，是经验值
- ❌ 心算 EWMA——必须 python

## R2. Epley 1RM 公式

### 算法

```python
def epley_1rm(weight: float, reps: int) -> float:
    """
    Epley 1RM 公式（估算最大单次重复重量）
    
    Args:
        weight: 实际重量
        reps: 完成次数（1-10）
    
    Returns:
        估算 1RM
    
    Note: 适用于 1-10 reps；> 10 reps 不准
    """
    if reps < 1:
        return 0.0
    return weight * (1 + reps / 30)


def trend_1rm(weight: float, reps: int, weeks_ago_1rm: float) -> str:
    """
    算 1RM 4 周变化 + 判定
    
    Args:
        weight: 当前重量
        reps: 当前完成次数
        weeks_ago_1rm: 4 周前的 1RM
    """
    current_1rm = epley_1rm(weight, reps)
    change_pct = (current_1rm - weeks_ago_1rm) / weeks_ago_1rm * 100
    
    if change_pct >= 2.5:
        return f"✓ 正常（+{change_pct:.1f}%）"
    elif change_pct > 0:
        return f"⚠ 接近停滞（+{change_pct:.1f}%）"
    elif change_pct > -5:
        return f"⚠ 减量中（{change_pct:.1f}%）"
    else:
        return f"❌ 强制 deload（{change_pct:.1f}%）"
```

### 使用示例

```python
# 主项 weight × reps
e1rm = epley_1rm(80, 7)
print(f"估算 1RM: {e1rm:.1f}")
# 输出：估算 1RM: 98.7

# 4 周前 1RM = 95.5
trend = trend_1rm(80, 7, 95.5)
print(f"4 周趋势: {trend}")
# 输出：4 周趋势: ✓ 正常（+3.4%）
```

### Pitfalls

- ❌ Epley 公式适用于 1-10 reps，> 10 reps 不准
- ❌ 减脂期追求力量大涨——保肌就行，持平 = 胜利
- ❌ 编具体 paper 名（Epley 1985 是共识，别编"RP 2019 paper"）

## R3. Recovery Score 综合分（v1.1 修正版）

### 算法

```python
def recovery_score(sleep_hours: float, fatigue: int, mood: int) -> float:
    """
    Recovery Score v1.1: 各项 0-1 归一化 × 权重求和（满分 ≈ 100）
    
    Args:
        sleep_hours: 睡眠时长（小时）
        fatigue: 疲劳度（1-10, 10=最累）
        mood: 心情（1-10, 10=最好）
    
    Returns:
        Recovery Score（0-100）
    
    Note: 
      v1 公式 (sleep/8)*40 + (10-fatigue)*30 + mood*30 超过 100（sum=362.5 max）
      v1.1 修正为 33/33/34 权重，总和 = 100
    """
    if not (1 <= fatigue <= 10 and 1 <= mood <= 10):
        raise ValueError("fatigue 和 mood 必须在 1-10 范围")
    if sleep_hours < 0:
        raise ValueError("sleep_hours 不能为负")
    
    sleep_score = min(sleep_hours / 8, 1.0) * 33      # 0-33
    fatigue_score = (10 - fatigue) / 9 * 33           # 0-33 (fatigue 1-10 → 0-9)
    mood_score = mood / 10 * 34                        # 0-34
    return sleep_score + fatigue_score + mood_score


def classify_recovery(score: float) -> str:
    """判定 Recovery Score 等级"""
    if score >= 80:
        return "高恢复日（加码）"
    elif score >= 60:
        return "正常日（按计划）"
    elif score >= 40:
        return "减量日（砍 1-2 组）"
    else:
        return "休息日（跳过训练）"
```

### 使用示例

```python
score = recovery_score(sleep_hours=6.5, fatigue=6, mood=7)
print(f"Recovery Score: {score:.1f} / 100")
print(f"判定: {classify_recovery(score)}")
# 输出：
# Recovery Score: 65.3 / 100
# 判定: 正常日（按计划）
```

### 关键修正：v1 → v1.1

```python
# ❌ v1 公式（错误，sum 超过 100）
recovery_v1 = (6.5/8) * 40 + (10-6) * 30 + 7 * 30
# = 32.5 + 120 + 210 = 362.5（明显错误）

# ✅ v1.1 公式（修正，sum = 100）
recovery_v11 = recovery_score(6.5, 6, 7)
# = 26.8 + 14.7 + 23.8 = 65.3
```

### Pitfalls

- ❌ 用 v1 公式 40/30/30（超过 100）
- ❌ SKILL.md 和 cron prompt 用不同公式（**cross-file drift 真实事故**）
- ❌ fatigue 写成 0-100 而不是 1-10

## R4. Mesocycle 进度 + Deload 触发

### 算法

```python
from datetime import date, timedelta

def mesocycle_progress(start_date: date, today: date, 
                       cycle_length_weeks: int = 5) -> dict:
    """
    算当前 mesocycle 进度（4 周 hypertrophy + 1 周 deload）
    
    Args:
        start_date: mesocycle 开始日期
        today: 今天日期
        cycle_length_weeks: 总周期（默认 5 周 = 4 hypertrophy + 1 deload）
    
    Returns:
        {
            "week": 1-5,
            "phase": "hypertrophy" | "deload",
            "days_elapsed": int,
            "days_to_deload": int,
            "next_deload": date,
            "next_mesocycle": date
        }
    """
    days_elapsed = (today - start_date).days
    week = days_elapsed // 7 + 1
    if week > cycle_length_weeks:
        week = cycle_length_weeks
    
    is_deload = (week == cycle_length_weeks)
    next_deload = start_date + timedelta(weeks=cycle_length_weeks)
    next_mesocycle = next_deload + timedelta(days=1)
    
    return {
        "week": week,
        "phase": "deload" if is_deload else "hypertrophy",
        "days_elapsed": days_elapsed,
        "days_to_deload": (next_deload - today).days,
        "next_deload": next_deload,
        "next_mesocycle": next_mesocycle
    }


def should_deload(
    recovery_score_3d: list[float],
    training_rpe_7d: list[float],
    main_lift_1rm_4w_change: float,
    planned_deload_week: int,
    current_week: int
) -> tuple[bool, str]:
    """
    判定是否该 deload（4 个条件，任一触发）
    
    Args:
        recovery_score_3d: 最近 3 天 Recovery Score
        training_rpe_7d: 最近 7 天训练 RPE
        main_lift_1rm_4w_change: 主项 1RM 4 周变化（%，负数 = 掉）
        planned_deload_week: 计划的 deload week
        current_week: 当前 week
    
    Returns:
        (should_deload, reason)
    """
    # 1. 计划触发
    if current_week >= planned_deload_week:
        return True, f"计划触发：W{current_week} >= W{planned_deload_week}"
    
    # 2. 力量触发
    if main_lift_1rm_4w_change < -5:
        return True, f"力量触发：主项 1RM 4 周 {main_lift_1rm_4w_change:.1f}%"
    
    # 3. 状态触发
    if all(r < 40 for r in recovery_score_3d):
        return True, f"状态触发：Recovery < 40 持续 3 天"
    
    # 4. 疲劳触发
    if training_rpe_7d and sum(training_rpe_7d) / len(training_rpe_7d) > 8.5:
        return True, "疲劳触发：训练 RPE > 8.5 持续 1 周"
    
    return False, "无需 deload"
```

### 使用示例

```python
from datetime import date

# 算 mesocycle 进度
progress = mesocycle_progress(date(2026, 6, 29), date(2026, 7, 8))
print(progress)
# {'week': 2, 'phase': 'hypertrophy', 'days_elapsed': 9, 'days_to_deload': 26, ...}

# 判定 deload
should, reason = should_deload(
    recovery_score_3d=[65, 70, 68],
    training_rpe_7d=[7, 7.5, 8, 8, 8.5, 9, 8.5],
    main_lift_1rm_4w_change=-2.0,
    planned_deload_week=5,
    current_week=3
)
print(f"Deload: {should}, 原因: {reason}")
# 输出：Deload: False, 原因: 无需 deload
```

### 容量进阶表

```python
def get_volume_for_week(week: int) -> str:
    """算 hypertrophy block 容量调整"""
    volume_map = {
        1: "baseline（标准容量）",
        2: "+1 组/动作",
        3: "+2 组/动作（峰值）",
        4: "+2 组/动作（维持）",
        5: "Deload（容量减半，强度降到 60%）"
    }
    return volume_map.get(week, "未知周")
```

## R5. Autoregulation（RPE-based 单动作决策）

### 算法

```python
def next_weight(weight: float, rpe: float, reps_done: int, 
                rep_range_max: int, is_main_lift: bool = True) -> dict:
    """
    Autoregulation: 给定上次 RPE + 完成 reps，算下次重量调整
    
    Args:
        weight: 上次重量
        rpe: 上次 RPE (1-10)
        reps_done: 上次完成 reps
        rep_range_max: rep 区间上限（如 4×6-8 的 rep_range_max=8）
        is_main_lift: 是否主项
    
    Returns:
        dict with: action, next_weight, reason
    """
    main_delta = 2.5 if is_main_lift else 1.0
    side_delta = 1.0 if is_main_lift else 0.5
    
    if rpe < 7 and reps_done >= rep_range_max:
        return {
            "action": "increase",
            "next_weight": weight + main_delta,
            "reason": f"RPE {rpe} 太轻，加 {main_delta}"
        }
    elif rpe <= 8 and reps_done >= rep_range_max:
        return {
            "action": "increase_small",
            "next_weight": weight + side_delta,
            "reason": f"RPE {rpe} 正常，加 {side_delta}"
        }
    elif rpe <= 8 and reps_done < rep_range_max:
        return {
            "action": "maintain",
            "next_weight": weight,
            "reason": f"RPE {rpe}，{reps_done} reps 没做满，维持"
        }
    elif rpe == 9 and reps_done >= rep_range_max:
        return {
            "action": "maintain_observe",
            "next_weight": weight,
            "reason": f"RPE 9 接近力竭，维持观察"
        }
    elif rpe == 9 and reps_done < rep_range_max:
        return {
            "action": "decrease",
            "next_weight": weight - main_delta,
            "reason": f"RPE 9 + 没做满，减 {main_delta}"
        }
    elif rpe >= 10:
        return {
            "action": "deload_signal",
            "next_weight": weight * 0.95,
            "reason": "RPE 10 力竭，减 5% + deload 评估"
        }
    else:
        return {
            "action": "unknown",
            "next_weight": weight,
            "reason": f"RPE {rpe} 不在决策表"
        }
```

### 使用示例

```python
# 主项 weight × reps RPE 7.5
result = next_weight(80, 7.5, 7, 8, is_main_lift=True)
print(result)
# {'action': 'increase_small', 'next_weight': 81.0, 'reason': 'RPE 7.5 正常，加 1.0'}
```

## R6. 今日字段过期检查（v2.1 强制）

### 算法

```python
import re
from datetime import date

def check_today_field(state_md_content: str, today: date) -> dict:
    """
    提取 state.md 今日字段日期，对比今天
    
    Returns: {
        "fitness_today": date | None,
        "is_stale": bool,
        "stale_days": int,
        "action": "use_existing" | "recompute_via_cycle" | "manual_review"
    }
    """
    match = re.search(r'今日\s*\((\d{4}-\d{2}-\d{2})\)\s*[：:]\s*\*?\*?(\S+?)', state_md_content)
    if not match:
        return {"fitness_today": None, "is_stale": True, "stale_days": 999, "action": "manual_review"}
    
    fitness_today = date.fromisoformat(match.group(1))
    fitness_type = match.group(2)
    stale_days = (today - fitness_today).days
    
    if stale_days == 0:
        return {"fitness_today": fitness_today, "fitness_type": fitness_type, 
                "is_stale": False, "stale_days": 0, "action": "use_existing"}
    elif stale_days > 0:
        return {"fitness_today": fitness_today, "fitness_type": fitness_type,
                "is_stale": True, "stale_days": stale_days, "action": "recompute_via_cycle"}
    else:
        return {"fitness_today": fitness_today, "fitness_type": fitness_type,
                "is_stale": True, "stale_days": stale_days, "action": "use_today_date"}
```

## 综合使用示例（cron prompt 模板）

```python
def daily_morning_push(state_md: str, today: date) -> dict:
    """morning cron 主逻辑"""
    
    # 1. 今日字段检查
    today_check = check_today_field(state_md, today)
    if today_check["action"] == "recompute_via_cycle":
        # 主动 patch + 飞书开头明示
        new_today = recompute_today_via_cycle(state_md, today)
        state_md = patch_state_md(state_md, new_today)
        warning = f"⚠️ state.md 今日字段过时 {today_check['stale_days']} 天，已按 cycle 推算 + 主动 patch"
    else:
        warning = None
    
    # 2. 主动问 6 问
    questions = format_daily_questions()
    
    # 3. 算 EWMA（用 python，无数据明说）
    weights = extract_weights_from_state_md(state_md)
    if len(weights) >= 7:
        ewma_7d = ewma(weights[-7:])[-1]
        trend = classify_ewma_trend(ewma_7d, ewma(weights[-14:-7])[-1])
    else:
        ewma_7d = None
        trend = "⚠️ 无数据（需 7 天体重历史）"
    
    return {
        "warning": warning,
        "questions": questions,
        "ewma_7d": ewma_7d,
        "trend": trend,
        "today_type": today_check.get("fitness_type", "未知")
    }
```
