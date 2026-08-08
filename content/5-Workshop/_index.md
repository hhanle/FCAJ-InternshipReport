---
title: "Workshop"
date: 2026-08-05
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# DEPLOYING THE KOTAEMON RAG CHAT SYSTEM ON AWS

This workshop guides you through moving a Kotaemon-based RAG Chat application from a local computer to AWS using a single-server architecture optimized for demonstration, persistent data, and low cost. The application runs in Docker on Amazon EC2; Amazon EBS separates application data from the container lifecycle; Amazon S3 stores backups; and IAM, Security Groups, Amazon CloudWatch, and AWS Budgets support security, monitoring, and cost control.

After completing the workshop, you will be able to:

- Run Kotaemon with Gemini chat and embedding models locally and on EC2.
- Preserve PDFs, SQLite, Chroma, LanceDB, and indexes when the container is restarted or recreated.
- Back up data from EBS to S3 and test a restore into an isolated directory.
- Monitor EC2, troubleshoot common issues, and clean up resources safely.

#### Contents

1. [Introduction](5.1-introduction/)
2. [Architecture](5.2-architecture/)
3. [Prerequisites and local execution](5.3-prerequisites-local/)
4. [Cost control and EC2 provisioning](5.4-cost-control-ec2/)
5. [Deployment and persistent storage](5.5-deployment-persistence/)
6. [Backup and restore](5.6-backup-restore/)
7. [Monitoring and security](5.7-monitoring-security/)
8. [Operations and troubleshooting](5.8-operations-troubleshooting/)
9. [Cost, cleanup, and future development](5.9-cost-cleanup-roadmap/)
