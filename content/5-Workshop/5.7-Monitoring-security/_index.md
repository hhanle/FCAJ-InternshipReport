---
title: "Monitoring and security"
date: 2026-08-05
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Monitor EC2 with CloudWatch

EC2 basic monitoring provides metrics including `CPUUtilization`, `NetworkIn`, `NetworkOut`, and `StatusCheckFailed`. Open **EC2 → Instances → Monitoring** or **CloudWatch → Metrics → EC2 → Per-Instance Metrics**, select the correct Instance ID, and observe load while building the image, indexing a PDF, and asking questions.

These metrics help determine whether:

- CPU remains high for a sustained period.
- Network traffic appears while downloading dependencies, calling Gemini, or backing up to S3.
- An EC2 system or instance status check has failed.

Basic monitoring does not provide RAM, filesystem usage, or application logs. Those require CloudWatch Agent or another logging solution and are not part of the current deployment.

## Create a CloudWatch Alarm

Create an alarm for the instance's `CPUUtilization` metric:

1. Select `CPUUtilization` for the correct Instance ID.
2. Use **Average** with a **5-minute** period.
3. Set the threshold to **Greater than 70%** for an appropriate evaluation period.
4. Name it `fcaj-rag-chat-high-cpu`, for example.
5. Inspect its `OK`, `In alarm`, or `Insufficient data` state after creation.


Consider adding an alarm for `StatusCheckFailed >= 1`. When SNS is configured, test the notification and retain evidence of the received email instead of only the alarm configuration.

## Review IAM and S3

- EC2 must use an IAM Role instead of long-term access keys.
- The role should allow only `ListBucket`, `GetObject`, and `PutObject` for the required bucket or prefix.
- S3 Block Public Access must remain enabled, with no public bucket policy or ACL.
- Verify the active identity with `aws sts get-caller-identity`.
- Do not grant `s3:*` on `*` when the workflow needs only one bucket.

## Protect secrets

Keep the Gemini API key in `.env` with mode `600`, exclude it from Git, and never copy it into the Docker image. Before sharing the repository, run:

```bash
git check-ignore .env
git log --all -- .env
git grep -n -i 'gemini_api_key' -- ':!*.md'
```

If a key was committed, revoke and rotate it, then clean Git history according to the team's procedure. A later phase may move the secret to SSM Parameter Store; do not describe that service as implemented without evidence.

## Pre-demo security checklist

- [ ] SSH port 22 allows only the administrator `/32` address.
- [ ] Port 7860 is not directly exposed.
- [ ] The default Kotaemon password has been changed.
- [ ] `.env`, API keys, and credentials do not appear in Git or screenshots.
- [ ] S3 Block Public Access is enabled.
- [ ] The correct IAM Role is attached to EC2 and scoped to the backup bucket.
- [ ] Account ID, ARN, IP address, Instance ID, and username are redacted in public material.
- [ ] Public HTTP is open only for the required period.

