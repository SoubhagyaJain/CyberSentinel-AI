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
| 🖥️ React Frontend | **Vercel** | *(Add your Vercel URL here)* |
| ⚙️ FastAPI Backend | **Render** | *(Add your Render URL here)* |
| 📊 Streamlit SOC Dashboard | **Render / Streamlit Cloud** | *(Add your Streamlit URL here)* |

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Machine Learning Pipeline](#-machine-learning-pipeline)
  - [Dataset & Data Engineering](#dataset--data-engineering)
  - [Model Training & Selection](#model-training--selection)
  - [Explainable AI (XAI) / SHAP](#explainable-ai-xai--shap)
- [Local Data Extraction Server](#-local-data-extraction-server)
- [Future Improvements & Roadmap](#-future-improvements--roadmap)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Quick Start](#-quick-start)
  - [Backend (FastAPI)](#1-fastapi-backend)
  - [Frontend (React + Vite)](#2-react-frontend)
  - [Streamlit SOC Dashboard](#3-streamlit-soc-dashboard)
- [Deployment Specifications](#-deployment-specifications)
- [API Reference](#-api-reference)
- [Contributing](#-contributing)

---

## 🔍 Overview

**CyberSentinel AI** bridges the gap between theoretical data science and practical cybersecurity analysis. Designed as an end-to-end network intrusion detection system (NIDS), it performs real-time traffic classification against modern cyber threats (DoS, DDoS, Probing, R2L). 

What sets this project apart is its **multi-model comparison pipeline** and **Explainable AI (XAI) integration**. Rather than acting as a "black box" that arbitrarily blocks IP addresses, CyberSentinel trains five disparate models simultaneously, visualizes their behavior using SHAP (SHapley Additive exPlanations), and provides plain-English justifications for its alerts.

---

## 🏗️ System Architecture

The project employs a decoupled, multi-tier microservices architecture consisting of a data ingestion layer, an inference API, and two distinct frontend consumption interfaces.

```mermaid
flowchart TD
    subgraph "Local Environment (Laptop/On-Premise)"
        DS[(10GB+ Network Datasets)]
        LDS[Local Data Server\ndata_server.py]
        DS -->|Chunked CSV Reads| LDS
    end

    subgraph "Cloud Infrastructure (Render)"
        API[FastAPI Backend Server]
        ML[ML Engine\nscikit-learn]
        
        LDS -->|HTTPS Stream via Ngrok| API
        API <-->|Train / Predict| ML
    end

    subgraph "Cloud Infrastructure (Vercel)"
        UI[React Dashboard]
    end
    
    subgraph "Alternative UI (Streamlit Cloud / Render)"
        SOC[Streamlit SOC Console]
    end

    API <-->|REST API JSON| UI
    SOC <-->|Direct Python Import / Fallback| ML
    
    classDef cloud fill:#0d1117,stroke:#2d3139,stroke-width:2px,color:#fff;
    classDef local fill:#161b22,stroke:#0f5323,stroke-width:2px,color:#fff;
    class "Local Environment (Laptop/On-Premise)" local;
    class "Cloud Infrastructure (Render)" cloud;
    class "Cloud Infrastructure (Vercel)" cloud;
    class "Alternative UI (Streamlit Cloud / Render)" cloud;
```

### Architecture Deep Dive

1. **Local Data Server (Ingestion Source)**: Cloud platforms strictly limit free-tier disk space. Because cyber datasets exceed 10GB, `data_server.py` runs on your local machine and acts as an HTTP streaming server. It dynamically reads, interleaved, samples, and streams CSV rows to the cloud backend.
2. **FastAPI Backend (Inference & Aggregation)**: The Render-hosted API. It downloads memory-safe dataset chunks (`sample_size` parameterized), trains models locally using `joblib` artifacts, and holds application state in-memory (simulation logs, packet counts, system metrics).
3. **React Client (Consumer)**: Deployed on Vercel, this is the modern, reactive interface. It polls the FastAPI endpoints for live inferences (`/api/predict`), metrics (`/api/system`), and current model states.
4. **Streamlit Console**: An alternative, fully-native Python frontend focused on deep analytical workflows (SHAP charts, confusion matrices, localized explanations).

---

## 🧠 Machine Learning Pipeline

### Dataset & Data Engineering

CyberSentinel leverages large-scale Network Intrusion datasets (e.g., CICIDS-2017, NSL-KDD, UNSW-NB15) combining benign packet captures and active attack vectors.

**Feature Engineering & Memory Management:**
Processing 10GB+ of CSV data in Python routinely causes `MemoryError`. CyberSentinel solves this with aggressive memory alignment:
- **Fractional Iterative Loading:** Instead of `pd.read_csv()` loading entirely to memory, we ingest chunks and apply immediate `frac=0.3` down-sampling.
- **Type Downcasting:** 64-bit precision is unnecessary for packet metrics. We immediately cast `float64 -> float32` and `int64 -> int32`, cutting RAM requirements by ~50%.
- **Selected Dimensionality:** We slice the dataset strictly to the 15 most critical TCP/IP feature vectors (e.g., `TCP_WIN_SCALE_OUT`, `FLOW_DURATION_MILLISECONDS`, `PROTOCOL`, `DST_TOS`) out of 80+ potential features.

**Preprocessing Strategy:**
- Target variables (`LABEL`) are grouped and encoded via `LabelEncoder()`.
- Numeric features (like duration or byte counts) are normalized using `StandardScaler()` to ensure gradient-based methods converge rapidly.

### Model Training & Selection

The `train_model.py` and backend `engine.py` orchestrate a concurrent train-test-evaluation sequence using a standardized 70/30 split. 

1. **Random Forest (n=30)**: Highly resistant to overfitting; provides robust baseline feature importance metrics.
2. **Decision Tree (depth=4)**: Ultra-fast inference with completely transparent branching, used as a reliable fallback.
3. **Gaussian Naïve Bayes**: Extremely lightweight, assumes feature independence (an assumption often violated in networking, yet yields surprisingly fast anomaly detection).
4. **XGBoost (Future-Ready)**: Capable of handling massive class imbalances inherent in intrusion data.
5. **Multi-Layer Perceptron (MLP)**: Deep learning representation to capture nonlinear relationships in protocol state behaviors.

**Evaluation:**
The backend assesses models synchronously scoring them against:
* **F1-Score (Weighted):** The definitive metric due to heavy class imbalances (Benign traffic usually overwhelms Malicious).
* **Recall:** Critical in NIDS. A false negative (missing an attack) is catastrophically worse than a false positive.
* **Accuracy & Precision.**

The engine auto-promotes the model with the highest F1-score to **Active Status** for live API inference.

### Explainable AI (XAI) / SHAP

In enterprise SOC environments, dropping packets without explanation is unacceptable. CyberSentinel incorporates **SHAP** to enforce operational transparency:

1. **Global Importance:** Generates summary plots across the entire test set to prove the model is evaluating valid features (e.g., "The model flags anomalies primarily based on rapid `TCP_WIN_SCALE` changes, which accurately reflects a SYN flood").
2. **Local Interpretability (Per-Packet Analysis):** For any specific packet flagged as a threat, SHAP decomposes the prediction probabilties into exact feature contributions (e.g., "`FLOW_DURATION_MILLISECONDS` pushed the probability of 'ATTACK' up by +34%").

---

## 📡 Local Data Extraction Server

To bypass cloud storage limits, the project ships with `data_server.py`.

1. Place `dataset-partX.csv` files in the root directory.
2. Run `python data_server.py`. It starts an HTTP server on port `7860`.
3. The server features built-in basic token authorization.
4. Expose the port using ngrok: `ngrok http 7860`.
5. Supply the resulting HTTPS URL to the Render backend environment variables. The backend now uses your laptop as a distributed SAN!

---

## 🚀 Future Improvements & Roadmap

While this project succeeds beautifully as an analytical dashboard and ML pipeline demonstration, actual enterprise NIDS require further evolution.

<details>
<summary><b>1. Move from Batch CSV to Live Packet Sniffing</b></summary>
Currently, predictions run against simulated data streams. To make this a functional, deployable firewall appliance:
- Integrate `pyshark` or `scapy`.
- Write a daemon that binds to a physical network interface (e.g., `eth0`), actively sniffing packets.
- Convert raw PCAP frames into the 15-feature numeric vectors required by the model in real-time (Rolling window calculation).
</details>

<details>
<summary><b>2. Incorporate Temporal Sequences (Deep Learning)</b></summary>
Attacks like "Slowloris" or advanced probing don't happen in single packets—they are behaviors spanning thousands of packets over minutes.
- Expand from single-vector Scikit-Learn models to sequence models.
- Implement Recurrent Neural Networks (LSTMs) or Transformers in PyTorch/TensorFlow to evaluate the *context* and *timing* of a flow over X milliseconds.
</details>

<details>
<summary><b>3. Hardened Enterprise Backend Architecture</b></summary>
- **Database Layer**: Transition away from holding simulation state in Python memory. Implement PostgreSQL for relational data and Redis for high-speed rate limiting and streaming logs.
- **Message Queues**: Decouple the ML inference engine from the FastAPI thread via Celery or Apache Kafka. 
- **RBAC**: Implement JSON Web Tokens (JWT) for authentication and Role-Based Access Control (Analyst vs. Admin views).
</details>

<details>
<summary><b>4. Distributed Big Data Training</b></summary>
Instead of chunking, downcasting, and sampling 10GB of CSVs to survive single-node RAM limits:
- Rewrite the training pipeline using **PySpark** or **Dask**.
- Allow the models to iteratively train on the entirety of the 50GB+ dataset across a distributed server cluster.
</details>

---

## 🛠️ Tech Stack

### Frontend — React (`/cyber-dashboard`)
| Package | Version | Purpose |
|---|---|---|
| React | 18.2 | Component-based UI formulation |
| Vite | 6.x | High-speed ESM build tool / HMR |
| TailwindCSS | 4.x | Inline utility-class styling |
| Framer Motion | 11.x | Fluid DOM element transitions |
| Recharts | 2.x | Real-time SVG charting |
| react-simple-maps | 3.x | Geo-threat visualization |

### Backend — FastAPI (`/cyber-dashboard/backend`)
| Package | Purpose |
|---|---|
| FastAPI | Asynchronous REST endpoint generation |
| Uvicorn | High-performance ASGI server |
| scikit-learn | Tree-based models & linear algorithms |
| XGBoost | Gradient boosting framework |
| Pandas / NumPy | DataFrame handling / tensor math |
| psutil | Hardware performance monitoring |

### Streamlit Dashboard (`/IntrusionDetectionDashboard`)
| Package | Purpose |
|---|---|
| Streamlit ≥ 1.25 | Rapid prototyping state-based UI |
| Plotly ≥ 5.15 | Highly interactive analytical charts |
| SHAP ≥ 0.42 | Game-theoretic explainability engine |

---

## 📁 Project Structure

```text
CyberSentinel-AI/
│
├── IntrusionDetectionDashboard/        # (Standalone) Streamlit SOC UI
│   ├── app.py                          
│   ├── config.py                       
│   ├── utils/                          # Contains ML, eval, SHAP logic specific to Streamlit
│   └── models/                         # Local joblib output storage
│
├── cyber-dashboard/                    # (Main Application) React + FastAPI
│   ├── src/                            # React Component Tree
│   ├── index.html                      
│   ├── vite.config.js                  
│   └── backend/                        # The Python REST Engine
│       ├── server.py                   # Uvicorn entry point & Route definitions
│       ├── requirements.txt            
│       └── ml/                         
│           ├── data.py                 # Remote/Local dataset ingestion handler
│           └── engine.py               # The core sklearn integration logic
│
├── train_model.py                      # Standalone barebones CLI trainer script
├── data_server.py                      # The Ngrok-ready dataset file server
└── .gitignore                          
```

---

## 🏁 Quick Start

### Prerequisites
- Python 3.10+
- Node.js 18+

### 1. FastAPI Backend

```bash
cd cyber-dashboard/backend

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate       # Unix
# venv\Scripts\activate       # Windows

# Install dependencies
pip install -r requirements.txt

# Start the ASGI server
uvicorn server:app --host 0.0.0.0 --port 8000 --reload
```
API Documentation available instantly at `http://localhost:8000/docs`

### 2. React Frontend

```bash
cd cyber-dashboard

npm install

# Hook up local API
echo "VITE_API_URL=http://localhost:8000" > .env.development

npm run dev
```
Dashboard live at `http://localhost:5173`

### 3. Streamlit SOC Dashboard

```bash
cd IntrusionDetectionDashboard
python -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt
streamlit run app.py
```
App live at `http://localhost:8501`. 

*(Note: Data must be present in the project root, or the app engages full synthetic fallback mode).*

---

## ☁️ Deployment Specifications

### Deploy FastAPI Backend to Render
1. Map repository to Render Web Service. Base dir: `cyber-dashboard/backend`.
2. Build Command: `pip install -r requirements.txt`
3. Start Command: `uvicorn server:app --host 0.0.0.0 --port $PORT`
4. Define env vars: `DATA_SOURCE_URL` (Ngrok URL of your laptop) and `DATA_SECRET`.

### Deploy React Frontend to Vercel
1. Map repository to Vercel. Base dir: `cyber-dashboard`.
2. Framework: Vite. Build Command: `npm run build`.
3. Add Environment Variable `VITE_API_URL` pointing strictly to your Render domain.

---

## 📡 API Reference

*Base URL: `http://localhost:8000` (or Render domain)*

| Method | Endpoint | Internal Operation |
|---|---|---|
| `GET` | `/api/health` | Rapid liveness probe. Returns up-time and loaded ML cache count. |
| `POST` | `/api/train` | Triggers the `engine.py` pipeline. Pulls rows, fits scaler, trains classifiers. |
| `GET` | `/api/models` | Returns hyper-parameters and validation metrics (F1, Accuracy) for UI leaderboard. |
| `POST` | `/api/set-active/{model_name}` | Overrides the auto-selected best model memory ptr mechanism. |
| `POST` | `/api/predict` | Pushes N synthesized packets through the `.predict_proba()` of the active model. |
| `GET` | `/api/dashboard` | Aggregates all session stats (Total blocked, Threat Level matrix). |

---

## 🤝 Contributing

Contributions are welcome. Please open an issue before submitting massive architectural PRs.

1. Fork the repo.
2. Branch: `feature/your-insight`
3. Commit with detailed messages.
4. Pull Request.

---

<div align="center">

**Built by [Soubhagya Jain](https://github.com/SoubhagyaJain)**

*Securing the network one vector at a time.*

</div>
