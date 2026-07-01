# DevPulse AI — Agile Team Intelligence Dashboard

A **full-stack Agile analytics dashboard** built as a single-page application (SPA) with JavaScript, Chart.js, and AI-driven standup generation. Simulates a production-grade PM tool with burndown charts, velocity tracking, risk analysis, and a natural language standup bot.

## Live Demo

[Open DevPulse AI](https://sneha-chandra.github.io/devpulse-ai) &nbsp;·&nbsp; [GitHub](https://github.com/Sneha-chandra/devpulse-ai)

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        DevPulse AI — SPA                        │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Data Layer                           │   │
│   │                                                         │   │
│   │  SPRINT_DATA{}   ← 14-day sprint, velocity history     │   │
│   │  TEAM_DATA{}     ← 5 members, workload, contributions  │   │
│   │  RISKS[]         ← 6 risks with probability/impact     │   │
│   │  STANDUPS{}      ← per-member yesterday/today/blockers │   │
│   │  AI_RESPONSES{}  ← keyword → response map (NLP chat)   │   │
│   └─────────────────────┬───────────────────────────────────┘   │
│                          │                                       │
│   ┌──────────────────────▼───────────────────────────────────┐  │
│   │              Tab Router & State Manager                  │  │
│   │                                                          │  │
│   │  showTab(id)     ← hide all, show active tab div        │  │
│   │  Active tab      ← CSS class toggle on nav buttons      │  │
│   │  Chart lifecycle ← destroy() old chart before redraw    │  │
│   └──────────────────────┬───────────────────────────────────┘  │
│                          │                                       │
│   ┌──────────────────────▼───────────────────────────────────┐  │
│   │           Chart.js Visualization Engine                  │  │
│   │                                                          │  │
│   │  Burndown Chart     → Line chart (ideal / actual /      │  │
│   │                        projected lines)                  │  │
│   │  Velocity Chart     → Bar chart (last 6 sprints)        │  │
│   │  Contributor Chart  → Doughnut chart (commit %)         │  │
│   │  Workload Chart     → Horizontal bar (hours/week)       │  │
│   │  Risk Trend Chart   → Line chart (risk score over time) │  │
│   └──────────────────────┬───────────────────────────────────┘  │
│                          │                                       │
│   ┌──────────────────────▼───────────────────────────────────┐  │
│   │                 AI Feature Modules                       │  │
│   │                                                          │  │
│   │  generateStandup()  ← NLP template + typeout animation  │  │
│   │  answerQuestion()   ← keyword routing + fuzzy matching  │  │
│   │  animateCount()     ← KPI counter animation (rAF)       │  │
│   │  renderRiskMatrix() ← probability × impact heatmap      │  │
│   └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Project File Structure

```
devpulse-ai/
├── index.html          # Complete SPA — all tabs, charts, AI logic, styles
└── README.md
```

> Single-file SPA pattern: production apps split this into components; this demonstrates all concepts in one auditable file.

---

## Feature Architecture

### Tab 1 — Sprint Overview

```
Sprint Metrics → animateCount() → KPI cards (velocity, burndown %, blockers, team health)
                              ↓
Burndown Chart (Chart.js Line)
  Datasets:
    - Ideal line   : linear interpolation from totalPoints to 0
    - Actual line  : daily logged remaining points
    - Projected    : linear regression from last 3 actual data points
```

### Tab 2 — AI Standup Generator

```
Team Member Selection (radio)
        ↓
generateStandup(member)
        ↓
  Template string assembly:
    - Yesterday: STANDUPS[member].yesterday
    - Today: STANDUPS[member].today
    - Blockers: STANDUPS[member].blockers (if any)
    - Mood: random from MOOD_PHRASES[]
        ↓
Typeout Animation:
    setInterval(30ms) per character → append to <div>
    clearInterval on complete
```

### Tab 3 — Team Analytics

```
Velocity Bar Chart    → Chart.js Bar, 6-sprint history, gradient fill
Contributor Doughnut  → Chart.js Doughnut, 5 members, commit percentages
Workload Horizontal   → Chart.js Bar (indexAxis: 'y'), hours/week per member
```

### Tab 4 — Risk Register

```
RISKS[] → renderRiskMatrix()
  For each risk:
    - Map probability (0-1) × impact (0-10) → matrix cell color
    - Heatmap: Low (green) / Medium (yellow) / High (orange) / Critical (red)
    - Risk score = probability × impact

Risk Trend Chart → Line chart, 8-week risk score history
```

### Tab 5 — AI Chat Assistant

```
User input → answerQuestion(query)
    ↓
Keyword tokenization (split on spaces, lowercase)
    ↓
Match against AI_RESPONSES{} keyword keys:
  "velocity", "blocker", "risk", "sprint", "burndown", "team", etc.
    ↓
Return matched response (or fallback generic answer)
    ↓
Typeout animation → append to chat log
```

---

## Technical Knowledge Implemented

### Chart.js Concepts
| Chart Type | Dataset | Key Config |
|---|---|---|
| Line (Burndown) | 3 datasets (ideal/actual/projected) | `tension: 0.4`, `borderDash` for ideal |
| Bar (Velocity) | Sprint velocity over time | `backgroundColor` gradient array |
| Doughnut (Contributor) | Per-member commit % | `cutout: '60%'`, center label plugin |
| Horizontal Bar (Workload) | Hours/week | `indexAxis: 'y'` |
| Line (Risk Trend) | Weekly risk score | `fill: true`, area gradient |

### JavaScript Patterns
| Pattern | Where Used |
|---|---|
| **Module pattern** | Data constants separated from UI logic |
| **requestAnimationFrame** | `animateCount()` for smooth KPI counter animation |
| **setInterval typeout** | `generateStandup()` — streams text character by character |
| **Chart lifecycle management** | `chart.destroy()` before re-render prevents canvas reuse errors |
| **Event delegation** | Single listener on nav for tab switching |
| **Template literals** | Multi-line standup string assembly |
| **Keyword-to-response routing** | `AI_RESPONSES` dict with `Object.keys()` iteration |

### CSS Architecture
| Concept | Application |
|---|---|
| CSS Custom Properties | `--primary`, `--accent`, `--bg-dark` color system |
| CSS Grid | Dashboard KPI card row, risk matrix |
| Flexbox | Nav bar, chat message layout |
| CSS Transitions | Tab fade-in, card hover lift |
| Glassmorphism | `.card { backdrop-filter: blur(10px); background: rgba(...) }` |
| Responsive Design | `@media (max-width: 768px)` breakpoints |

---

## Data Model

```javascript
// Sprint Data
SPRINT_DATA = {
  name: "Sprint 14",
  totalPoints: 84,
  completedPoints: 61,
  daysElapsed: 9,
  totalDays: 14,
  burndown: [84, 79, 72, 65, 61, 55, 50, 46, 40, ...], // daily remaining
  velocity: [72, 68, 75, 80, 77, 84]  // last 6 sprints
}

// Risk Model
RISKS = [{
  name: "API Integration Delay",
  probability: 0.7,
  impact: 8,
  owner: "Priya S.",
  mitigation: "..."
}]
```

---

## How to Run

```bash
open index.html
# or
python3 -m http.server 8080
# → http://localhost:8080
```

---

## Skills Demonstrated

`Single-Page Application (SPA) Architecture` · `Chart.js (5 chart types)` · `Natural Language Processing (keyword routing)` · `Data Visualization` · `Agile Metrics` · `Burndown / Velocity Analysis` · `Risk Matrix` · `CSS Glassmorphism` · `requestAnimationFrame` · `Typeout Animation` · `Responsive Web Design` · `Vanilla JavaScript (ES6+)`
