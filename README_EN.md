<p align="center">
  <b>🇺🇸 English</b> · <a href="README.md">🇨🇳 中文</a>
</p>

<h1 align="center">🏋️ TiSu · AI Fitness Coach Framework</h1>

<p align="center">
  <b>Turn your AI Agent into a 24/7 personal fitness coach</b><br>
  Works through Feishu / WeChat / Telegram — daily check-ins, state tracking, mesocycle programming
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-1.0.0-blue" alt="Version">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License">
  <img src="https://img.shields.io/badge/AI-Agent%20Framework-orange" alt="AI Agent">
  <img src="https://img.shields.io/badge/platform-Feishu%20%7C%20WeChat%20%7C%20Telegram%20%7C%20CLI-brightgreen" alt="Platforms">
</p>

---

## 🔥 In One Sentence

**Connect your AI Agent to messaging apps. It asks "how was your workout?" every day, adjusts plans dynamically based on your state. No apps to open, no logs to keep, no plans to write.**

---

## 🎯 What It Does

Instead of forcing you to open a fitness app and log manually, **TiSu flips the model** — the AI Agent messages you proactively through your chat app.

Every morning you get a message:

```
📱 [Feishu/WeChat/Telegram] Good morning! How are we feeling today?
  Weight: ? | Sleep: ? | Energy (1-10): ?
  Mood (1-10): ? | Stress (1-10): ?
```

Your reply:

```
175 | 7h | 8 | 9 | 4
```

The Agent automatically:
- ✅ Logs everything to your training journal (`state.md`)
- ✅ Computes EWMA weight trend with Python
- ✅ Calculates Recovery Score → decides push/maintain/deload
- ✅ Generates today's workout (Push/Pull/Legs + sets/reps/weights)
- ✅ Records RPE → auto-adjusts next session's weight

**No app to open. Your chat app IS your fitness coach UI.**

---

## 🏗️ Architecture

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {
  'background': '#ffffff',
  'primaryColor': '#2563eb',
  'primaryTextColor': '#fff',
  'primaryBorderColor': '#1d4ed8',
  'lineColor': '#64748b',
  'secondaryColor': '#f0f9ff',
  'tertiaryColor': '#f8fafc',
  'clusterBkg': '#f8fafc',
  'clusterBorder': '#e2e8f0',
  'nodeBorder': '#94a3b8',
  'fontSize': '13px'
}}}%%
graph TB
    subgraph PLATFORM["📱 Messaging Platform"]
        direction LR
        F1["Feishu"] --- F2["WeChat"]
        F2 --- F3["Telegram"]
        F3 --- F4["CLI"]
    end

    subgraph AGENT["🤖 AI Agent Core"]
        direction LR
        A1["🕐 Daily Cron<br/><small>9:30 check-in · 18:30 review</small>"]
        A2["📊 State Engine<br/><small>Recovery Score · EWMA · RPE</small>"]
        A3["🛡️ Anti-Hallucination<br/><small>Read-first · Python compute</small>"]
    end

    subgraph DATA["💾 Data & Knowledge"]
        direction LR
        D1["📝 state.md<br/>Training Journal"]
        D2["📚 references/<br/>Knowledge Base"]
        D3["🧮 Python Algorithms<br/>EWMA / Recovery<br/>Templates"]
    end

    PLATFORM <==>|"real-time messages"| AGENT
    AGENT -.->|"daily push"| PLATFORM
    AGENT ==>|"read/write"| DATA
```

> 💡 **Flow**: Your message arrives from chat app → Agent processes (compute state, check knowledge, anti-hallucination guard) → Training plan pushed back. All automated, you just reply.

---

## ⏰ Daily Auto-Pilot

| Time | Action | What the Agent Does |
|------|--------|-------------------|
| **9:30 AM** 🐦 | Morning check-in | Read state.md → Ask 6 metrics → Compute Recovery → Generate plan |
| **Post-workout** 💪 | RPE logging | Ask "how was that set?" → Auto-adjust next weight |
| **6:30 PM** 🌆 | Evening review | Daily summary + autoregulation decision |
| **Monday 9:30** 📊 | Weekly report | 7-day trend analysis + next week recommendations |
| **1st of Month** 📈 | Monthly report | Mesocycle progress + strategy adjustment (bulk/cut switch) |

---

## 🛡️ Anti-Hallucination for Fitness

The #1 problem with AI fitness coaches: **fabricated exercises, made-up papers, invented timelines.**

| ❌ Common Hallucination | ✅ TiSu's Approach |
|------------------------|-------------------|
| "Ab wheel rollout requires quadriceps engagement" (fabricated anatomy) | `read_file` the reference first. If not found, say "I don't know" |
| "According to a 2019 RP study..." (invented citation) | Only cite content from `references/`. No citation = no claim |
| "You'll bench 80kg in 3 weeks" (invented timeline) | "Based on current EWMA trend +0.5kg/week, ~4-6 weeks **if consistent**" |
| Mental math on 1RM / EWMA (likely wrong) | Python computation with full trace shown |

---

## 💡 Design Philosophy

```
Data > Subjective    — EWMA beats single-day weight readings
State > Plan         — Recovery Score drives volume, not a fixed cycle
Proactive > Reactive — Agent asks first, doesn't wait for you
Algorithm > Guess    — Python compute, never mental math
Range > Date         — Give ranges (X-Y weeks), never specific dates
```

---

## 🚀 Quick Start

```bash
git clone https://github.com/chouxiangdick/TiSu.git
cd TiSu
cat SKILL.md
cat references/case-study-fitness-coach.md
```

**Requires an AI Agent framework** with cron scheduling + messaging gateway support (e.g. Hermes Agent).

---

## 📚 Repo Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Core methodology: 4 pillars + 4 anti-hallucination rules + 11-step upgrade process |
| `references/algorithm-templates.md` | 6 Python algorithm templates (EWMA / 1RM / Recovery / Mesocycle / Autoregulation / Today field) |
| `references/cron-sync-checklist.md` | Multi-cron coordination + cross-file consistency checks |
| `references/case-study-fitness-coach.md` | Full case study: v1 (reactive) to v2 (proactive) fitness coach in 11 steps |

---

## 📜 License

MIT
