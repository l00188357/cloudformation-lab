# CloudFormation Assignment — [Cassiana de Oliveira] (L00188357)

## 📋 Assignment Overview

This repository contains my individual solution for the CloudFormation assignment.
It covers:
- **Part A(a):** A static website hosted on Amazon S3, created via CloudFormation, with my name and Lnumber as the header.
- **Part A(b):** A three-tier architecture (Web tier, Application tier, Database tier) built with CloudFormation.
- **Part B:** Best practices applied — security, scalability, performance, and commented code throughout.

---

## 📅 Project Timeline / Cronogram

I planned to execute this assignment over 15 days.
The plan below was informed by research and my own study planning notes.

| Day | Focus | Branch | Deliverable |
|-----|-------|--------|-------------|
| 1 | Research CloudFormation, S3, YAML basics | `main` | `research-notes.md` |
| 2 | Repo structure + YAML skeleton | `feature/static-website` | Template skeleton |
| 3 | Add S3 bucket resource | `feature/static-website` | Bucket resource |
| 4 | Add bucket policy + outputs | `feature/static-website` | Working Part A YAML |
| 5 | Deploy + create HTML files | `feature/static-website` | Live website + screenshot |
| 6 | Refactor Part A (best practices) | `refactor/part-a-best-practices` | Improved Part A |
| 7 | Review + update README | `main` | README updated, PR merged |
| 8 | Research 3-tier architecture | `feature/three-tier` | Architecture notes |
| 9 | Build network layer (VPC) | `feature/three-tier` | `network.yaml` |
| 10 | Build web tier (ALB) | `feature/three-tier` | `web-tier.yaml` |
| 11 | Build app tier (EC2 + ASG) | `feature/three-tier` | `app-tier.yaml` |
| 12 | Build database tier (RDS) | `feature/three-tier` | `database-tier.yaml` |
| 13 | Deploy, test, screenshot, document | `feature/three-tier` | Full README + screenshots |
| 14 | Final polish + submit | `main` | Final submission |

---

## 📚 Research & Learning Resources

The following resources were used during the research phase of this assignment.

### YouTube Videos
- [AWS CloudFormation Explained | Deploy Infrastructure as Code](https://youtu.be/iasdbb6p2so?si=A5VtGmxe9cN-K-lG) — introductory overview of Infrastructure as Code (IaC) and AWS CloudFormation.
- [Part-28 || AWS CloudFormation Tutorial for Beginners | Step-by-Step Infrastructure as Code (IaC)] (https://www.youtube.com/watch?v=6jFLpf0aDg0) - Tutorial of AWS CloudFormation.

### Official Documentation
- [AWS CloudFormation] (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [AWS Lambda & IaC Foundation](https://docs.aws.amazon.com/lambda/latest/dg/foundation-iac.html)
- [Hosting a Static Website on Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)

### LLM Assistance Declaration
As per the assignment brief, I declare the following use of LLMs:
- **Tool used:**  DeepSeek (planning and conceptual explanations) and Claude (troubleshooting AWS CLI issues and generating the initial YAML skeleton)
- **Purpose:** Used for conceptual explanations (what is YAML, what is a three-tier architecture), planning the study timeline, and helping me understand AWS documentation.
- **Prompt asked Claude on 18th October 2026:** Write a skeleton static-website.yaml with only the top-level sections: AWSTemplateFormatVersion, Description, Parameters, Resources, Outputs — leave them empty or with placeholders.

---

## 🏗️ Part A(a): Static Website on S3

🔗 **Live Website URL:** https://d1hseph1mrdj8x.cloudfront.net/

This section uses a CloudFormation template (`static-website.yaml`) to deploy a secure, public-facing static website. The S3 bucket is kept private, and content is delivered globally via Amazon CloudFront using Origin Access Control (OAC).

**How to deploy (AWS Console):**
1. Open the **AWS CloudFormation** console and click **Create stack** (With new resources).
2. Under "Prepare template", select **Upload a template file** and upload `static-website.yaml`.
3. Name the stack (e.g., `my-static-website`), set the parameters (`ProjectName=my-site`, `Environment=dev`), and click through to create the stack.
4. Wait for the status to change to `CREATE_COMPLETE`.
5. Open the **Amazon S3** console, find the newly created bucket, and **Upload** the `index.html` file.
6. Go to the **CloudFormation Outputs** tab to find the live `WebsiteURL`.

## 🏗️ Part A(b): Three-Tier Architecture

*(To be filled in on Day 13)*

## ✅ Best Practices Applied

1. **Origin Access Control (OAC):** The basic tutorial approach makes the S3 bucket public. I refactored the code to make the S3 bucket **completely private** and used the modern CloudFront OAC to securely authenticate and read from the bucket.
2. **Encryption & Versioning:** I added Server-Side Encryption (AES256) to the S3 bucket to secure data at rest, and enabled Bucket Versioning to allow for content rollback.
3. **HTTPS Enforcement:** I configured the CloudFront distribution to use `redirect-to-https` and enforced a minimum TLS protocol version (`TLSv1.2_2021`), securing data in transit.
4. **SPA Routing:** I added `CustomErrorResponses` so that 403 and 404 errors redirect to `/index.html` with a 200 status code, which is a best practice for modern static sites.

### Illities Analysis
- **Security:** Data is encrypted at rest and in transit. The S3 bucket is not publicly accessible; only the specific CloudFront distribution can read it via OAC.
- **Scalability:** Amazon CloudFront is a CDN that automatically scales to handle any amount of traffic. S3 and CloudFront are managed services, meaning there are no servers to provision or scale manually.
- **Performance:** CloudFront caches the website content at edge locations globally, reducing latency for users regardless of their geographic location.


## 📸 Screenshots

### 1. Live Website
<img src="screenshots/website.png" width="400" />
<img src="screenshots/website2.png" width="400" />

### 2. CloudFormation Stack Deployed
<img src="screenshots/stack.png" width="400" />
<img src="screenshots/stack2.png" width="400" />
<img src="screenshots/stack3.png" width="400" />

### 3. Amazon S3 Bucket
<img src="screenshots/bucket.png" width="400" />
<img src="screenshots/bucket2.png" width="400" />
<img src="screenshots/bucket3.png" width="400" />

---

## 🔗 Important

- **Collaborator:** Ruth.Lennon@atu.ie has been added to this repository.
- **Code commits:** All commits are my own work.
- **Lab write-up:** This README.md serves as my lab write-up.