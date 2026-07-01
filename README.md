# TiSu — Methodology for Building Long-Term Companion AI Coach Skills

> **Upgrade your AI skills from "user asks → LLM answers" (v1) to "proactive tracking + state monitoring + mesocycle programming + anti-hallucination" (v2)**
>
> Use cases: Fitness / Learning / Nutrition / Writing / Investing / Meditation / Habit Building

---

## 🏗️ The 4 Pillars

| Pillar | v1 (Reactive) | v2 (Proactive) |
|--------|--------------|----------------|
| **Proactive Tracking** | Wait for user input → LLM responds | Cron asks → Compute state → Generate plan |
| **State Monitoring** | Mechanical cycle-based plans | Recovery Score drives decisions (4 dimensions) |
| **Mesocycle Programming** | Simple rotation | 4-week + 1-week deload mesocycle |
| **Anti-Hallucination** | LLM fabricates plausible explanations | 4 hard rules enforced at runtime |

## ⏰ 4 Cron Roles

| Cron | Frequency | Purpose |
|------|-----------|---------|
| `morning-X-plan` | Daily 9:30 | Check-in + daily plan |
| `evening-X-reminder` | Daily 18:30 | Review + autoregulation |
| `weekly-X-checkin` | Monday 9:30 | Weekly review + trend analysis |
| `monthly-X-report` | 1st of month 9:30 | Monthly report + strategy adjustment |

## 🛡️ 4 Anti-Hallucination Hard Rules

1. **User reports anomaly** → `read_file` first, never fabricate explanations
2. **Cite data** → Only from references, never invent papers/URLs
3. **Compute numbers** → Use Python, never mental math
4. **Predict future** → Give a range (X-Y weeks), never a specific date

## 📁 Repository Structure

```
TiSu/
├── README.md                    # English
├── README_zh.md                 # Chinese
├── SKILL.md                     # Main method file (4 pillars + 11-step process)
├── .gitignore
└── references/
    ├── algorithm-templates.md   # 6 core algorithms (EWMA / 1RM / Recovery / Mesocycle / Autoregulation / Today field)
    ├── cron-sync-checklist.md   # Cross-file drift prevention workflow
    └── case-study-fitness-coach.md # Real-world fitness coach v1→v2 case study
```

## 🚀 Quick Start

```bash
git clone https://github.com/chouxiangdick/TiSu.git
cd TiSu
cat SKILL.md
cat references/case-study-fitness-coach.md
```

## 🎯 When to Use This

- Fitness / nutrition coaching
- Learning / exam coaching
- Writing / creative coaching
- Investing / financial coaching
- Meditation / mental health coaching
- Habit building coaching

Any AI skill that needs **long-term companionship + data tracking + proactive adjustment + state awareness**.

## ✨ Design Principles

1. **Data > Subjective** — EWMA beats single-day data points
2. **State > Plan** — Recovery Score determines training volume
3. **Proactive > Reactive** — Cron asks, don't wait for the user
4. **Anti-Hallucination > Fluency** — Never fabricate explanations, numbers, or papers
5. **Sync > Independence** — SKILL.md and cron prompts must agree
6. **Algorithm > Experience** — Python compute, never mental math
7. **Range > Specifics** — Give ranges, not exact dates

## 📜 License

MIT
