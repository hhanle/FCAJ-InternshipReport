---
title: "Blog 2"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Running into HTTP 429 Errors When Calling the Gemini API on AWS – How Our Team Fixed It

Hello everyone! 👋

While working on our capstone project (a RAG chatbot running on Amazon EC2 and integrated with the Gemini API), our team encountered an issue that anyone who has worked with third-party API integrations has probably faced at least once: **HTTP 429 – Too Many Requests**.

At first, we were quite confused because everything worked perfectly on our local machines without any unusual behavior. However, as soon as we deployed the application to EC2 and tested it with multiple users at the same time, requests started flooding the Gemini API, and the server politely responded with **HTTP 429**.

### 🤔 Why Did Everything Work Locally but Fail on EC2?

After analyzing the problem, we realized that EC2 itself wasn't the issue—the real problem was the sudden increase in traffic.

- **On our local machines:** only one person was testing, sending just a few requests per minute. Almost any API could handle that easily.
- **On EC2:** multiple users were accessing the application simultaneously, with several worker processes sending requests in parallel. The request rate quickly exceeded the API's rate limit, resulting in HTTP 429 errors.

To put it simply: when only one person was using the API, it happily handled every request. But once the whole team started using it together, it immediately became overwhelmed. 😄

### ❗ What Happens If You Ignore This Error?

At first, we underestimated HTTP 429, thinking it was just a minor issue that could be solved by retrying a few times. However, after looking into it more carefully, we realized that without proper error handling, it could cause several serious problems:

- A task could fail halfway through—for example, reading a file → calling the API → writing results to the database. If the API call fails in the middle, the stored data may become incomplete or inconsistent.
- If the error isn't caught and handled properly, it can break the entire processing workflow of a worker running on EC2.
- End users only see a generic **HTTP 500 Internal Server Error**, with no clue about what actually went wrong.

### 🛠️ How We Solved It

Instead of applying a temporary fix, we redesigned the entire API-calling workflow to make it more **resilient**. Our solution consists of four main parts.

#### 1️⃣ Retry with Exponential Backoff + Jitter

Instead of retrying immediately—which would only make congestion worse—we configured the system to wait progressively longer after each failed attempt:

- First failure: wait **2 seconds** before retrying.
- Second failure: wait **4 seconds**.
- Third failure: wait **8 seconds**.

We also added a small amount of **jitter** (around ±0.5 seconds) so that different workers wouldn't all retry at exactly the same time and create another traffic spike.

#### 2️⃣ Reduce Requests with Batching and Chunking

To avoid reaching the rate limit too quickly, we also optimized how requests were sent:

- Combine multiple small tasks into a single API request whenever possible, reducing the total number of API calls.
- Split large datasets into smaller chunks instead of sending one huge request.

We also removed unnecessary fields from the payload to keep requests as lightweight as possible.

#### 3️⃣ Have a Backup Plan: Switch to Amazon Bedrock

If the Gemini API keeps failing (due to temporary outages or quota exhaustion), our system automatically falls back to **Amazon Bedrock**, AWS's managed generative AI service.

One important lesson we learned is that using Bedrock doesn't eliminate quota concerns—it has its own limits depending on your AWS account, Region, and token usage. However, it prevents us from relying entirely on a single AI provider.

#### 4️⃣ Monitor Everything with Amazon CloudWatch

We send all application logs (HTTP 429 errors, timeouts, etc.) from EC2 to **Amazon CloudWatch Logs**, then create **Metric Filters** to count the number of occurrences.

If the error count exceeds a predefined threshold within a certain period, **CloudWatch Alarms** automatically send notifications to our team's email through **Amazon SNS**, so we don't have to monitor logs manually.

### 💡 Lessons We Learned

- Never assume that **"it works on my local machine"** means **"it will work in production."** As traffic and the number of users increase, problems that never appeared during development can quickly surface.
- Retry is **not** a magic solution. It should only be used for temporary failures. If the underlying problem persists, endless retries simply waste resources and make the system slower.
- Any operation that may be retried should be designed to be **idempotent**, ensuring that multiple executions still produce the same correct result.
- Always have a **backup plan** when depending on external services that you don't control.

This was the first time our team truly redesigned an application workflow with **resilience** in mind instead of simply making it "work." We learned a lot throughout the process.

Have you ever encountered API rate limiting while integrating third-party services on AWS? We'd love to hear how you handled it—feel free to share your experience! 🙌

![Blog 2](/images/blog2.jpg)

### 📚 References

- Exponential Backoff and Jitter – AWS Architecture Blog: https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
- REL05-BP03 Control and Limit Retry Calls – AWS Well-Architected Framework: https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/rel_mitigate_interaction_failure_limit_retries.html
- Retry Behavior – AWS SDKs and Tools Reference Guide: https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html
- Identity-Based Policy Examples for Amazon Bedrock (IAM permissions, `bedrock:InvokeModel`): https://docs.aws.amazon.com/.../security_iam_id-based
- InvokeModel API Reference – Amazon Bedrock: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InvokeModel.html
- Alarming on Logs – Amazon CloudWatch User Guide: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Alarm-On-Logs.html
- Creating a Metric Filter for a Log Group – Amazon CloudWatch Logs: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CreateMetricFilterProcedure.html