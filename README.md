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
