<div align="center">

# 🛡️ CyberSentinel AI

### Real-Time Network Intrusion Detection System powered by Machine Learning

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)](https://streamlit.io/)
[![Vite](https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white)](https://vitejs.dev/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

> A full-stack SOC (Security Operations Center) platform that trains, compares, and deploys multiple ML models to detect cyber intrusions in real time — with an interactive React dashboard and an educational XAI (Explainable AI) engine.

**🚀 Live Deployments**

| Service | Platform | URL |
|---|---|---|
| 🖥️ React Frontend | **Vercel** | *(https://cyber-sentinel-ai-ten.vercel.app/)* |
| ⚙️ FastAPI Backend | **Render** | ** |
| 📊 Streamlit SOC Dashboard | **Render / Streamlit Cloud** | ** |

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Features](#-features)
- [ML Models](#-ml-models)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Quick Start](#-quick-start)
  - [Backend (FastAPI)](#1-fastapi-backend)
  - [Frontend (React + Vite)](#2-react-frontend)
  - [Streamlit SOC Dashboard](#3-streamlit-soc-dashboard)
- [Deployment](#-deployment)
  - [Deploy Backend to Render](#deploy-fastapi-backend-to-render)
  - [Deploy Frontend to Vercel](#deploy-react-frontend-to-vercel)
- [API Reference](#-api-reference)
- [Dataset](#-dataset)
- [Screenshots](#-screenshots)

---

## 🔍 Overview

**CyberSentinel AI** is an end-to-end network intrusion detection system built for real-world SOC workflows. It ingests network traffic data, trains five machine learning classifiers, compares them in a live leaderboard, and streams simulated packet predictions through a real-time dashboard with full Explainable AI (XAI) breakdowns.

The system is built across two frontends:

- **Streamlit SOC Dashboard** — A Python-native, feature-rich analyst console for model training, XAI, and simulation.
- **React Dashboard (Vercel)** — A sleek, modern interactive UI connected to the FastAPI backend, suitable for production deployments and demos.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CyberSentinel AI                        │
├──────────────────┬──────────────────┬───────────────────────┤
│  React Frontend  │  FastAPI Backend │  Streamlit Dashboard  │
│  (Vercel)        │  (Render)        │  (Standalone)         │
│                  │                  │                       │
│  React 18        │  POST /api/train │  Streamlit 1.25+      │
│  Vite 6          │  GET  /api/models│  Plotly / Pandas      │
│  Tailwind CSS 4  │  POST /api/predict  XGBoost / sklearn    │
│  Recharts        │  GET  /api/health│  SHAP Explainability  │
│  Framer Motion   │  GET  /api/system│                       │
│  react-icons     │  FastAPI + Uvicorn                       │
└──────────────────┴──────────────────┴───────────────────────┘
                          │
                  ┌───────▼────────┐
                  │   ML Engine    │
                  │ Random Forest  │
                  │ Decision Tree  │
                  │ Gaussian NB    │
                  │ XGBoost        │
                  │ MLP Neural Net │
                  └────────────────┘
```

---

## ✨ Features

### 🧠 Machine Learning
- Train **5 classifiers simultaneously** with one click
- Auto-selects the **best model by F1 score**
- **Model registry** to switch active model on the fly
- SHAP + feature importance explainability engine
- ROC-AUC, Precision, Recall, F1, Confusion Matrix per model

### 🚨 Real-Time SOC Operations
- **Live packet simulation** — classifies packets with confidence scores every 500ms
- **Adaptive threat level** — LOW / MODERATE / HIGH based on rolling 30-packet window
- Live packet log with source IP, protocol, label, and confidence
- Session metrics: Total Packets, Blocked, Unique IPs, Attack Rate

### 📈 Model Comparison
- Side-by-side accuracy, F1, ROC-AUC, and training time charts
- **Best model** and **active model** summary cards
- Per-model Confusion Matrix viewer

### 🔬 Explainable AI (XAI) Engine
- Global Feature Importance charts (tree-based models)
- Auto-generated **Model Behavior Analysis** narrative
- Local Sample Explanation — "Why was this packet flagged?"
- Confidence gauge (HIGH / MODERATE / LOW) with probability distribution
- Responsible AI notice with model limitations guidance

### 🖥️ System Health
- Live RAM and CPU monitoring via `psutil`
- High memory usage warnings

---

## 🤖 ML Models

| Model | Strengths |
|---|---|
| **Random Forest** | High accuracy, robust, low overfitting |
| **Decision Tree** | Fast, interpretable, good baseline |
| **Gaussian Naïve Bayes** | Extremely lightweight, good for skewed data |
| **XGBoost** | Gradient boosting, state-of-the-art performance |
| **MLP Neural Network** | Deep learning approximation, captures non-linearity |

All models are multi-class classifiers supporting **Normal** traffic and multiple intrusion types (DoS, DDoS, Probe, R2L, etc.)

---

## 🛠️ Tech Stack

### Frontend — React (`/cyber-dashboard`)
| Package | Version | Purpose |
|---|---|---|
| React | 18.2 | UI framework |
| Vite | 6.x | Build tool / dev server |
| TailwindCSS | 4.x | Utility-first styling |
| Framer Motion | 11.x | Animations |
| Recharts | 2.x | Charts & graphs |
| react-simple-maps | 3.x | Geo-threat visualization |
| react-icons | 5.x | Icon library |

### Backend — FastAPI (`/cyber-dashboard/backend`)
| Package | Purpose |
|---|---|
| FastAPI | REST API framework |
| Uvicorn | ASGI server |
| scikit-learn | ML models |
| XGBoost | Gradient boosting |
| Pandas / NumPy | Data processing |
| psutil | System metrics |
| joblib | Model serialization |

### Streamlit Dashboard (`/IntrusionDetectionDashboard`)
| Package | Purpose |
|---|---|
| Streamlit ≥ 1.25 | Dashboard framework |
| Plotly ≥ 5.15 | Interactive charts |
| scikit-learn ≥ 1.3 | ML models |
| XGBoost ≥ 1.7.5 | Gradient boosting |
| SHAP ≥ 0.42 | Explainability |
| Matplotlib | Additional plots |

---

## 📁 Project Structure

```
CyberSentinel-AI/
│
├── IntrusionDetectionDashboard/        # Streamlit SOC Dashboard
│   ├── app.py                          # Main Streamlit app (841 lines)
│   ├── config.py                       # App configuration & constants
│   ├── requirements.txt                # Python dependencies
│   ├── Dockerfile                      # Docker deployment config
│   ├── assets/
│   │   └── style.css                   # Custom dark theme CSS
│   ├── utils/
│   │   ├── preprocessing.py            # Data loading & feature engineering
│   │   ├── training.py                 # Model training logic
│   │   ├── evaluation.py               # Metrics & chart generation
│   │   ├── model_io.py                 # Save/load models (joblib)
│   │   ├── explainability.py           # SHAP integration
│   │   └── logger.py                   # Logging setup
│   └── models/                         # Saved model artifacts (.joblib)
│
├── cyber-dashboard/                    # React Frontend + FastAPI Backend
│   ├── src/                            # React components & pages
│   ├── index.html                      # HTML entry point
│   ├── vite.config.js                  # Vite configuration
│   ├── package.json                    # Node.js dependencies
│   ├── .env.production                 # Production environment variables
│   └── backend/                        # FastAPI ML backend
│       ├── server.py                   # FastAPI app & REST endpoints
│       ├── requirements.txt            # Python dependencies
│       └── ml/                         # ML data + engine modules
│           ├── data.py                 # Data loading & preprocessing
│           └── engine.py               # Model training, eval, simulation
│
├── train_model.py                      # Standalone model training script
├── .gitignore                          # Excludes datasets, venv, artifacts
└── README.md                           # This file
```

---

## 🚀 Quick Start

### Prerequisites
- Python 3.10+
- Node.js 18+
- npm 9+

### 1. FastAPI Backend

```bash
# Navigate to backend
cd cyber-dashboard/backend

# Create and activate virtual environment
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Start the server
uvicorn server:app --host 0.0.0.0 --port 8000 --reload
```

API will be live at `http://localhost:8000`
Swagger docs at `http://localhost:8000/docs`

---

### 2. React Frontend

```bash
# Navigate to frontend
cd cyber-dashboard

# Install dependencies
npm install

# Set development environment variable
# Create .env.development:
echo "VITE_API_URL=http://localhost:8000" > .env.development

# Start dev server
npm run dev
```

Dashboard will be live at `http://localhost:5173`

---

### 3. Streamlit SOC Dashboard

```bash
# Navigate to dashboard
cd IntrusionDetectionDashboard

# Create and activate virtual environment
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run the app
streamlit run app.py
```

Dashboard will be live at `http://localhost:8501`

> **Note:** Place your CSV dataset files (e.g., `dataset-part1.csv`) in the project root directory (`Detect cyber Intrusion/`). The app will automatically discover and load them. The system also supports **synthetic fallback mode** when no dataset is found.

---

## ☁️ Deployment

### Deploy FastAPI Backend to Render

1. Push your code to GitHub (datasets are excluded via `.gitignore`).
2. Go to [render.com](https://render.com) → **New Web Service**.
3. Connect your GitHub repository.
4. Configure the service:

| Setting | Value |
|---|---|
| **Root Directory** | `cyber-dashboard/backend` |
| **Runtime** | `Python 3` |
| **Build Command** | `pip install -r requirements.txt` |
| **Start Command** | `uvicorn server:app --host 0.0.0.0 --port $PORT` |

5. Deploy and copy your Render URL (e.g., `https://cybersentinel-api.onrender.com`).

> ⚠️ **Important:** The backend starts in **synthetic mode** when no dataset is available on Render. Packet simulation and model comparisons work without real CSV files.

---

### Deploy React Frontend to Vercel

1. Go to [vercel.com](https://vercel.com) → **New Project**.
2. Import your GitHub repository.
3. Configure build settings:

| Setting | Value |
|---|---|
| **Root Directory** | `cyber-dashboard` |
| **Framework Preset** | `Vite` |
| **Build Command** | `npm run build` |
| **Output Directory** | `dist` |

4. Add **Environment Variable**:

| Key | Value |
|---|---|
| `VITE_API_URL` | `https://your-render-backend-url.onrender.com` |

5. Click **Deploy** 🚀

---

## 📡 API Reference

Base URL: `https://your-render-url.onrender.com`

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | Health check — returns status & model count |
| `POST` | `/api/train` | Train one or all ML models |
| `GET` | `/api/models` | List all trained models with metrics |
| `POST` | `/api/set-active/{model_name}` | Switch the active model |
| `POST` | `/api/predict` | Simulate packet predictions |
| `POST` | `/api/simulation/reset` | Reset simulation state |
| `GET` | `/api/system` | RAM and CPU system metrics |
| `GET` | `/api/dashboard` | Aggregated dashboard stats |

### Example: Train All Models

```bash
curl -X POST https://your-api.onrender.com/api/train \
  -H "Content-Type: application/json" \
  -d '{"sample_size": 100000}'
```

### Example: Simulate 5 Packets

```bash
curl -X POST https://your-api.onrender.com/api/predict \
  -H "Content-Type: application/json" \
  -d '{"count": 5}'
```

---

## 📊 Dataset

This project uses the **CICIDS / NSL-KDD / UNSW-NB15** style network intrusion datasets. Datasets are **not included** in this repository due to their large size (10+ GB).

### Expected Format

CSV files named `dataset-part1.csv`, `dataset-part2.csv`, etc. placed in the project root. The system auto-discovers and concatenates all parts.

### Recommended Datasets

| Dataset | Source |
|---|---|
| CICIDS 2017/2018 | [UNB CIC](https://www.unb.ca/cic/datasets/) |
| NSL-KDD | [UNB CIC NSL-KDD](https://www.unb.ca/cic/datasets/nsl.html) |
| UNSW-NB15 | [UNSW Research](https://research.unsw.edu.au/projects/unsw-nb15-dataset) |

> **Without a dataset:** Both the Streamlit app and FastAPI backend operate in **synthetic fallback mode** — they generate dummy traffic for demonstration purposes.

---

## 📸 Screenshots

> *Screenshots will appear here once the application is deployed.*

---

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Built with ❤️ by [Soubhagya Jain](https://github.com/SoubhagyaJain)**

⭐ Star this repo if you found it useful!

</div>
