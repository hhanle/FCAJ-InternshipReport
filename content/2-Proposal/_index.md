---
title: "Proposal"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# DEPLOYING THE KOTAEMON RAG CHAT SYSTEM ON AWS

### Team Members
|Student ID|Member Name|
|-|-|
|3122411020|Đàm Thị Ngọc Châu|
|3122411049|Lê Gia Hân|
|3122411141|Phan Thị Hồng Nhiên|
|3122411173|Võ Hoàng Kim Quyên|
|3122411243|Phan Thị Hải Vân|

### 1. Executive Summary  
FCAJ RAG CHAT is a Retrieval-Augmented Generation (RAG) document question-answering application based on the open-source Kotaemon project. It has been successfully run in a local environment using Docker and the Gemini API for both the chat and embedding models.

The goal of this project is to deploy the application to AWS Cloud in a containerized form, with persistent data storage, basic security configuration, operational monitoring, cost control, and a clear resource cleanup process after the demo, while meeting FCAJ's minimum requirement for the number of AWS services used.

The deployment architecture is simplified compared with a fully containerized model (without ECS Fargate, Application Load Balancer, EFS, or NAT Gateway) to reduce cost and complexity, making it suitable for a short-term workshop: EC2 runs the Kotaemon Docker container, EBS provides persistent data storage, S3 stores backups and supporting evidence, CloudWatch provides monitoring, IAM controls permissions, and AWS Budgets provides cost alerts.  

### 2. Problem Statement  
#### Current Problems
- The application currently runs only on a personal computer and cannot be accessed remotely for demonstrations or sharing.
- Application data may be lost when the Docker container is restarted or rebuilt if separate persistent storage is not available.
- There is no operational monitoring mechanism for CPU, network, and server status, nor any alerting mechanism for incidents.
- There is a security risk if the Gemini API key is exposed or accidentally committed to GitHub.
- There is no cost alert mechanism, which may result in unexpected AWS charges.

#### Solution
- Containerize and deploy the application on EC2 so that it can be accessed through a public IP address.
- Attach Amazon EBS as persistent storage and mount it to the application's data directory.
- Periodically back up data to Amazon S3.
- Configure CloudWatch for monitoring and alerts (high CPU usage and failed status checks).
- Use an IAM Role and/or SSM Parameter Store to protect the API key and access permissions, avoiding hard-coded credentials in the source code.
- Configure AWS Budgets to send alerts when costs exceed the defined threshold.  

#### Benefits
- The application can be accessed remotely.
- Data persists across container restarts or rebuilds.
- Operational monitoring and proactive cost control are available.
- The system follows basic AWS security practices (IAM, security groups, and secret management).  

### 3. Solution Architecture  
The system uses a simplified architecture on a single EC2 instance, with persistent storage (EBS) and backup/asset storage (S3) separated, together with monitoring and cost control.

![RAG CHAT](/images/2-Proposal/platform_architecture.png)

#### Main Workflow
- Users → Browser → EC2 (via HTTP/public IP)
- EC2 ↔ EBS (persistent storage), EC2 → S3 (backup), EC2 → CloudWatch (metrics), EC2 → Gemini API (chat/embedding, with permissions granted through an IAM Role)
- CloudWatch → CloudWatch Alarm → Dev/Admin (incident alert)
- AWS Budgets → Email alert → Dev/Admin (cost alert)

#### AWS Services Used
|**Service**|**Role**|  
|-------------|-----------|
|**Amazon EC2**|Runs the FCAJ RAG Chat Docker container|
|**Amazon EBS**|Stores ktem_app_data: SQLite database, vector index, and uploaded files|
|**Amazon S3**|Stores backups, sample documents, and screenshots used as report evidence|
|**CloudWatch Metrics**|Monitors EC2 CPU, network, and status checks|
|**CloudWatch Alarms**|Provides alerts when CPU usage is high or status checks fail|
|**AWS Budgets**|Sends email alerts when costs exceed the defined threshold|
|**IAM**|The IAM Role FCAJ-EC2-S3-Backup-Role is attached to EC2 and grants access to S3/CloudWatch, avoiding hard-coded access keys|
|**VPC / Subnet / Security Group**|Controls access to SSH and the application port|
|**SSM Parameter Store**|Securely stores the Gemini API key/configuration|
|**CloudWatch Logs**|Stores application/Docker logs with a 7-day retention period|


#### Network Design
- Use the default VPC (no NAT Gateway is created, avoiding unnecessary hourly costs).
- The Security Group allows SSH access only from the students' IP addresses.
- The Security Group allows application port 80 (public HTTP—SSL/HTTPS is not configured) from the internet and maps it to port 7860 of the Docker container for a public demonstration.
- EC2 runs in a public subnet and uses the EC2 instance's public IP address instead of a Load Balancer. 

*Known Limitations*
- The application is currently served over plain HTTP, without an SSL/HTTPS certificate configured (because the simplified architecture does not use ALB + ACM).
- The CloudWatch Alarm is not currently connected to an Amazon SNS Topic, so it only displays the alarm status in the AWS Console and does not automatically send emails to Dev/Admin. An SNS Topic can be added at a later stage if necessary.
- By default, CloudWatch (EC2 basic monitoring) only monitors CPU, network, and status checks. Monitoring disk space and memory usage within Ubuntu requires the optional installation of the CloudWatch Agent.

### 4. Technical Implementation  
The deployment process is divided into stages, from the local baseline through reporting and resource cleanup:
* a. Local baseline: verify that Docker, the Gemini chat/embedding models, and the PDF upload–question-answering workflow operate reliably (completed).
* b. AWS account setup & cost safety: select a region, create a Budget alert, and confirm that no expensive services are used.
* c. EC2 infrastructure provisioning: launch an Ubuntu instance and configure the Security Group, key pair, and EBS.
* d. Environment setup & application deployment: install Docker/Git/AWS CLI, clone the repository, build the image, and run the container on EC2.
* e. Persistent storage & backup: mount EBS to the application data directory, test persistence, and periodically back up data to S3.
* f. Monitoring, security, testing & optimization: configure CloudWatch and IAM, perform end-to-end testing, clean up resources, and write the workshop report.

### 5. Roadmap & Implementation Milestones  
|No.|Stage|Duration|
|-|-|-|
|1|Local baseline|0.5–1 day|
|2|AWS setup & cost safety|0.5 day|
|3|Provision EC2 instance|0.5–1 day|
|4|Configure Security Group|0.5 day|
|5|Set up the server environment|0.5–1 day|
|6|Deploy the application to EC2|1 day|
|7|Persistent storage with EBS|0.5–1 day|
|8|S3 asset storage & backup|0.5 day|
|9|Manual backup to S3|0.5 day|
|10|Monitoring with CloudWatch|0.5–1 day|
|11|Security & configuration review|0.5 day|
|12|End-to-end testing on AWS|1 day|
|13|Cost optimization & cleanup|0.5 day|
|14|Write the workshop report|2–4 days|

### 6. Budget Estimate  
* Proposed configuration: EC2 t3.medium (2 vCPU / 4 GB RAM) to ensure stable PDF indexing, with an estimated actual cost of approximately **USD 15.35/month** for a demo running ~120 hours/month. 
* AWS Budgets is configured to send an email alert when costs reach the USD 20 threshold (providing a safety margin over the estimated cost). Other items (S3, basic CloudWatch metrics, and the Gemini API free tier) have negligible or zero cost.

### 7. Risk Assessment  
|Risk|Impact Level|
|-|-|
|Data loss after rebuilding/restarting the container|High|
|Exposure of the Gemini API key on GitHub|High|
|Unexpected AWS charges|High|
|Missing cleanup steps after the demo|High|
|Insufficient EC2 RAM / unexpected container shutdown|Medium|
|Public IP changes after restart / incorrect Security Group configuration|Medium|
|Gemini model not supported / API quota exceeded|Medium|
|Accidental EBS deletion / publicly accessible S3 bucket|Medium|
|Missing CloudWatch logs when errors occur|Low|
|Slow processing of large PDF files|Medium|
|Differences between the local and EC2 environments|Medium|

### 8. Expected Outcomes  
*Technical*
- Successfully deploy the RAG application on AWS using a simple, cost-effective containerized architecture.
- Ensure data persistence across restarts/rebuilds using EBS, with backups to S3.
- Set up basic monitoring (CloudWatch) and cost control (AWS Budgets).
- Apply basic security practices: IAM, Security Groups, and secret management without hard-coding.

*Value*
- Improve skills in deploying and operating systems on AWS.
- Understand how to simplify cloud architecture to suit the budget and scale of a demo/workshop.
- Produce complete documentation and evidence for the FCAJ workshop report, with the option to expand to a more comprehensive deployment solution (ECS/ALB/EFS) in the future if needed.
