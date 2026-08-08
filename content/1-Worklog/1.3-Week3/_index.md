---
title: "Week 3 Worklog"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

* Master the configuration and management of AWS Site-to-Site VPN using strongSwan and AWS Transit Gateway.
* Understand and deploy a Hybrid DNS architecture using Amazon Route 53 Resolver for DNS resolution between on-premises and AWS environments.
* Gain hands-on experience with multi-VPC connectivity using VPC Peering and AWS Transit Gateway.
* Strengthen knowledge of AWS Compute services, including EC2, AMI, EBS, Instance Store, User Data, Instance Metadata, and Auto Scaling.

### Week 3 Tasks:
| Day | Task                                                                                                                                                                                                   | Start Date | Completion Date | Reference Material                        |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- | --------------- | ----------------------------------------- |
| 2   | **- Continue Lab 5:** <br>&emsp; + Set up AWS Site-to-Site VPN: <br>&emsp; * Configure VPN using strongSwan with Transit Gateway: Create Customer Gateway, Create Transit Gateway, Create VPN Connection, Create Transit Gateway Attachment, Configure Route Tables, Configure Customer Gateway <br>&emsp; + Clean up Resources <br>&emsp; + Infrastructure as Code Templates| 06/07/2026   | 06/07/2026      | [Lab 5 Configure VPN using strongSwan with Transit Gateway](https://000003.awsstudygroup.com/vi/5-vpnsitetosite/5.3-vpnsitetosite-optional/) <br> [Lab 5 Clean up Resources](https://000003.awsstudygroup.com/vi/6-cleanup/) <br> [Lab 5 Infrastructure as Code Templates](https://000003.awsstudygroup.com/vi/7-infrastructureascode/)|
| 3   | **- Practice Lab 6:** Hybrid DNS Management with Amazon Route 53 <br>&emsp; + Preparation steps: Create Key Pair, Launch CloudFormation Template, Configure Security Group <br>&emsp; + Connect to RDGW <br>&emsp; + Deploy Microsoft AD <br>&emsp; + Configure DNS: Create Route 53 Outbound Endpoint, Create Route 53 Resolver Rules, Create Route 53 Inbound Endpoints, Test the results <br>&emsp; + Clean up resources | 07/07/2026   | 07/07/2026      | [Lab 6 Hybrid DNS Management with Amazon Route 53](https://000010.awsstudygroup.com/vi/) |
| 4   | **- Practice Lab 7:** Network Integration with VPC Peering <br>&emsp; + Preparation steps: Launch CloudFormation Template, Create Security Group, Create EC2 instance <br>&emsp; + Update Network ACL <br>&emsp; + Create Peering Connection <br>&emsp; + Enable Cross-Peer DNS <br>&emsp; + Clean up resources | 08/07/2026   | 08/07/2026      | [Lab 7 Network Integration with VPC Peering](https://000019.awsstudygroup.com/vi/) |
| 5   | **- Practice Lab 8:** Centralized Network Management with AWS Transit Gateway <br>&emsp; + Preparation steps: Create Key Pair, Launch CloudFormation Template <br>&emsp; + Create Transit Gateway <br>&emsp; + Create Transit Gateway Attachments <br>&emsp; + Create Transit Gateway Route Tables <br>&emsp; + Add Transit Gateway Routes to VPC Route Tables <br>&emsp; + Clean up resources | 09/07/2026   | 09/07/2026      | [Lab 8 Centralized Network Management with AWS Transit Gateway](https://000020.awsstudygroup.com/vi/) |
| 6   | - Learn about AWS Compute services (Module 03): EC2, AMI, EBS, Instance Store, User Data, Metadata, and Auto Scaling | 10/07/2026   | 10/07/2026      | [Youtube AWS Study Group](https://www.youtube.com/@AWSStudyGroup) |


### Week 3 Achievements:

* Successfully configured an AWS Site-to-Site VPN using strongSwan integrated with AWS Transit Gateway.
* Successfully deployed a Hybrid DNS model using Amazon Route 53 Resolver, including Inbound Endpoint, Outbound Endpoint, and Resolver Rules.
* Established and tested connectivity between VPCs through VPC Peering and AWS Transit Gateway.
* Understood how to manage network routing using Route Tables, Transit Gateway Attachments, and Cross-Peer DNS.
* Strengthened knowledge of AWS Compute services, including EC2, AMI, EBS, Instance Store, User Data, Instance Metadata, and Auto Scaling.
