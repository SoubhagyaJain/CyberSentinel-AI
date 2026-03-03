<div align="center">

# CyberSentinel

[![Build Status](https://img.shields.io/badge/Build-Passing-brightgreen.svg)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue.svg)](#)

</div>

---

## TL;DR

*   **What it is**: An end-to-end, real-time Network Intrusion Detection System (NIDS) and AI-powered Security Operations Center (SOC) dashboard.
*   **Who it’s for**: Security Analysts, Threat Hunters, and SOC teams managing enterprise network security.
*   **Why it matters**: It moves beyond "black-box" ML detections by training multiple models concurrently and using Explainable AI (XAI) to provide human-readable justification for every blocked packet.

## Demo

![Demo Placeholder](https://via.placeholder.com/800x400?text=Animated+GIF+or+Screenshot+Placeholder)

## Why this exists

Enterprise environments generate massive amounts of network traffic, making manual threat detection impossible. While traditional ML models can flag anomalies, they often fail to explain *why* an action was taken, leading to disruptive false positives. CyberSentinel fixes this by orchestrating a multi-model pipeline that dynamically selects the best classifier and pairs it with SHAP-based feature interpretability. Analysts aren't just told a packet is malicious—they are shown exactly which TCP handshake anomaly or timing irregularity triggered the alert.

## Features

**Core Analysis**

*   **Dynamic Model Auto-Selection**: Concurrently trains Random Forest, Decision Tree, Naïve Bayes, XGBoost, and MLP architectures, auto-promoting the highest F1-Score model.
*   **Explainable AI (XAI)**: Generates human-readable narratives (e.g., "Flagged due to anomalous TCP window scaling") for every detection.
*   **Local Data Streaming**: Bypasses cloud compute limits by streaming 10GB+ CSV network datasets directly from your laptop to the cloud inference engine via Ngrok.

**Frontend Experience**

*   **Dual Interfaces**: Includes a fast, reactive Vite+React C-Suite dashboard and a deeply analytical Python Streamlit SOC console.
*   **Live Telemetry**: Real-time traffic processing, unique IP tracking, and dynamic threat level calculations (Low, Moderate, High).

## Architecture (High Level)

```text
Local Environment                     Cloud Environment (Render/Vercel)
-----------------                     ---------------------------------

[10GB+ CSV Datasets]                  [FastAPI Gateway] <---> [React UI]
         |                                   |  ^                 ^
         v                                   v  |                 |
[data_server.py] -- Ngrok HTTPS --> [ML Inference Engine]         |
(Streams chunks)                    (Scikit-learn/XGBoost)        |
                                             |                    |
                                             v                    |
                                     [Threat State Cache] <-------+
```

*   **Local Ingestion**: `data_server.py` reads massive CSV datasets and streams chunks locally to avoid remote memory constraints.
*   **Inference Engine**: FastAPI receives data, trains models, selects the best performer, and executes real-time `.predict_proba()` against incoming streams.
*   **Frontend Consumers**: React dashboard polls for live system updates while the optional Streamlit app provides deep analytical sandbox experiences.

## Tech Stack

*   **Frontend**: React 18, Vite, TailwindCSS, Framer Motion, Recharts
*   **Analytics UI**: Streamlit 1.25+, Plotly 5.15+
*   **Backend**: FastAPI, Uvicorn, Python 3.10+
*   **ML Engine**: Scikit-Learn, XGBoost, Pandas, NumPy, SHAP
*   **Infra/Deploy**: Render (Backend), Vercel (Frontend frontend), Ngrok (Data Tunnel)

## Project Structure

```text
CyberSentinel/
├── IntrusionDetectionDashboard/   # Standalone Streamlit SOC UI & deep analytics
├── cyber-dashboard/               # React UI & FastAPI Backend workspace
│   ├── src/                       # React frontend source code
│   └── backend/                   # FastAPI inference engine and ML logic
├── data_server.py                 # Local HTTP streaming server for large CSVs
└── train_model.py                 # Standalone CLI model trainer
```

## Quickstart

The fastest path to run the API and Streamlit UI locally.

```bash
# Clone the repository
git clone https://github.com/SoubhagyaJain/CyberSentinel-AI.git
cd CyberSentinel-AI

# 1. Start the Data Server (Requires dataset-partX.csv in root)
python data_server.py &

# 2. Start the FastAPI Backend
cd cyber-dashboard/backend
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
uvicorn server:app --host 0.0.0.0 --port 8000 &

# 3. Start the Streamlit SOC UI
cd ../../IntrusionDetectionDashboard
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
streamlit run app.py
```

## Installation

### 1. Requirements

*   Python 3.10+
*   Node.js 18+ (if running React frontend)
*   At least 8GB RAM recommended for data processing.

### 2. Backend Setup

```bash
cd cyber-dashboard/backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. React Frontend Setup

```bash
cd cyber-dashboard
npm install
echo "VITE_API_URL=http://localhost:8000" > .env.development
```

## Configuration

The backend data stream requires configuration if you are running the API remotely but serving data locally.

| Variable | Description | Example / Default |
| :--- | :--- | :--- |
| `DATA_SOURCE_URL` | The Ngrok or external URL pointing to your local `data_server.py`. | `https://xxxx.ngrok-free.app` |
| `DATA_SECRET` | Secret token to authenticate the data stream requests. | `cybersentinel-local-2024` |
| `VITE_API_URL` | (Frontend) The base URL for the FastAPI backend. | `http://localhost:8000` |

## Usage

**Start Local Data Server**
Run from the repository root:
```bash
python data_server.py
```

**Start FastAPI Backend**
Run from `cyber-dashboard/backend`:
```bash
uvicorn server:app --reload
```
API Documentation: `http://localhost:8000/docs`

**Start React UI**
Run from `cyber-dashboard`:
```bash
npm run dev
```

**Trigger Model Training via API**
```bash
curl -X POST http://localhost:8000/api/train \
  -H "Content-Type: application/json" \
  -d '{"sample_size": 100000}'
```

## Testing

*Testing configuration is not currently instrumented in the repository. Please see Assumptions & TODOs.*

## Quality

Ensure code formatting before pushing:

```bash
# Python (run from root)
flake8 .
black .

# React (run from cyber-dashboard)
npm run lint
```

## Deployment

**Cloud Backend (Render)**

1.  Connect your repository to Render as a Web Service. Root dir: `cyber-dashboard/backend`.
2.  Build Command: `pip install -r requirements.txt`
3.  Start Command: `uvicorn server:app --host 0.0.0.0 --port $PORT`
4.  Set `DATA_SOURCE_URL` and `DATA_SECRET` environment variables.

**Cloud Frontend (Vercel)**

1.  Connect your repository to Vercel. Framework preset: Vite. Root dir: `cyber-dashboard`.
2.  Set Environment Variable `VITE_API_URL` to your Render API domain.

**Local Data Edge**

1. Run `data_server.py` on your laptop where the large CSVs reside.
2. Expose the port: `ngrok http 7860`. Copy the HTTPS link to your Render env vars.

## Security

*   **Data Authorization**: `data_server.py` utilizes basic Bearer-token authorization to prevent public scraping of your local CSV datasets. Always change the default `DATA_SECRET` in a production scenario.
*   **State Management**: Current state is held in-memory via FastAPI `app.state`. This is suitable for demonstration but volatile in multi-worker production.

## Troubleshooting

1.  **Issue: `MemoryError` when training models.**
    *   *Fix*: Reduce the `sample_size` in the training request (e.g., down to `10000`). Ensure `float64` downcasting is active in `ml.data.py`.
2.  **Issue: API endpoints return 500 when fetching data.**
    *   *Fix*: Ensure `data_server.py` is running locally, your Ngrok tunnel is active, and `DATA_SOURCE_URL` is set correctly in the backend environment.
3.  **Issue: Streamlit UI says "Model Offline".**
    *   *Fix*: Click the "Train All Models" button in the sidebar to populate the `.joblib` model cache.
4.  **Issue: React Dashboard shows no data.**
    *   *Fix*: Verify `VITE_API_URL` exactly matches the running FastAPI instance domain, with no trailing slashes.

## Roadmap

*   Deploy real-time packet sniffing (`pyshark`/`scapy`) to replace CSV simulations.
*   Transition in-memory state architecture to PostgreSQL and Redis.
*   Integrate Deep Sequence Models (LSTMs) for multi-packet temporal attack detection.
*   Implement JWT-based Role Based Access Control (RBAC).

## Contributing

1.  Fork the repository.
2.  Create a feature branch (`git checkout -b feature/AddAwesomeFeature`).
3.  Commit your changes (`git commit -m 'Add AwesomeFeature'`).
4.  Push to the branch (`git push origin feature/AddAwesomeFeature`).
5.  Open a Pull Request.

## License

Distributed under the MIT License. See `LICENSE` for more information.

## Assumptions & TODOs

- [x] Assume `dataset-partX.csv` files are manually placed in root directory.
- [ ] TODO: Configure automated Python `pytest` suites for the FastAPI backend.
- [ ] TODO: Configure Cypress or Jest tests for the React frontend.
- [ ] TODO: Add GitHub Actions CI/CD workflows for automated linting and testing.
- [ ] TODO: Add the actual GIFs/Screenshots into the repository assets folder and update the Demo placeholder link.
