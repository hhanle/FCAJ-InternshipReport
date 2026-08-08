---
title: "Operations and troubleshooting"
date: 2026-08-05
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Pre-demo startup procedure

Complete each step in order and proceed only after its expected result is met:

|Step|Action|Expected result|
|-|-|-|
|1|Select Singapore and start EC2|`Running`, status checks `2/2`|
|2|Obtain the current public IPv4 address|A current address after Stop/Start|
|3|Inspect the Security Group|SSH 22 and HTTP 80 have the correct sources|
|4|Connect to EC2 over SSH|An Ubuntu shell is available|
|5|Check the Docker service|State is `active`|
|6|Check the container|`Up`, with `80 → 7860`|
|7|Inspect the mount and logs|Correct EBS path and no critical error|
|8|Call the application from EC2|An HTTP response is returned|
|9|Open the interface in a browser|Kotaemon is displayed|
|10|Check a PDF and one question|Existing data and citations work|

## Quick diagnostic commands

```bash
sudo systemctl is-active docker
docker start fcaj-rag-chat 2>/dev/null || true
docker ps
docker inspect fcaj-rag-chat \
  --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
docker logs --tail 100 fcaj-rag-chat
curl -I http://127.0.0.1
df -h /opt/fcaj
free -h
aws s3 ls s3://YOUR_BACKUP_BUCKET/ktem_app_data/ \
  --recursive --summarize --region ap-southeast-1
```

On a Windows administration computer, test the port before opening a browser:

```powershell
Test-NetConnection YOUR_PUBLIC_IP -Port 80
```

## Troubleshooting table

|Symptom|Common cause|Check and resolution|
|-|-|-|
|SSH timeout|Public IP changed or port 22 source is wrong|Get the new IP; verify the administrator IP and Security Group|
|Web timeout|Port 80 is closed or the container is stopped|Use `Test-NetConnection`, `docker ps`, and local `curl`|
|Container is `Exited`|Insufficient RAM or an `.env`, model, or dependency error|Inspect `docker logs`, `free -h`, and environment variables|
|Interface errors immediately after start|Application is still initializing|Follow logs and wait for readiness instead of repeatedly restarting|
|PDFs or indexes are missing|Wrong bind mount or an empty host directory|Use `docker inspect`; inspect `/opt/fcaj/ktem_app_data`|
|S3 returns `AccessDenied`|Role is missing or its policy lacks permission|Run `aws sts get-caller-identity`; inspect the instance profile and policy|
|Gemini returns 401/403|Invalid, expired, or unauthorized API key|Check `.env`, the model name, and rotate the key|
|Gemini returns 429|Quota exceeded|Reduce requests, use a small PDF, wait for quota, and keep fallback evidence|
|CPU remains high|Large image build/PDF index or concurrent queries|Inspect CloudWatch and logs; reduce workload or stop the task|
|Alarm sends no email|No alarm action or unconfirmed SNS subscription|Inspect Actions and the SNS subscription|

## Application update procedure

1. Back up data to S3 and verify the objects.
2. Record the currently running image name for rollback.
3. Pull source changes and build a new versioned image such as `v2`.
4. Stop the old container while preserving the EBS directory.
5. Run the new container with the same `.env`, port, and bind mount.
6. Inspect logs, the interface, PDFs, and one query.
7. If validation fails, run the old image with the same bind mount.

Do not rely only on the `latest` tag. Versioned tags identify the running image and make rollback explicit.

## End-to-end test

A complete test covers: open interface → upload PDF → index → ask a question → inspect citations → restart container → verify data → back up to S3 → restore into a new directory. Record time, result, and redacted evidence for every step.

Do not publish exact RAG quality percentages without a benchmark set. If evaluation is required, use 20–30 questions across several documents and measure retrieval success, groundedness, citations, response time, and indexing time.
