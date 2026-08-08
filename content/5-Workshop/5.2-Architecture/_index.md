---
title: "Architecture"
date: 2026-08-05
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Overall architecture

The system runs in the `ap-southeast-1` (Singapore) Region inside the default VPC. An Ubuntu `t3.medium` EC2 instance is placed in a public subnet and receives HTTP requests through its public IPv4 address. Docker forwards host port `80` to Kotaemon port `7860`.

![RAG Chat deployment architecture on AWS](/images/2-Proposal/platform_architecture.png)

## Access and data flows

|Flow|Description|
|-|-|
|Web access|Browser → public IPv4 → Security Group → EC2 port 80 → container port 7860|
|Active data|Kotaemon → `/app/ktem_app_data` → bind mount → `/opt/fcaj/ktem_app_data` on EBS|
|Backup|EC2 → AWS CLI → IAM Role → S3 prefix `ktem_app_data/`|
|Monitoring|EC2 → basic CloudWatch metrics → CloudWatch Alarm → operator views the Console|
|Cost|Account charges → AWS Budgets → alert thresholds|
|Models|Kotaemon on EC2 → external Gemini API → embeddings or answers|

## Component responsibilities

|Component|Responsibility|Status|
|-|-|-|
|Amazon EC2|Runs Ubuntu, Docker, and Kotaemon|Implemented|
|Amazon EBS|Stores PDFs, SQLite, Chroma, LanceDB, cache, and indexes|Implemented|
|Amazon S3|Stores an independent backup in a private bucket|Implemented|
|Amazon CloudWatch|Monitors CPU, network, status checks, and a CPU alarm|Partially implemented|
|AWS IAM|Provides temporary EC2-to-S3 permissions|Implemented|
|VPC and Security Group|Controls SSH port 22 and HTTP port 80|Implemented|
|AWS Budgets|Tracks the account budget|Implemented|
|Gemini API|Provides chat and embedding models|External to AWS|

## Architecture decisions

The single-EC2 architecture was selected because it is easy to deploy and observe and fits the demonstration budget. EBS replaces EFS because there is only one server. The image is built directly on EC2, so ECR and ECS Fargate are unnecessary. A public subnet avoids a NAT Gateway, while direct public IPv4 access avoids ALB cost.

The trade-offs are that EC2 is a single point of failure, the public IP may change after Stop/Start, HTTP traffic is unencrypted, and the system cannot scale automatically. These limitations must remain explicit in the report and be addressed in the roadmap before a production deployment.

## Security principles

- Do not store long-term AWS access keys on EC2; use an IAM Role.
- Restrict SSH to an administrator `/32` address.
- Enable S3 Block Public Access and grant access only to the backup bucket.
- Never commit `.env` or `GEMINI_API_KEY` to Git.
- Open public HTTP only for the demonstration period and change default passwords first.
