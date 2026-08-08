---
title: "Backup and restore"
date: 2026-08-05
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Create a private S3 bucket

In `ap-southeast-1`, create a globally unique bucket such as `fcaj-rag-chat-backup-UNIQUE_SUFFIX`. Keep **Block all public access** enabled and use default SSE-S3 encryption. Do not use this bucket for public website hosting.

Organize objects by prefix:

```text
s3://YOUR_BACKUP_BUCKET/
├── ktem_app_data/
└── evidence/
```

## Grant access through an IAM Role

Create an EC2 IAM Role such as `FCAJ-EC2-S3-Backup-Role` and restrict its policy to the backup bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::YOUR_BACKUP_BUCKET"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::YOUR_BACKUP_BUCKET/*"
    }
  ]
}
```

Attach it to EC2 through **Actions → Security → Modify IAM role**. Verify identity and access on the server:

```bash
aws sts get-caller-identity
aws s3 ls s3://YOUR_BACKUP_BUCKET --region ap-southeast-1
```

## Manual backup

For a consistent SQLite and index backup, stop the container during a short maintenance window:

```bash
docker stop fcaj-rag-chat
aws s3 sync /opt/fcaj/ktem_app_data/ \
  s3://YOUR_BACKUP_BUCKET/ktem_app_data/ \
  --region ap-southeast-1
docker start fcaj-rag-chat
```

Verify object count and total size:

```bash
aws s3 ls s3://YOUR_BACKUP_BUCKET/ktem_app_data/ \
  --recursive --summarize \
  --region ap-southeast-1
```

{{% notice warning %}}
Do not add `--delete` to the routine sync command. It may remove S3 objects when a source file is missing or the wrong source path is selected.
{{% /notice %}}

## Safe restore test

Restore into an empty directory without overwriting production data:

```bash
sudo mkdir -p /opt/fcaj/restore_test
sudo chown ubuntu:ubuntu /opt/fcaj/restore_test
aws s3 sync s3://YOUR_BACKUP_BUCKET/ktem_app_data/ \
  /opt/fcaj/restore_test/ \
  --region ap-southeast-1
du -sh /opt/fcaj/ktem_app_data /opt/fcaj/restore_test
find /opt/fcaj/ktem_app_data -type f | wc -l
find /opt/fcaj/restore_test -type f | wc -l
```

For full validation, stop the main container and run a test container with `/opt/fcaj/restore_test` mounted at `/app/ktem_app_data`. Verify the PDF list, indexes, and a previously used question. Stop the test container and restart the primary container afterward.

## Incident restore procedure

1. Stop the application to prevent further writes.
2. Rename the damaged directory to retain a rollback path.
3. Create a new `ktem_app_data` directory and sync data from S3.
4. Check file permissions and the mount with `docker inspect`.
5. Start the application and inspect logs, PDFs, indexes, and a query.
6. Delete the old directory only after confirming the restore.

Retain screenshots or logs from the restore test. The source report currently evidences backup only; complete a restore test before claiming that the backup process is fully validated.
