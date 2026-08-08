---
title: "Cost, cleanup, and future development"
date: 2026-08-05
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Cost estimate

Cost depends on EC2 runtime, EBS capacity, public IPv4, S3, and CloudWatch configuration. The source report provides two calculations: an initial estimate using 60 GiB EBS and 1 GB of CloudWatch Logs, and a reconciled estimate using approximately 30 GiB EBS without Logs because Logs are not implemented.

|Scenario|60 GiB EBS + Logs estimate|~30 GiB EBS, no Logs|Comment|
|-|-|-|-|
|60 demo hours/month|USD 11.36|About USD 7.48|Below the USD 15 budget|
|120 demo hours/month|USD 15.35|About USD 11.47|The initial estimate slightly exceeds the budget|
|Continuous operation|USD 55.89|About USD 52.00|Unsuitable for the demo budget|

These figures are project estimates, not an invoice. Before submission, update the AWS Pricing Calculator and compare it with **Billing/Cost Explorer**, because prices and actual usage may differ.

## Cost reduction measures

- Stop EC2 when it is not being used for building, testing, or demonstration.
- Use `t3.medium` while memory is required; measure load before downsizing.
- Size EBS from actual data and remove unnecessary snapshots.
- Do not create a NAT Gateway, ALB, EFS, ECR, or CloudWatch Logs without a requirement and budget.
- Define S3 lifecycle and retention clearly instead of keeping unnecessary copies.
- Monitor the 80% and 100% budget thresholds without relying on alerts alone.

## Cleanup order

Clean up only after retaining evidence and validating the backups that must remain:

1. Save final redacted screenshots and verify the S3 backup.
2. Stop EC2 if more validation is required; terminate it when the project ends.
3. Release any unused static public IPv4 or Elastic IP.
4. Verify the backup, then delete unused EBS volumes.
5. Delete test or redundant snapshots.
6. Remove temporary objects; empty and delete the S3 bucket if no backup must remain.
7. Delete CloudWatch alarms, SNS topics/subscriptions, and test log groups if created.
8. Delete the custom Security Group after no resource references it.
9. Detach and remove unused IAM Roles and policies.
10. Remove `.env` from retired hosts and rotate the Gemini API key when appropriate.
11. Review Billing and Cost Explorer after cleanup for remaining charges.

{{% notice warning %}}
Do not delete EBS or S3 until the backup is checked and the data is confirmed unnecessary. Track every item explicitly; stopping EC2 alone does not mean cleanup is complete.
{{% /notice %}}

## Current assessment

|Area|Assessment|Basis|
|-|-|-|
|Demo suitability|Good|One EC2 is easy to deploy and explain|
|Data persistence|Fair|EBS and S3 backup exist; documented restore evidence is still needed|
|Security|Basic|IAM Role, SG, and private S3 exist; HTTP and `.env` remain|
|Observability|Basic|Metrics/alarm exist; action, RAM, disk, and application logs are missing|
|Cost control|Fair|A budget exists and expensive services are avoided; cleanup must be completed|
|Scalability|Low|One EC2 is a single point of failure|

## Development roadmap

|Priority|Action|Completion condition|
|-|-|-|
|P0|Redact sensitive data, change default credentials, and rotate keys|No account ID, ARN, IP address, or password remains in public material|
|P0|Complete persistence and restore tests|Evidence exists before/after restart and for restore into a new directory|
|P0|Complete cleanup after submission|Billing shows no unplanned resource|
|P1|Add HTTPS|The browser connects with valid TLS|
|P1|Connect alarms to SNS|A test notification is received|
|P1|Evaluate RAG with a benchmark question set|A documented method and result table exist|
|P2|Automate backup and retention|A schedule, logs, failure alerts, and periodic restore tests exist|
|P2|Add CloudWatch Agent/logging|RAM, disk, application logs, and retention are available|
|P3|Evaluate ALB, ECS/ECR, and EFS|Implement only when multiple instances, an SLA, and budget require them|

The workshop outcome is a demonstrable prototype: the application runs on EC2, EBS separates data from the container, S3 provides a backup, and IAM, basic monitoring, and budget controls are in place. Roadmap items are prerequisites for moving toward production, not components already implemented.
