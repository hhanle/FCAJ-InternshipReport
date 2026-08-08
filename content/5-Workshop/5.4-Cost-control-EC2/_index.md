---
title: "Cost control and EC2 provisioning"
date: 2026-08-05
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Select a Region and create a budget

Use the `ap-southeast-1` (Singapore) Region for all resources. Before creating EC2, open **Billing and Cost Management → Budgets → Create budget**, select **Cost budget**, and configure:

- Period: Monthly.
- Budget: **USD 15**, matching the evidenced configuration in the report.
- 80% alert: USD 12.
- 100% alert: USD 15.
- Email: an address monitored throughout the workshop.

{{% notice warning %}}
AWS Budgets sends alerts; it is not a hard spending limit and does not stop EC2 automatically. Billing data can be delayed, so stop the instance whenever it is not in use.
{{% /notice %}}

## Launch EC2

Open **EC2 → Instances → Launch instances** and configure:

|Property|Recommended value|
|-|-|
|Name|`fcaj-rag-chat`|
|AMI|Ubuntu Server 24.04 LTS|
|Instance type|`t3.medium` (2 vCPU, 4 GiB RAM)|
|Key pair|Create or select a securely managed key pair|
|VPC/Subnet|Default VPC, public subnet|
|Public IPv4|Enable for the demonstration|
|EBS|gp3, 30–60 GiB depending on document size and budget|

Wait until the instance is **Running** with **Status checks 2/2**. Record its Instance ID, Availability Zone, and current public IPv4 address, but redact these values from published screenshots.

## Configure the Security Group

Create a Security Group named `fcaj-rag-chat-sg` with these inbound rules:

|Type|Port|Source|Purpose|
|-|-|-|-|
|SSH|22|Administrator IP address `/32`|Server administration|
|HTTP|80|`0.0.0.0/0` only during the demo|Web interface access|

Do not expose port `7860` directly because Docker maps host port 80 to it. For an internal test, restrict port 80 to the required client addresses instead of opening it to the entire internet.

## Connect over SSH

From PowerShell, connect with the key pair:

```powershell
ssh -i .\YOUR_KEY.pem ubuntu@YOUR_PUBLIC_IP
```

If SSH times out, verify the current public IPv4 address, the administrator's network IP, the port 22 inbound rule, and the Security Group attached to the instance.

## Install the server environment

On EC2, update packages and install Docker, Git, and AWS CLI:

```bash
sudo apt-get update
sudo apt-get install -y docker.io git awscli
sudo systemctl enable --now docker
sudo usermod -aG docker ubuntu
```

Sign out and reconnect so that Docker group membership takes effect. Verify the installation:

```bash
docker --version
git --version
aws --version
sudo systemctl is-active docker
```

At the end of this section, EC2 should be healthy, SSH should be restricted to the administrator, HTTP should be open only to the intended scope, and Docker should be ready.
