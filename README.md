<div align="center">

# 🛡️ CyberSentinel AI

**Elite Real-Time Network Intrusion Detection & AI SOC Dashboard**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)](https://streamlit.io/)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=black)](https://scikit-learn.org/)
[![Vite](https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white)](https://vitejs.dev/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

> CyberSentinel AI is an end-to-end, production-ready Security Operations Center (SOC) platform. It bridges theoretical data science and practical cybersecurity by training, evaluating, and deploying multiple Machine Learning architectures to detect zero-day exploits and network intrusions in real-time, all while demystifying decisions through Explainable AI (XAI).

🌐 **LIVE DEPLOYMENTS (Vercel & Render)** | _Coming Soon / Add URLs here_

</div>

---

## 📖 Table of Contents

- [Executive Summary](#-executive-summary)
- [System Architecture](#-system-architecture)
- [Engineering The Data Pipeline: Overcoming Cloud Limits](#-engineering-the-data-pipeline)
- [Machine Learning & Model Operations (MLOps)](#-machine-learning--mlops)
  - [The Multi-Model Benchmark](#the-multi-model-benchmark)
  - [Explainable AI (XAI) & Feature Interpretability](#explainable-ai-xai--feature-interpretability)
- [Dual-Frontend Paradigm](#-dual-frontend-paradigm)
  - [1. React + Vite Cyber Dashboard](#1-react--vite-cyber-dashboard)
  - [2. Streamlit Advanced SOC Console](#2-streamlit-advanced-soc-console)
- [Tech Stack Overview](#-tech-stack-overview)
- [Getting Started (Local & Cloud Deployment)](#-getting-started)
- [Future Roadmap](#-future-roadmap)

---

## 🎯 Executive Summary

In enterprise environments, dropping packets aggressively without valid justification disrupts legitimate business functions. CyberSentinel was built not just to **detect** network anomalies like DoS, DDoS, Probing, and R2L attacks, but to **explain** why they were flagged.

**The core tenets of this platform:**
1. **Dynamic Model Auto-Selection:** Instead of relying on a hardcoded algorithm, the backend concurrently trains and evaluates multiple classifiers (Random Forest, Decision Tree, Naïve Bayes, XGBoost, MLP), dynamically promoting the architecture with the highest F1-Score to the live inference engine.
2. **Explainable AI (XAI):** Utilizing feature-importance matrices and SHAP-inspired logic, the platform translates raw numeric outputs into human-readable SOC narratives (e.g., _"Connection flagged due to unusual TCP Window Scale anomalies"_).
3. **Decoupled Edge Ingestion:** Cybersecurity CSV datasets frequently exceed 10GB+, instantly crashing free-tier cloud environments. CyberSentinel introduces a localized Ngrok-tunneled HTTP data server (`data_server.py`) that streams down-sampled, byte-casted rows sequentially to the cloud inference layer, bypassing PaaS disk limits.

---

## 🏗️ System Architecture

CyberSentinel operates as a multi-tier microservice ecosystem, decoupling heavy data processing, RESTful ML inference, and reactive frontend clients.

```mermaid
flowchart TD
    %% Local Ingestion Layer
    subgraph DataEdge ["Data Ingestion Edge (Local / On-Premise)"]
        RawData[(Massive CSV Datasets\n10GB+)]
        DS[data_server.py\nStreaming Server]
        RawData -- "Iterative Chunk Reading" --> DS
    end

    %% Cloud Inference Layer
    subgraph CloudBackend ["Cloud ML Inference Backend (FastAPI / Render)"]
        API[FastAPI Gateway]
        MLCore[Scikit-Learn & XGBoost Engine]
        State[In-Memory Threat State & Cache]
        
        DS -- "HTTPS Stream (Ngrok Tunneling)" --> API
        API <--> MLCore
        API <--> State
    end

    %% Cloud Clients
    subgraph Clients ["Frontend Consumers"]
        ReactUI[React + Vite Web Dashboard\n(Vercel)]
        StreamlitUI[Streamlit SOC Console\n(Direct Python UI)]
    end

    API -- "REST API (JSON / Webhooks)" --> ReactUI
    StreamlitUI -. "Optional: Standalone Execution Pipeline" .-> MLCore
```

---

## ⚡ Engineering the Data Pipeline

Processing network datasets (like CICIDS-2017 or NSL-KDD) natively on platforms like Docker/Render results in immediate `MemoryError` and Disk-Space quota breaches.

### 1. The Local Distributed SAN strategy
Instead of hosting 10GB CSV files in the cloud, `data_server.py` runs locally. It spins up a highly optimized `http.server` that reads `[dataset-part1...part4].csv` simultaneously, utilizing `itertools` to interleave raw data and stream exact row-counts over an Ngrok tunnel to the FastAPI backend.

### 2. Aggressive In-Memory Downcasting
Once chunked data arrives at the cloud backend, 64-bit precision is universally scrapped:
- `float64` → `float32`
- `int64` → `int32`
This seemingly simple step slashes RAM allocation footprint by ~50%, allowing matrix multiplication protocols (like MLP calculation) to survive low-tier cloud instances.

### 3. Dimensionality Reduction
Out of 80+ potential features in standard network captures, CyberSentinel tightly slices data down to the **15 most critical TCP/IP feature vectors** (e.g., `TCP_WIN_SCALE`, `FLOW_DURATION`, `PROTOCOL`, `DST_TOS`), discarding null fields and infinite ratios to prevent gradient overflow.

---

## 🧠 Machine Learning & MLOps

### The Multi-Model Benchmark

The `engine.py` pipeline orchestrates a concurrent train-test-evaluation sequence employing a standardized 70/30 split. The system races the following models against each other:

| Model Architecture | Purpose & Strengths |
| :--- | :--- |
| **Random Forest** (n=30) | High resistance to overfitting, produces robust baseline feature importance metrics. |
| **XGBoost** | Handles the massive class imbalances inherent in intrusion datasets via gradient-penalization. |
| **Decision Tree** (depth=4) | Ultra-fast inference with mathematically transparent branching rules. |
| **Multi-Layer Perceptron** | Deep learning representation capable of discovering severe non-linear relationships across OSI layers. |
| **Gaussian Naïve Bayes** | Evaluates independent statistical probabilities. Extremely lightweight. |

**Auto-Promotion:** The system generates detailed evaluation matrix metrics (Accuracy, Precision, Recall, and ROC-AUC). Because False Negatives in NIDS are catastrophic, the model achieving the highest **Weighted F1-Score** is seamlessly elected to "Active Status" and begins proxying all live `/api/predict` traffic.

### Explainable AI (XAI) & Feature Interpretability

CyberSentinel implements dynamic global and local feature importance transparency.

* **Global Level:** Displays high-level radar and bar charts representing exactly which packet features dictate the model’s splits (e.g. tracking `TCP_WIN_SCALE_OUT` as the primary differentiator for a SYN flood).
* **Local Level (Per-Packet):** For every simulated packet blocked, the engine calculates the probability distribution across all possible labels, translating mathematical feature weights into actionable English text like: _"The TCP negotiation parameters diverged from standard handshake behavior."_

---

## 🖥️ Dual-Frontend Paradigm

CyberSentinel exposes the ML engine through two distinct views depending on user personas.

### 1. React + Vite Cyber Dashboard (`/cyber-dashboard`)
**Intended for:** C-Suite, IT Display Screens, and High-Level Overviews.
* Built entirely in React 18 and TailwindCSS 4.
* Uses **Framer Motion** for liquid-smooth HUD animations and **Recharts** for live telemetry.
* Hits the FastAPI `/api/dashboard`, `/api/system`, and `/api/predict` endpoints via automated polling interval loops.

### 2. Streamlit Advanced SOC Console (`/IntrusionDetectionDashboard`)
**Intended for:** Security Analysts, Data Scientists, and Threat Hunters.
* A massive fully-native pure-Python dashboard.
* Contains interactive Plotly Confusion Matrices and ROC-AUC curve rendering.
* Manages the "Model Sandbox" where users can surgically load specific `joblib` artifacts and alter hyperparameters on the fly.
* Displays the "Threat Intelligence" detailed logs with Explainable AI reasoning strings under the 'Intelligence' Tab.

---

## 🛠️ Tech Stack Overview

| Category | Technology |
|---|---|
| **Core ML Engine** | Scikit-Learn, XGBoost, Pandas, NumPy |
| **REST Infrastructure** | FastAPI, Uvicorn, Python 3.10+ |
| **React Interface** | React 18, Vite, TailwindCSS, Framer Motion, Recharts |
| **Analyst Interface** | Streamlit 1.25+, Plotly 5.15+ |
| **DevOps / Hosting** | Render (Backend), Vercel (React), Ngrok (Data Tunnel) |

---

## 🏁 Getting Started

### Prerequisites
* Python `3.10+` Minimum
* Node.js `18+` Minimum

### Phase 1: Local Data Server (Ingestion source)
If using the massively large datasets locally, drop them into the project root (`dataset-partX.csv`).
```bash
python data_server.py
```
*(Optional) To expose to the cloud backend over WAN:*
```bash
ngrok http 7860
```

### Phase 2: FastAPI Backend Engine
```bash
cd cyber-dashboard/backend
python -m venv venv
source venv/bin/activate  # Unix
venv\Scripts\activate     # Windows

pip install -r requirements.txt

# Start the highly optimized Uvicorn worker
uvicorn server:app --host 0.0.0.0 --port 8000 --reload
```
_Visit `http://localhost:8000/docs` to see the live Swagger OpenAPI specification._

### Phase 3: The React Frontend
```bash
cd cyber-dashboard
npm install
# Route queries to localhost
echo "VITE_API_URL=http://localhost:8000" > .env.development
npm run dev
```

### Alternative: The Streamlit SOC Console
Instead of the React/FastAPI combo, you can run the monolithic analytical dashboard directly:
```bash
cd IntrusionDetectionDashboard
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
streamlit run app.py
```

---

## 🚀 Future Roadmap

* **Live PCAP Sniffing:** Replacing simulated CSV inference with direct integration of `pyshark` or `scapy` to evaluate packets off physical Network Interface Cards (NICs), transforming the codebase from a Dashboard into an active Next-Gen Firewall (NGFW). 
* **Deep Sequence Models:** Upgrading from stateless Scikit-Learn evaluation to PyTorch LSTMs, permitting the system to remember long-tail connection contexts identifying low-and-slow DoS attacks like "Slowloris".
* **Kafka & PostgreSQL Integration:** Migrating from in-process FastAPI memory states to high-throughput Apache Kafka streaming with persistent distributed PostgreSQL logging.

---

<div align="center">
<b>Designed & Architected by <a href="https://github.com/SoubhagyaJain">Soubhagya Jain</a></b>
<br/>
<i>Securing the Zero-Trust network layer, one packet vector at a time.</i>
</div>
