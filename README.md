# Containerized Three-Tier Application

A full-stack **Task Manager** application built with a three-tier architecture — containerized with Docker and orchestrated locally using Kubernetes (KinD).

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 · Vite · Axios · React Router |
| Backend | FastAPI · SQLAlchemy · PyMySQL · Uvicorn |
| Database | MySQL 8 (Docker Hub) |
| Containerization | Docker · Docker Hub |
| Orchestration | Kubernetes · KinD |

---

## Architecture

```
Browser (localhost:3000)
        │
        ▼
┌─────────────────┐
│  React Frontend │  NodePort :30080 → :3000
│  nginx:alpine   │
└────────┬────────┘
         │ proxy /api/
         ▼
┌─────────────────┐
│ FastAPI Backend │  ClusterIP :8000
│ python:3.11     │
└────────┬────────┘
         │ SQLAlchemy
         ▼
┌─────────────────┐
│   MySQL 8       │  ClusterIP :3306
│   PVC backed    │
└─────────────────┘
```

**KinD Cluster:** 1 Control Plane · 2 Worker Nodes

---

## Project Structure

```
├── docker-compose.yml          # Local dev
├── kind-config.yaml            # KinD cluster config
├── .env.example
├── frontend/
│   ├── Dockerfile              # Multi-stage: node → nginx
│   ├── nginx.conf
│   └── src/
│       ├── components/
│       ├── pages/
│       ├── services/
│       ├── hooks/
│       ├── utils/
│       └── constants/
├── backend/
│   ├── Dockerfile
│   └── app/
│       ├── main.py
│       ├── core/               # DB + config
│       ├── api/v1/endpoints/
│       ├── models/
│       ├── schemas/
│       └── crud/
└── k8s/
    ├── namespace.yml
    ├── frontend/               # deployment + NodePort service
    ├── backend/                # deployment + ClusterIP + configmap
    └── mysql/                  # deployment + ClusterIP + PV + PVC + secret
```

---

## API Endpoints

Base URL: `/api/v1`

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Health check |
| `GET` | `/items` | Get all tasks |
| `POST` | `/items` | Create task |
| `DELETE` | `/items/{id}` | Delete task |

---

## Getting Started

### Prerequisites

- Docker
- kubectl
- KinD

### 1. Create KinD Cluster

```bash
kind create cluster --config kind-config.yaml --name task-manager
```

### 2. Apply Kubernetes Manifests

```bash
kubectl apply -f k8s/namespace.yml
kubectl apply -f k8s/mysql/
kubectl apply -f k8s/backend/
kubectl apply -f k8s/frontend/
```

### 3. Verify Pods

```bash
kubectl get all -n task-manager
```

All pods should be `Running`.

### 4. Access App

```
http://localhost:3000
```

---

## Docker Hub Images

| Service | Image |
|---|---|
| Frontend | `arshadkhan007/3-tier-application:frontend-v1` |
| Backend | `arshadkhan007/3-tier-application:backend-v1` |
| Database | `mysql:8` (official) |

---

## Local Dev (Docker Compose)

```bash
docker-compose up
```

---

## Environment Variables

### Backend

| Variable | Source | Description |
|---|---|---|
| `DB_HOST` | ConfigMap | MySQL service name |
| `DB_PORT` | ConfigMap | MySQL port |
| `DB_USER` | ConfigMap | DB username |
| `DB_NAME` | Secret | Database name |
| `DB_PASSWORD` | Secret | DB password |

---

## K8s Best Practices Applied

- All resources scoped to a named namespace
- Secrets for sensitive data (base64 encoded)
- ConfigMap for non-sensitive config
- Resource requests + limits on every pod
- Liveness + Readiness probes defined
- ClusterIP for internal services
- NodePort only for frontend external access
- PV + PVC for MySQL data persistence
- `imagePullPolicy: Always` for latest image pulls
- Multi-stage Docker builds for minimal image size
- Non-root nginx serving React build
