---
title: "Deployment and persistent storage"
date: 2026-08-05
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Clone and configure the application

On EC2, clone the repository into the home directory:

```bash
cd ~
git clone REPOSITORY_URL fcaj-rag-chat
cd fcaj-rag-chat
```

Create `.env` from the project template and add the Gemini API key. Restrict the file to its owner:

```bash
chmod 600 .env
```

Verify that Git excludes the file:

```bash
git check-ignore .env
git status --short
```

## Prepare data on EBS

In the simple single-volume configuration, the EC2 root volume is already EBS. Create a dedicated directory and assign it to the Ubuntu user:

```bash
sudo mkdir -p /opt/fcaj/ktem_app_data
sudo chown -R ubuntu:ubuntu /opt/fcaj
df -h /opt/fcaj
```

This directory stores SQLite, Chroma, LanceDB, uploaded PDFs, cache, and indexes. Because it is outside the container's writable layer, recreating the container does not delete it.

{{% notice note %}}
EBS protects data from the container lifecycle, but not from deletion of the volume or termination of an instance whose root volume is deleted. The S3 backup in section 5.6 provides an independent protection layer.
{{% /notice %}}

## Build the image

Build the image directly on EC2:

```bash
docker build -t fcaj-rag-chat:v1 .
docker images fcaj-rag-chat
```

If the build stops, use `free -h`, `df -h`, and `docker system df` to inspect memory and disk usage before retrying.

## Run the container

```bash
docker run -d \
  --name fcaj-rag-chat \
  --restart unless-stopped \
  --env-file ~/fcaj-rag-chat/.env \
  -p 80:7860 \
  -v /opt/fcaj/ktem_app_data:/app/ktem_app_data \
  fcaj-rag-chat:v1
```

Verify the container and bind mount:

```bash
docker ps
docker inspect fcaj-rag-chat \
  --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
docker logs --tail 100 fcaj-rag-chat
curl -I http://127.0.0.1
```

The mount output must show:

```text
/opt/fcaj/ktem_app_data -> /app/ktem_app_data
```

Open `http://YOUR_PUBLIC_IP` in a browser. The first startup may take several minutes while the application loads dependencies and initializes.

## Test persistence

1. Upload a PDF, wait for indexing, and ask a question.
2. Record the document list and an answer with its citation.
3. Restart the container with `docker restart fcaj-rag-chat`.
4. Reopen the application and verify that the PDF, index, and previous query still work.
5. For a stronger test, remove **only the container**, then repeat the same `docker run` command with the existing bind mount.

```bash
docker stop fcaj-rag-chat
docker rm fcaj-rag-chat
# Repeat the docker run command above with the same /opt/fcaj/ktem_app_data directory
```

{{% notice warning %}}
Do not delete `/opt/fcaj/ktem_app_data` during this test. Recheck the mount path before recreating the container to avoid accidentally starting the application with an empty directory.
{{% /notice %}}
