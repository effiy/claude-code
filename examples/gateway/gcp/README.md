---
title: Claude Gateway on Google Cloud
tags:
  - claude
  - gateway
  - gcp
  - vertex-ai
  - agent-platform
  - cloud-run
  - gke
  - cloud-sql
  - secret-manager
category: deployment
created: 2025-06-15
updated: 2026-10-20
source: claude-code/examples/gateway/gcp
type: reference
status: stable
lifecycle: production
---

# Claude Gateway on Google Cloud

## 💡 概述

Reference deployment artifacts for running Claude Gateway on GCP with Agent
Platform (formerly Vertex AI) as the upstream: Cloud Run or GKE, Cloud SQL for
PostgreSQL, Secret Manager, and service-account auth to Agent Platform.

These files are provided as a working example rather than a supported production
deployment. Adapt them to your own environment.

---

## ⚡ 功能

- End-to-end setup via `setup.sh` script using `gcloud`
- Docker-based runtime image for the `claude gateway` binary
- GCP-shaped gateway configuration template (Agent Platform upstream, Google Workspace IdP)
- Terraform provisioning of the full architecture (two-pass apply)

---

## 📊 指标

「待补充」

---

## 🏛️ 设计

The deployment architecture targets either **Cloud Run** or **GKE** for compute,
backed by **Cloud SQL for PostgreSQL** for state, **Secret Manager** for
credentials, and **service-account auth** to Agent Platform (Vertex AI) for the
upstream model access.

---

## 📁 目录

| File | Purpose |
|---|---|
| `setup.sh` | Scripts the walkthrough end to end via `gcloud` |
| `Dockerfile` | Runtime image for the `claude gateway` binary |
| `gateway.yaml.example` | Gateway config template, GCP-shaped (Agent Platform upstream, Google Workspace IdP) |
| `terraform/` | Provisions the full architecture (two-pass apply — see `terraform/README.md`) |

---

## 🔗 资源

- **Walkthrough**: https://code.claude.com/docs/en/claude-apps-gateway-on-gcp

---

## 🚀 快速开始

「待补充」

---

## ⚙️ 技术栈

- **Cloud**: Google Cloud Platform (GCP)
- **Compute**: Cloud Run, GKE
- **Database**: Cloud SQL for PostgreSQL
- **Secrets**: Secret Manager
- **Auth**: Service account auth to Agent Platform
- **Upstream**: Agent Platform (formerly Vertex AI)
- **IdP**: Google Workspace
- **IaC**: Terraform (see `terraform/` subdirectory)
- **Container**: Docker