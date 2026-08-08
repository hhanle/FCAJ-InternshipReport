---
title: "Blog 3"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# 🔐 I Used to Hardcode API Keys in ECS Task Definitions — Here's How We Fixed It with AWS Systems Manager Parameter Store

Hello everyone! 👋

While deploying our RAG chatbot (forked from Kotaemon) to Amazon ECS Fargate, we needed to configure the Gemini API Key for the backend container. Our usual approach was straightforward: simply paste the API Key into the environment variables section of the ECS task definition and deploy the service.

Everything worked perfectly. The application ran without any issues. However, while reviewing our project before submitting the final report, I suddenly realized we had introduced a fairly serious security problem.

#### 😬 The Problem: The API Key Was Exposed Everywhere

An ECS task definition is not a good place to store secrets. The API Key is displayed directly in the AWS Management Console, meaning anyone with permission to view the task definition can immediately see the key—even if they only have read-only access and cannot modify anything.

On top of that, whenever we needed to rotate the key (for example, if it was compromised or expired), we had to manually edit the task definition and create a new revision. It worked, but it was tedious and easy to forget.

Looking back, it felt like leaving the front door wide open while making sure the gate was securely locked. We spent time securing the network layer (Security Groups, Application Load Balancer, etc.) but overlooked the fact that the API Key itself was the most sensitive asset.

#### 🛠️ The Solution: Move All Secrets to AWS Systems Manager Parameter Store

After researching AWS best practices, we decided to remove all sensitive values (the Gemini API Key and several internal configuration values) from the task definition and store them in AWS Systems Manager Parameter Store as SecureString parameters. This means the values are encrypted with AWS KMS when stored and only decrypted when they are actually needed.

Our implementation consisted of three main steps.

**Create SecureString Parameters**

Instead of storing the actual value directly in the task definition, we created a dedicated parameter with a hierarchical naming convention, for example:

`/fcaj-rag-chat/prod/gemini-api-key`

We selected the SecureString type so AWS automatically encrypts the value using the default KMS key.

**Grant Permissions to the Task Execution Role**

This was the part that took us the longest to figure out.

Containers cannot automatically read values from Parameter Store. When the secrets section is used in an ECS task definition, the ECS/Fargate agent uses the Task Execution Role to retrieve parameter values before the container starts.

This role must have the ssm:GetParameters permission for the specific parameter. If the parameter is encrypted with a customer-managed KMS key, the role also needs the kms:Decrypt permission for that key.

Without these permissions, the task may fail during startup with errors such as ResourceInitializationError.

**Reference the Parameter Instead of the Actual Secret**

Inside the container definition, instead of defining the real API Key in the environment section, we configured the secrets section and referenced the ARN of the Parameter Store entry.

When the container starts, the ECS agent automatically retrieves the value from Parameter Store and injects it into the container as an environment variable.

As a result, the task definition only stores the parameter name or ARN instead of the plaintext API Key.

However, because the secret is still injected into the container as an environment variable, applications should avoid logging all environment variables and should carefully restrict access to the running container.

#### ✅ The Results

- Viewing the task definition in the AWS Console now only shows the parameter path instead of the actual API Key.
- Rotating the API Key is much easier—we simply update the value in Parameter Store and perform a Force New Deployment for the ECS service without modifying the task definition.
- We feel much more comfortable sharing task definitions or deployment configurations with teammates because sensitive information is no longer exposed.

#### 💡 Lessons We Learned

- Don't assume "if it works, it's good enough." Security is often overlooked in places that seem insignificant, such as environment variables.
- The Task Execution Role and the Task Role are two different IAM roles, and it's easy to confuse them when you're new to ECS. Permissions for reading Parameter Store secrets must be granted to the Task Execution Role.
- For secrets that require versioning, lifecycle management, or automatic rotation, AWS Secrets Manager is generally a better choice. However, automatic rotation depends on the type of secret and may require an additional Lambda rotation function. For our relatively simple use case, Parameter Store SecureString was more than sufficient.

If your team is still hardcoding API keys or other secrets directly into ECS task definitions, I'd highly recommend trying this approach. It's surprisingly easy to implement and provides much better security.

Has anyone here run into permission issues with Parameter Store or AWS Secrets Manager while working with ECS? We'd love to hear how you solved them! 🙌

![Blog 3](/images/blog3.png)

#### 📚 References

- Pass sensitive data to an Amazon ECS container – Amazon ECS Developer Guide: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specifying-sensitive-data.html
- Pass Systems Manager parameters through Amazon ECS environment variables – Amazon ECS Developer Guide: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-ssm-paramstore.html
- SecureString parameters – AWS Systems Manager User Guide: https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-securestring.html
- AWS KMS encryption for SecureString parameters – AWS Systems Manager User Guide: https://docs.aws.amazon.com/systems-manager/latest/userguide/secure-string-parameter-kms-encryption.html
- Creating a Parameter Store parameter using the AWS CLI – AWS Systems Manager User Guide: https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html
- AWS Systems Manager Parameter Store Overview: https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html