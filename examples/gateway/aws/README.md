---
title: Claude Apps Gateway on AWS
tags:
  - claude
  - gateway
  - aws
  - bedrock
  - ecs
  - eks
  - rds
  - terraform
category: deployment
created: 2025-06-15
updated: 2026-10-20
source: claude-code/examples/gateway/aws
type: reference
status: stable
lifecycle: production
---

# Claude Apps Gateway on AWS

## 💡 概述

Reference deployment artifacts for running Claude apps gateway on AWS with
Amazon Bedrock as the upstream: ECS on Fargate or EKS, Amazon RDS for
PostgreSQL, AWS Secrets Manager, and IAM-role auth to Bedrock.

These files are provided as a working example rather than a supported production
deployment. Adapt them to your own environment.

---

## ⚡ 功能

- End-to-end setup via `setup.sh` script using the `aws` CLI
- Docker-based runtime image for the `claude gateway` binary
- AWS-shaped gateway configuration template (Bedrock upstream, Okta IdP)
- Terraform provisioning of the full architecture (two-pass apply)

---

## 📊 指标

「待补充」

---

## 🏛️ 设计

The deployment architecture targets either **ECS on Fargate** or **EKS** for
compute, backed by **Amazon RDS for PostgreSQL** for state, **AWS Secrets Manager**
for credentials, and **IAM-role auth** to Amazon Bedrock for the upstream
model access.

---

## 📁 目录

| File | Purpose |
|---|---|
| `setup.sh` | Scripts the walkthrough end to end via the `aws` CLI |
| `Dockerfile` | Runtime image for the `claude gateway` binary (bakes in `gateway.yaml`) |
| `gateway.yaml.example` | Gateway config template, AWS-shaped (Bedrock upstream, Okta IdP) |
| `terraform/` | Provisions the full architecture (two-pass apply — see `terraform/README.md`) |

---

## 🔗 资源

- **Walkthrough**: https://code.claude.com/docs/en/claude-apps-gateway-on-aws
- **Related**: AWS-maintained samples for various customer environments at
  https://github.com/aws-samples/anthropic-on-aws/tree/main/claude-apps-gateway

---

## 🚀 快速开始

「待补充」

---

## ⚙️ 技术栈

- **Cloud**: AWS
- **Compute**: ECS on Fargate, EKS
- **Database**: Amazon RDS for PostgreSQL
- **Secrets**: AWS Secrets Manager
- **Auth**: IAM roles to Amazon Bedrock
- **IaC**: Terraform (see `terraform/` subdirectory)
- **Container**: Docker