---
title: "Blog 1"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
# Amazon EC2 Isn't as Simple as It Seems: Lessons I Learned from AWS Labs

Hello everyone!

While working through AWS hands-on labs, Amazon EC2 is undoubtedly one of the first services that anyone learning Cloud will encounter. At first, it seems as simple as "renting a virtual machine," but the more labs I completed, the more I realized there are a few concepts that are easy to misunderstand if you are not careful. Today, I'd like to share some of the lessons I learned along the way.

### 1. Stop Is Different from Terminate
At first, I thought shutting down the operating system was enough. However, the instance still exists in the **Stopped** state, and the attached EBS volume can still incur storage charges even though compute charges stop. Only when you **Terminate** an instance is it permanently deleted (and the default EBS volume is also deleted unless you choose to keep it).

This is something many beginners easily misunderstand because **Stop** feels completely safe, while in reality, you may still be charged a small amount if you forget to clean up your EBS volumes.

### 2. Free Tier Is Calculated by Total Hours, Not Per Instance
The AWS Free Tier provides **750 EC2 instance hours per month**, but these hours are shared across all eligible EC2 instances, not allocated separately to each instance. If you run multiple instances simultaneously during your labs or simply forget to stop them after use, your free hours will be consumed much faster than you might expect.

### 3. Automate Instance Shutdown Instead of Relying on Memory
One approach I found quite useful is combining **Amazon EventBridge** with **AWS Lambda** to automatically stop EC2 instances at a fixed time every day (for example, 11:00 PM), instead of relying on yourself to remember after each lab session. For beginners like me who occasionally forget, this is an effective way to reduce the risk of unexpected charges.

### Things to Do After Every Lab Session
* ✅ Check the EC2 Dashboard to make sure there are no instances still in the **Running** state.
* ✅ Give your instances clear names (including the creation date) so it's easier to identify which ones can be deleted.
* ✅ Consider automating instance shutdown with **AWS Lambda** and **Amazon EventBridge** for short-term testing environments.
* ✅ Enable **Billing Alerts** to receive notifications before your AWS costs exceed your expected budget.

![Blog 1](/images/blog1.jpg)

Overall, Amazon EC2 is one of the most important services for understanding AWS. However, precisely because it seems so familiar and straightforward, many beginners—including myself—can easily overlook small but important details. I hope these lessons will be helpful to anyone who is currently learning Cloud on AWS!