---
title: "Introduction"
date: 2026-08-05
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Workshop objectives

Kotaemon is an open-source RAG application that lets users upload documents, build indexes, ask questions, and receive answers with citations. This project does not rebuild Kotaemon's core RAG algorithms. Its contribution is the Gemini configuration, Docker packaging, AWS deployment, persistent storage design, backup, monitoring, security, and operational procedures.

The workshop addresses two practical problems: an application running only on a personal computer is difficult to share, and data stored in a container's writable layer may be lost when the container is deleted or recreated. The solution uses one EC2 instance for compute, EBS for active data, and S3 for an independent backup.

## RAG processing flow

The system processes documents in two stages:

1. **Indexing:** read a PDF or URL, normalize its content, split it into chunks, create embeddings with Gemini, and store vectors with metadata.
2. **Question answering:** receive a question, retrieve relevant chunks using hybrid search, assemble the context, and ask the Gemini chat model to produce an answer with citations.

According to the configuration recorded in the report, documents are split into 1,024-token chunks with a 256-token overlap. Chroma stores embeddings, LanceDB stores documents, and SQLite stores configuration and metadata. Retrieval returns 10 chunks by default, and no dedicated reranker is enabled.

## Deployment scope

|In scope|Currently out of scope|
|-|-|
|One EC2 instance running Docker and Kotaemon|Multi-instance cluster or Auto Scaling|
|EBS storing `ktem_app_data`|EFS or a managed database|
|Private S3 backup|Automated backups and scheduled recovery tests|
|Basic CloudWatch metrics and a CPU alarm|RAM, disk usage, CloudWatch Agent, and centralized logs|
|HTTP port 80 for the demo|HTTPS, domain name, WAF, and ALB|
|Gemini API for chat and embeddings|Amazon Bedrock in the current workflow|

## Completion criteria

- The Kotaemon interface is accessible from a browser.
- The container runs reliably with the `unless-stopped` restart policy.
- Data remains available after restarting or recreating the container.
- EC2 can back up data to S3 through an IAM Role without long-term access keys.
- CloudWatch displays metrics and an alarm; AWS Budgets tracks spending.
- Startup, verification, restore, troubleshooting, and cleanup procedures are documented.

