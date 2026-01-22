# Retail Infrastructure — CI/CD & Cloud Delivery (GCP Dev)

Infrastructure and delivery automation for the **Retail Platform**.

This repository focuses on **Continuous Delivery (CD)** and deployment workflows, while application build and test pipelines (CI) live in the individual application repositories.

**Backend Repo:** https://github.com/suissitakwa/retail  
**UI Repo:** https://github.com/suissitakwa/retail-ui  
**Portfolio:** https://portfolio-showcase--suissitakwa.replit.app  

---

## Purpose

This repository demonstrates how I design and operate **production-style delivery pipelines**:
- clean separation between CI and CD
- repeatable deployments to cloud environments
- environment-aware configuration (dev today, promotable to prod)
- secure handling of secrets and credentials

> The current deployment target is a **development environment on Google Cloud**.  
> The pipeline is intentionally designed to be extensible to staging and production.

---

## High-Level Architecture

```text
Developer Push
      ↓
GitHub Actions (CI)
- build
- test
- package artifact / image
      ↓
Jenkins (CD)
- deploy to Google Cloud (dev)
- rollout verification
