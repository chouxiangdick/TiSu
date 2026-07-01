# TiSu（提苏）— 长期陪伴型 AI Coach Skill 设计方法论

> **把"用户问 → LLM 答"的工具型 skill，升级为"主动跟踪 + 状态关注 + 周期化 + 抗幻觉"的私教型 skill**
>
> 适用场景：健身 / 学习 / 营养 / 写作 / 投资 / 冥想 / 习惯养成

---

## 🏗️ 4 大支柱

| 支柱 | v1（被动） | v2（主动） |
|------|-----------|-----------|
| **主动跟踪** | 等用户问 → LLM 答 | cron 主动问 → 算状态 → 出计划 |
| **状态关注** | 按 cycle 机械出 plan | Recovery Score 驱动（4 维度评估） |
| **周期化编程** | 简单循环 | 4 周 + 1 周 deload mesocycle |
| **抗幻觉** | LLM 编合理解释 | 4 条硬规则强制约束 |

## ⏰ 4 个 cron 角色

| cron | 频率 | 做什么 |
|------|------|--------|
| `morning-X-plan` | 每日 9:30 | 日问 + 日计划 |
| `evening-X-reminder` | 每日 18:30 | 日复盘 + autoregulation |
| `weekly-X-checkin` | 每周一 9:30 | 周复盘 + 趋势分析 |
| `monthly-X-report` | 每月 1 号 9:30 | 月报 + 战略调整 |

## 🛡️ 4 条抗幻觉硬规则

1. **用户报告异常** → 先 `read_file` 查源，不编解释
2. **引用数据** → 只引 references 里的，不编 paper/URL
3. **计算数字** → 用 python 算，不心算
4. **推断未来** → 给范围（X-Y 周），不给具体日期

## 📁 仓库结构

```
TiSu/
├── README.md                    # English
├── README_zh.md                 # 中文
├── SKILL.md                     # 主文件（4 大支柱 + 11 步流程）
├── .gitignore
└── references/
    ├── algorithm-templates.md   # 6 个核心算法（EWMA / 1RM / Recovery / Mesocycle / Autoregulation / 今日字段）
    ├── cron-sync-checklist.md   # 跨文件同步验证流程（防 cross-file drift）
    └── case-study-fitness-coach.md # 健身教练 v1→v2 实战案例
```

## 🚀 快速上手

```bash
git clone https://github.com/chouxiangdick/TiSu.git
cd TiSu
cat SKILL.md                     # 看完整方法论
cat references/case-study-fitness-coach.md  # 看实战案例
```

## 🎯 适用场景

- 健身/营养教练
- 学习/考试教练
- 写作/创作教练
- 投资/理财教练
- 冥想/心理教练
- 习惯养成教练

任何需要**长期陪伴 + 数据跟踪 + 主动调整 + 状态关注**的 AI Coach skill 都适用。

## ✨ 设计原则

1. **数据 > 主观** — EWMA 比单日数据更可靠
2. **状态 > plan** — Recovery Score 决定训练量
3. **主动 > 被动** — cron 问，不等用户说
4. **抗幻觉 > 流畅** — 不编解释，不编数字，不编 paper
5. **同步 > 独立** — SKILL.md 和 cron prompt 必须一致
6. **算法 > 经验** — python 算，不心算
7. **范围 > 具体** — 推断给范围，不给日期

## 📜 License

MIT
