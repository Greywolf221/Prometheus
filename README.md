<div align="center">

#  PROMETHEUS
### Autonomous River Pollution Surveillance System

[![Next.js](https://img.shields.io/badge/Next.js-16-black?style=flat-square&logo=next.js)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110-009688?style=flat-square&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python)](https://python.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.7-3178C6?style=flat-square&logo=typescript)](https://typescriptlang.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.4-F7931E?style=flat-square&logo=scikit-learn)](https://scikit-learn.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

*An Engineering Project in Community Service (EPICS) — VIT Bhopal University, 2026*

</div>

---

## 📌 Overview

Prometheus is a full-stack autonomous surveillance system that simulates how a **drone swarm** can monitor a river network for pollution events in real time. It integrates:

- A **procedurally generated river graph** with physics-based pollution diffusion
- A **DFS/BFS-driven drone swarm** that autonomously searches for the contamination source
- Two **trained ML models** that predict where the pollution came from and where it will spread next
- A **live dashboard** built in Next.js that visualizes everything as it happens

The system addresses a real-world problem: conventional river monitoring relies on fixed sensors that are expensive, sparse, and passive. Prometheus proposes an intelligent, autonomous, and scalable alternative.

---

## 🎯 Problem Statement

Industrial discharge, agricultural runoff, and urban waste continuously degrade river water quality. When a pollution event occurs, two critical questions must be answered quickly:

1. **Where did the contamination originate?** (Source Localization)
2. **Where will it reach next, and when?** (Spread Forecasting)

This system answers both using autonomous drone navigation and machine learning.

---

## 🧠 Key Features

| Feature | Description |
|---|---|
| **Procedural River Generation** | Generates a unique river topology every run — main channel, lateral tributaries, sub-branches |
| **Physics-Based Pollution Diffusion** | Directional decay model: downstream spreads at 15%/tick, upstream at 4%/tick |
| **Drone Swarm (1–4 drones)** | DFS traversal with shared visited set — drones coordinate to divide the search space |
| **Efficient Search Mode** | Pollution-gradient-guided DFS — drones prefer high-contamination neighbors |
| **AI Spread Forecast** | Click any node → ML model predicts ETA and toxicity level at that point |
| **AI Source Localization** | Scans all 40 nodes → Random Forest returns top-3 probable pollution origin nodes |
| **Deploy to Suspects** | One-click dispatch of swarm to the AI's top predicted source nodes |
| **Mission Debrief** | Post-mission summary: which drone found it, optimal path, battery used, nodes scanned |
| **Real-time Mission Log** | Terminal-style log feed with timestamped events and severity levels |

---

## 🏗️ System Architecture

```text
┌─────────────────────────────────────────┐      HTTP/REST
│         FRONTEND (Next.js 16)           │ ◄──────────────► ┌──────────────────────────┐
│                                         │                   │   BACKEND (FastAPI)       │
│  page.tsx ── Orchestrator               │                   │                           │
│  ├── river-map.tsx ── SVG visualizer    │  POST /predict-   │  spread_model.pkl         │
│  ├── control-panel.tsx ── Mission ctrl  │  spread           │  RandomForestRegressor    │
│  ├── mission-log.tsx ── Event feed      │  ──────────────►  │  Input: distance_jumps    │
│  └── metric-card.tsx ── Live stats      │                   │  Output: ETA, toxicity    │
│                                         │  POST /predict-   │                           │
│  lib/river-simulation.ts               │  source           │  source_model.pkl         │
│  ├── Graph generation (procedural)      │  ──────────────►  │  RandomForestClassifier   │
│  ├── BFS/DFS traversal                  │                   │  Input: 40-node snapshot  │
│  ├── Pollution diffusion physics        │                   │  Output: top-3 origins    │
│  └── Toxicity color mapping             │                   │                           │
└─────────────────────────────────────────┘                   └──────────────────────────┘
```

---

## 🤖 Machine Learning Pipeline

### Dataset Generation (`generate_data.py`)

Generates 8,000 synthetic pollution simulations using the same decay physics as the frontend:

- **Dataset 1** — Spread Prediction: records `distance_jumps → ETA, target_toxicity` (~3.5 MB)
- **Dataset 2** — Source Localization: records 40-node pollution snapshots → actual source node (~1.8 MB)

### Model Training (`train_models.py`)

| Model | Type | Input | Output | File |
|---|---|---|---|---|
| Spread Predictor | RandomForestRegressor (100 trees) | `distance_jumps` (int) | `eta_seconds`, `target_toxicity` | `spread_model.pkl` |
| Source Localizer | RandomForestClassifier (100 trees) | 40 pollution floats | top-3 source node IDs + probabilities | `source_model.pkl` |

### API Endpoints (`main.py`)

**POST `/predict-spread`**
```json
// Request
{ "origin_node": 31, "target_node": 5, "edges": [{"source": 0, "target": 1}, ...] }

// Response
{ "status": "success", "predicted_path": [31, 24, 15, 8, 5], "eta": 47.3, "predicted_toxicity": 0.42 }
```

**POST `/predict-source`**
```json
// Request
{ "node_pollutions": [0.05, 0.12, 0.87, ..., 0.03] }  // exactly 40 values

// Response
{ "status": "success", "top_predictions": [
    { "node_id": 31, "probability": 64.2 },
    { "node_id": 18, "probability": 21.1 },
    { "node_id": 22, "probability": 8.4 }
  ]
}
```

---

## 🚀 Running the Project

### Prerequisites

- Node.js 18+
- Python 3.11+
- npm or pnpm

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/prometheus-river-surveillance.git
cd prometheus-river-surveillance
```

### 2. Start the Frontend

```bash
npm install
npm run dev
```

Frontend will be running at `http://localhost:3000`

### 3. Start the AI Backend

```bash
cd prometheus-ai
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

Backend API will be running at `http://localhost:8000`

> Both services must be running simultaneously for AI features to work.
> The simulation and drone patrol work fully offline — only the AI Predict and AI Diagnostics buttons require the backend.

### 4. (Optional) Retrain the Models

If you want to regenerate training data and retrain from scratch:

```bash
cd prometheus-ai
python generate_data.py     # regenerates both CSVs
python train_models.py      # retrains and overwrites .pkl files
```

---

## 📁 Project Structure

```text
prometheus-river-surveillance/
│
├── app/                          # Next.js App Router
│   ├── page.tsx                  # Main application (orchestrator, ~37KB)
│   ├── layout.tsx                # Root layout
│   └── globals.css               # Global styles
│
├── components/
│   ├── prometheus/               # Domain-specific components
│   │   ├── river-map.tsx         # SVG river visualization
│   │   ├── control-panel.tsx     # Mission control UI
│   │   ├── mission-log.tsx       # Real-time event log
│   │   ├── metric-card.tsx       # Live stats display
│   │   ├── status-indicator.tsx  # System status dot
│   │   └── logo.tsx              # Prometheus branding
│   └── ui/                       # shadcn/ui primitive components
│
├── lib/
│   └── river-simulation.ts       # Core simulation engine
│       ├── generateRiverNetwork() # Procedural graph generation
│       ├── spreadPollution()      # Tick-based diffusion physics
│       ├── initializePollution()  # BFS-based initial distribution
│       └── getToxicityColor()     # Pollution → color mapping
│
├── prometheus-ai/                 # Python AI backend
│   ├── main.py                   # FastAPI server (2 endpoints)
│   ├── train_models.py           # Model training script
│   ├── generate_data.py          # Synthetic dataset generator
│   ├── requirements.txt          # Python dependencies
│   ├── spread_model.pkl          # Trained regressor
│   ├── source_model.pkl          # Trained classifier
│   ├── dataset_1_spread_prediction.csv
│   └── dataset_2_source_localization.csv
│
├── docs/
│   └── PROMETHEUS_EXPANDED_REPORT.docx   # Full EPICS Phase II report
│
├── .gitignore
├── LICENSE
├── package.json
└── README.md
```

---

## 🛠️ Tech Stack

**Frontend**
- Next.js 16 (App Router)
- React 19
- TypeScript 5.7
- Tailwind CSS v4
- shadcn/ui (Radix UI primitives)
- lucide-react (icons)

**Backend**
- FastAPI 0.110
- scikit-learn 1.4 (RandomForest models)
- NetworkX 3.2 (graph operations)
- pandas 2.2 (feature engineering)
- joblib (model serialization)

**Algorithms**
- Depth-First Search (drone traversal)
- Breadth-First Search (shortest path, pollution initialization)
- Pollution diffusion physics (directional decay model)
- Random Forest Regression (spread forecasting)
- Random Forest Classification (source localization)

---

## 📊 Results

| Metric | Value |
|---|---|
| Source Localization Accuracy | ~87% (top-1), ~96% (top-3) |
| Spread Prediction MAE (ETA) | ±3.2 seconds |
| Spread Prediction MAE (Toxicity) | ±0.04 |
| Training Data Size | 8,000 simulations per model |
| Avg. Drone Convergence (4 drones, efficient mode) | ~18 steps |

---

## 📄 Report

The full Phase II EPICS project report is available in [`docs/PROMETHEUS_EXPANDED_REPORT.docx`](docs/PROMETHEUS_EXPANDED_REPORT.docx).

It covers the motivation, literature review, system architecture, ML performance metrics, individual contributions, and future scope.

---

## 👥 Team

| Name | Reg. No. | Branch |
|---|---|---|

| Purab B Rao | 23BAC10016 | ECE (AI & Cybernetics) |

| Thilak Raaj Ganesan | 23BAI10724 | CSE (AI & ML) |

---

## 🔮 Future Scope

- Replace synthetic training data with real river sensor readings
- Integrate Graph Neural Networks (GNNs) for more accurate spread modeling
- Add Reinforcement Learning for optimized drone path planning
- Implement edge computing deployment on physical drone hardware
- Extend water quality parameters to include pH, turbidity, dissolved oxygen
- Mobile app for real-time field alerts and remote monitoring

---

<div align="center">
Made for EPICS — VIT Bhopal University, March 2026
</div>
