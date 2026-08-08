---
title: "Blogs Posted"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section will list and introduce the blogs you have posted to [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj). For example:

###  [Blog 1 - Amazon EC2 Isn't as Simple as It Seems: Lessons from My AWS Labs](3.1-blog1/)
This article shares practical lessons learned from hands-on experience with Amazon EC2 while completing AWS lab exercises. It highlights several concepts that beginners often misunderstand, such as the difference between **Stop** and **Terminate**, how the **AWS Free Tier** for EC2 is calculated, and effective ways to avoid unexpected costs through resource management and automation. In addition, the article provides a few best practices to follow after each lab session, helping learners use EC2 more efficiently, safely, and cost-effectively.

###  [Blog 2 - Running into HTTP 429 Errors When Calling the Gemini API on AWS – How Our Team Fixed It](3.2-blog2/)
This article shares the team's real-world experience troubleshooting and resolving **HTTP 429 (Too Many Requests)** errors encountered while deploying a Retrieval-Augmented Generation (RAG) chatbot on Amazon EC2 that integrates with the Gemini API. It explains why the issue only appeared in the production environment, discusses the potential impact of unhandled rate-limiting errors, and presents a resilient solution combining exponential backoff with jitter, batching and chunking, Amazon Bedrock as a fallback service, and Amazon CloudWatch for monitoring and alerting. Finally, the article summarizes the key lessons learned about designing reliable cloud applications that can gracefully handle failures and external service limitations.

###  [Blog 3 - I Used to Hardcode API Keys in ECS Task Definitions — Here's How We Fixed It with AWS Systems Manager Parameter Store](3.3-blog3/)
This article shares our team's experience improving the security of a RAG chatbot deployed on Amazon ECS Fargate by replacing hardcoded Gemini API keys in the ECS Task Definition with **AWS Systems Manager Parameter Store**. It discusses the security risks of storing sensitive information directly in task definition environment variables and explains the process of migrating secrets to **SecureString** parameters, configuring the **Task Execution Role** with the appropriate permissions, and referencing secrets securely through ECS. Finally, the article summarizes key lessons learned about secret management on AWS, the differences between the Task Execution Role and Task Role, and when to choose AWS Systems Manager Parameter Store or AWS Secrets Manager to better protect and manage sensitive information in cloud applications.