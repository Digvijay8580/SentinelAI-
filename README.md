# SentinelAI Enterprise Edition 🛡️  

**Production-Grade Insider Threat Detection Platform**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](#license)  
[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)  
[![Node.js](https://img.shields.io/badge/Node.js-18+-green.svg)](https://nodejs.org/)  
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-14+-blue.svg)](https://www.postgresql.org/)  
[![FastAPI](https://img.shields.io/badge/FastAPI-Backend-teal.svg)](https://fastapi.tiangolo.com/)  
[![React](https://img.shields.io/badge/React-SOC%20Dashboard-61dafb.svg)](https://react.dev/)

SentinelAI is an **enterprise-grade security monitoring system** designed to detect **insider threats, suspicious workstation behavior, and policy violations in real time**—helping organizations identify malicious insider activity **before data exfiltration occurs**.

---

## ✨ Key Capabilities

- **Windows Endpoint Agents**: lightweight, behavior-focused monitoring (no full disk scans)
- **FastAPI Backend**: secure APIs, authentication, RBAC, and event ingestion
- **PostgreSQL Analytics Store**: structured storage for events, violations, alerts, and reports
- **SOC Dashboard (React + Vite)**: overview, employees, trends, alerts, reports, admin panel
- **Real-Time WebSocket Alerts**: instant risk updates and incident notifications
- **Automated Risk Scoring & Threat Detection**: weighted violation scoring with **daily decay**
- **Event Deduplication**: same event within **5 minutes** counted once
- **Token Separation**: distinct validation paths for **user tokens** vs **agent tokens**

---

## 🧩 System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    SentinelAI Architecture                    │
├──────────────┬───────────────────────┬────────────────────────┤
│ Endpoint     │ Central Backend       │ SOC Dashboard          │
│ Agents       │ (FastAPI + PostgreSQL)│ (React + Vite)         │
│ (Windows)    │                       │                        │
│              │ Auth + RBAC           │ Overview               │
│ Detect:      │ Risk Engine           │ Employees              │
│ • USB usage  │ Violation Database    │ Threat Trends          │
│ • File copy  │ Alert System          │ Alerts                 │
│ • Apps       │ Email Service         │ Reports                │
│ • Ports      │ Report Generator      │ Admin Panel            │
│ • Logins     │ WebSocket Broadcast   │                        │
└──────────────┴───────────────────────┴────────────────────────┘
```

---

## 📐 Key Design Principles

| Principle | Implementation |
|---|---|
| **Event-Driven** | Agents send events only when violations occur |
| **Risk Decay** | Risk scores automatically decay by **20% per day** |
| **Dedup Protection** | Same event within **5 minutes** counted once |
| **Token Separation** | Agent tokens and user tokens use different validation paths |
| **RBAC Security** | **Admin → Analyst → Viewer** permission hierarchy |
| **Lightweight Agents** | Only monitors configured behaviors; no full disk scans |

---

## 📁 Project Structure

```
sentinelai/
│
├── agent/
│   ├── agent.py
│   ├── requirements.txt
│   └── sentinel_agent.spec
│
├── backend/
│   ├── main.py
│   ├── database.py
│   ├── requirements.txt
│   │
│   ├── auth/
│   │   └── jwt_handler.py
│   │
│   ├── models/
│   │   └── models.py
│   │
│   ├── routes/
│   │   ├── auth.py
│   │   ├── employees.py
│   │   ├── events.py
│   │   ├── violations.py
│   │   ├── alerts.py
│   │   ├── reports.py
│   │   ├── admin.py
│   │   └── websocket.py
│   │
│   └── services/
│       ├── risk_engine.py
│       ├── websocket_manager.py
│       ├── email_service.py
│       ├── report_service.py
│       └── scheduler.py
│
├── dashboard/
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       └── App.jsx
│
├── docker/
│   ├── docker-compose.yml
│   └── Dockerfile.backend
│
├── .env.example
└── README.md
```

---

## 🚀 Quick Start Deployment

### ✅ Prerequisites

- **Python 3.11+**
- **Node.js 18+**
- **PostgreSQL 14+**
- **Docker (optional)**
- **Windows system** (for the endpoint agent)

---

### 1) 🗄️ PostgreSQL Setup

**Docker (recommended):**
```bash
docker run -d \
  --name sentinelai-pg \
  -e POSTGRES_USER=sentinel \
  -e POSTGRES_PASSWORD=sentinel_pass \
  -e POSTGRES_DB=sentinelai \
  -p 5432:5432 \
  postgres:16-alpine
```

**Native PostgreSQL:**
```sql
CREATE USER sentinel WITH PASSWORD 'sentinel_pass';
CREATE DATABASE sentinelai OWNER sentinel;
GRANT ALL PRIVILEGES ON DATABASE sentinelai TO sentinel;
```

---

### 2) 🧠 Backend Setup

```bash
cd sentinelai

# Create virtual environment
python -m venv venv

# Activate environment
# Linux / macOS
source venv/bin/activate
# Windows
venv\Scripts\activate

# Install dependencies
pip install -r backend/requirements.txt

# Configure environment
cp .env.example .env

# Generate secure JWT key
python -c "import secrets; print(secrets.token_hex(32))"

# Add to .env
# JWT_SECRET_KEY=<generated_key>

# Start backend
# Development
uvicorn backend.main:app --host 0.0.0.0 --port 8000 --reload

# Production
uvicorn backend.main:app --host 0.0.0.0 --port 8000 --workers 4
```

---

### 3) 👤 Seed Admin Account

Run once after backend startup:

```bash
curl -X POST http://localhost:8000/api/auth/seed-admin
```

**Default credentials:**
- **Username:** `admin`  
- **Password:** `SentinelAdmin2024!`  

> ⚠️ **Change these immediately in production.**

---

### 4) 🖥️ SOC Dashboard Setup

```bash
cd sentinelai/dashboard

# Install dependencies
npm install

# Run dev server
npm run dev
```

- Dev: **http://localhost:5173**

**Production build:**
```bash
npm run build
```
Serve the `dist/` folder with Nginx (or any static server).

---

### 5) 🪪 Windows Endpoint Agent

```bash
# Install dependencies
pip install -r agent/requirements.txt

# Configure environment variables
export SENTINEL_BACKEND=http://192.168.1.100:8000
export AGENT_NAME=WORKSTATION-ALICE

# Run agent
python agent/agent.py
```

**Build executable (PyInstaller):**
```bash
pip install pyinstaller
cd agent
pyinstaller sentinel_agent.spec --clean
```
Output: `dist/SentinelAgent.exe`

**Auto-start on Windows:**
1. `Win + R` → `shell:startup`  
2. Add a shortcut to `SentinelAgent.exe`

> ⚠️ Run the agent with **administrator privileges** for full monitoring capabilities.

---

## 🔐 JWT Token System

SentinelAI uses **two token types**:

| Token Type | Subject | Expiry | Purpose |
|---|---|---|---|
| **User Token** | `username` | 60 minutes | Dashboard login |
| **Agent Token** | `employee_id` | 30 days | Agent API access |

**Example agent token payload:**
```json
{
  "sub": "42",
  "name": "WORKSTATION-01",
  "type": "agent",
  "exp": 1735689600,
  "iat": 1733097600
}
```

Agents automatically **re-register** if their token expires.

---

## 🧾 RBAC Permissions

| Action | Admin | Analyst | Viewer |
|---|---|---|---|
| View employees | ✅ | ✅ | ✅ |
| View alerts | ✅ | ✅ | ✅ |
| Resolve alerts | ✅ | ✅ | ❌ |
| Send warnings | ✅ | ✅ | ❌ |
| Disable USB | ✅ | ❌ | ❌ |
| Export reports | ✅ | ❌ | ❌ |
| Create users | ✅ | ❌ | ❌ |

---

## 📊 Risk Scoring System

### Violation Weights
| Event | Score |
|---|---:|
| USB Insertion | +40 |
| Bulk File Copy | +50 |
| Night Login | +20 |
| Unauthorized App | +50 |
| Keylogger Detection | +80 |
| Suspicious Port | +60 |

### Risk Levels
| Score | Level |
|---:|---|
| 0–49 | Low |
| 50–119 | Medium |
| 120+ | High |

### Risk Decay
If no violations occur, **risk score = risk score × 0.80 per day** (applied by the background scheduler).

---

## 🧪 Testing the System

**Submit a test event:**
```bash
curl -X POST http://localhost:8000/api/events/submit \
  -H "Authorization: Bearer <AGENT_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"event_type":"usb_insertion","metadata":{"drive":"E:\\"}}'
```

---

## 🔴 Real-Time WebSocket

- **Endpoint:** `WS /ws/events?token=<jwt>`
- **Streamed events:** `risk_update`, `warning_sent`, `pong`

---

## 🔒 Security Recommendations (Production)

- Change **`JWT_SECRET_KEY`**
- Change **PostgreSQL credentials**
- Deploy behind **Nginx with HTTPS**
- Use **App Passwords** for email alerts
- Restrict **firewall access** to backend APIs
- Ensure agents run with **administrator privileges**

---

## 🧯 Troubleshooting

| Problem | Solution |
|---|---|
| Backend fails to start | Check PostgreSQL connection and `.env` configuration |
| Agent receives `401` | Re-register agent token |
| WebSocket disconnects | Verify CORS configuration |
| USB disable not working | Run agent as administrator |
| Email alerts missing | Verify SMTP settings |
| Risk score not decaying | Ensure scheduler is running |

---

## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a pull request with clear descriptions, tests where applicable, and documentation updates.

---

## 📄 License

This project is licensed under the **MIT License**. See the `LICENSE` file for details.

---

**SentinelAI — Enterprise-grade insider threat detection platform built for SOC teams and security engineers.**
