---
title: "Prerequisites and local execution"
date: 2026-08-05
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Prerequisites

Prepare the following before starting:

- An AWS account permitted to create EC2, EBS, S3, an IAM Role, a CloudWatch Alarm, and an AWS Budget.
- A computer with Git, Docker Engine or Docker Desktop, and at least 4 GB of available memory.
- A valid Gemini API key with quota for the selected chat and embedding models.
- The `fcaj-rag-chat` repository and a small, non-sensitive PDF for testing.
- An email address that can receive AWS Budgets notifications.


## Prepare the source code

Clone the repository and enter the project directory:

```bash
git clone REPOSITORY_URL fcaj-rag-chat
cd fcaj-rag-chat
```

Create the project's `.env` file and add the Gemini API key. Use the exact variable name expected by the source code.

```dotenv
GEMINI_API_KEY=YOUR_GEMINI_API_KEY
```

Ensure `.gitignore` excludes `.env`, then verify that it is not staged:

```bash
grep -qxF '.env' .gitignore || echo '.env' >> .gitignore
git status --short
```

## Build and run locally

Build the image from the project Dockerfile:

```bash
docker build -t fcaj-rag-chat:local .
```

Create a data directory outside the container and run the application:

```bash
mkdir -p ./ktem_app_data
docker run -d \
  --name fcaj-rag-chat-local \
  --env-file .env \
  -p 7860:7860 \
  -v "$(pwd)/ktem_app_data:/app/ktem_app_data" \
  fcaj-rag-chat:local
```

In PowerShell, replace `$(pwd)` with `${PWD}` if Docker Desktop does not accept Bash syntax. Open `http://localhost:7860` after the container becomes ready.

## Baseline test

1. Confirm that the container is `Up` with `docker ps`.
2. Run `docker logs --tail 100 fcaj-rag-chat-local` and verify that no model or API-key error appears.
3. Sign in to Kotaemon and verify the Gemini chat and embedding models.
4. Upload a small PDF and wait for indexing to finish.
5. Ask at least three questions answered by the document and inspect the citations.
6. Restart the container, reopen the interface, and confirm that the document remains available.

```bash
docker restart fcaj-rag-chat-local
docker inspect fcaj-rag-chat-local \
  --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
```

## Evidence to retain

- Successful `docker ps` output and startup logs.
- The model configuration page with the API key redacted.
- An indexed PDF and an answer with a citation.
- Mount inspection output and data retained after restart.

Proceed to AWS only after the local baseline passes. This separates application/model problems from networking, IAM, or EC2 configuration problems.
