# Retail Infrastructure — CI/CD & Cloud Delivery (GCP Dev / GKE)

Infrastructure and delivery automation for the **Retail Platform**.

This repository focuses on **Continuous Delivery (CD)** and deployment workflows, while application build and test pipelines (CI) live in the individual application repositories.

**Backend Repo:** https://github.com/suissitakwa/retail  
**UI Repo:** https://github.com/suissitakwa/retail-ui  
**Portfolio:** https://portfolio-showcase--suissitakwa.replit.app  

---

## Purpose

This repository demonstrates how I design and operate **production-style delivery pipelines**:
- clear separation between **CI (GitHub Actions)** and **CD (Jenkins)**
- repeatable deployments to cloud environments
- environment-aware configuration (dev today, promotable to prod)
- secure handling of secrets and credentials

---

## Secrets Management

`k8s/dev/secrets.yaml` is listed in `.gitignore` and **must never be committed with real values**.

Before deploying, apply the real secret to the cluster manually or via Jenkins:

```bash
# Option A — apply manually (one-time or after rotation)
kubectl apply -f k8s/dev/secrets.yaml   # after filling in real values locally

# Option B — let Jenkins inject secrets from Jenkins Credentials
# The Jenkinsfile reads STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, and JWT_SECRET_KEY
# from Jenkins credentials and writes a real secrets.yaml at deploy time.
```

The secret must exist in the `retail-dev` namespace before the backend pod starts.
Required keys: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `JWT_SECRET_KEY`.

> Current deployment target: **Google Kubernetes Engine (GKE) — Dev environment**

---

## CI/CD Overview

```text
┌──────────────────────────────────────────────────────────────────────┐
│                           Retail Platform (Multi-Repo)               │
│  Backend: suissitakwa/retail     UI: suissitakwa/retail-ui           │
│  CD/Infra: suissitakwa/retail-infra                                  │
└──────────────────────────────────────────────────────────────────────┘

Developer Workflow
------------------
   git push / PR
        │
        ▼
┌───────────────────────────────┐
│ GitHub Actions (CI)           │
│ - Build                       │
│ - Unit tests                  │
│ - Checks/Lint (optional)      │
│ - Build Docker image          │
│ - Push image to registry      │
└───────────────────────────────┘
        │
        │  (Trigger Jenkins job manually OR via webhook)
        ▼
┌───────────────────────────────┐
│ Jenkins (CD)                  │
│ - Pull image tag/commit SHA   │
│ - Deploy to GKE (DEV)         │
│ - Rollout verification        │
│ - Smoke check / health check  │
└───────────────────────────────┘
        │
        ▼
┌───────────────────────────────┐
│ Google Cloud (DEV)            │
│ - GKE workloads (Deploy/Svc)  │
│ - Secrets/Config per env      │
│ - Prometheus metrics scraping │
└───────────────────────────────┘
```

---

## Autoscaling

`k8s/dev/backend-hpa.yaml` defines a `HorizontalPodAutoscaler` (`autoscaling/v2`) targeting the `retail-backend` Deployment: `minReplicas: 2`, `maxReplicas: 4`, scaling on CPU utilization (70% of the 250m request). It relies on GKE's built-in `metrics-server` (present by default on both Standard and Autopilot clusters — no extra manifest needed). Applied by Jenkins in the "Deploy Application" stage alongside `backend.yaml`.

Raising `minReplicas` from 1 to 2 also removes the brief availability gap a single-replica rolling update otherwise causes. Memory is intentionally not used as a scaling metric yet — the backend's 512Mi memory request is already close to the JVM's own configured ceiling (see `retail/Dockerfile`), so a memory-based trigger needs that request right-sized first.

The `retail/` repo's k6 load test (`loadtest/checkout-flow.js`) was run locally against Docker Postgres/Redis, not against this cluster, so its recorded numbers don't map directly to the 70% CPU threshold above — that figure is an industry-default starting point. Re-running k6 against the live ingress while watching `kubectl get hpa -n retail-dev -w` is the real verification that ties load testing to autoscaling behavior.
