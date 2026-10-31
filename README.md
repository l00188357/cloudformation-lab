# CloudFormation Assignment — [Cassiana de Oliveira] (L00188357)

## 📋 Assignment Overview

This repository contains my individual solution for the CloudFormation assignment.
It covers:
- **Part A(a):** A static website hosted on Amazon S3, created via CloudFormation, with my name and Lnumber as the header.
- **Part A(b):** A three-tier architecture (Web tier, Application tier, Database tier) built with CloudFormation.
- **Part B:** Best practices applied — security, scalability, performance, and commented code throughout.

---

## 📚 Research & Learning Resources

The following resources were used during the research phase of this assignment.

### YouTube Videos
- [AWS CloudFormation Explained | Deploy Infrastructure as Code](https://youtu.be/iasdbb6p2so?si=A5VtGmxe9cN-K-lG) — introductory overview of Infrastructure as Code (IaC) and AWS CloudFormation.
- [Part-28 || AWS CloudFormation Tutorial for Beginners | Step-by-Step Infrastructure as Code (IaC)](https://www.youtube.com/watch?v=6jFLpf0aDg0) - Tutorial of AWS CloudFormation.

### Official Documentation
- [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [AWS Lambda & IaC Foundation](https://docs.aws.amazon.com/lambda/latest/dg/foundation-iac.html)
- [Hosting a Static Website on Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)

### LLM Assistance Declaration
As per the assignment brief, I declare the following use of LLMs:
- **Tool used:**  DeepSeek (planning and conceptual explanations) and Claude (troubleshooting AWS CLI issues and generating the initial YAML skeleton)
- **Purpose:** Used for conceptual explanations (what is YAML, what is a three-tier architecture), planning the study timeline, and helping me understand AWS documentation.
- **Prompt asked Claude on 18th October 2026:** Write a skeleton static-website.yaml with only the top-level sections: AWSTemplateFormatVersion, Description, Parameters, Resources, Outputs — leave them empty or with placeholders.
- **Prompt asked Claude on 31st October 2026:** 1)is this yaml file good enough to fufill part A-b of this assignment? 
2) “yes please”, in reply to the offer of a revised template
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

 My first version (three-tier.yaml) was my own attempt. I asked Claude to review it, and it identified several problems (no targets registered with the load balancer, no NAT for the private subnets, a region-specific AMI, SSH open to the internet). Claude then generated a revised template, which I typed into tier.yaml, committed, deployed and debugged myself. So I decided to use the second version (tier.yaml). This allowed me to improve the architecture, resolve design limitations and apply more AWS best practices.
Both versions are based on a three-tier architecture (Web Tier, Application Tier and Database Tier). However, Version 2 provides a more complete separation between the tiers, improved security and better scalability.
One of the main lessons was that creating private subnets alone does not provide the connectivity required by EC2 instances to download software. I needed to configure NAT and routing correctly.
I also learned the importance of registering application instances with load balancer target groups, using Auto Scaling Groups instead of relying on fixed instances, and avoiding unnecessary public access.
I decided to keep Version 1 in the repository because it represents my initial development work and demonstrates how I improved my understanding of AWS CloudFormation throughout the assignment.

Version 2 (tier.yaml) is the improved implementation and the version I used as my final three-tier architecture submission.

- The **web tier** (nginx) sits in public subnets behind the internet-facing ALB. It serves `/health` locally and reverse-proxies everything else to the internal ALB.
- The **app tier** (a tiny Python `http.server` on `:8080`) sits in private subnets behind the internal ALB. Its `/db` endpoint opens a TCP socket to RDS:3306 to prove end-to-end connectivity.
- The **database tier** is a private RDS MySQL instance reachable only from the app tier's security group.

### Deployment model

- **Two Auto Scaling Groups** (web and app), each with a target-tracking CPU policy at 50%, ELB health checks, and 300-second grace periods.
- **Rolling updates** configured with `AutoScalingRollingUpdate` (min 1 instance in service, batch size 1, 2-minute pause).
- **NAT Gateway** in `PublicSubnet1` so private instances can reach the internet for `dnf update` and SSM — without being reachable from it.
- **No SSH anywhere.** Administrative access is via **SSM Session Manager** using the `AmazonSSMManagedInstanceCore` managed policy and the instance profile attached to both launch templates.

**How to deploy (AWS Console):**
1. Open the **AWS CloudFormation** console and click **Create stack** (With new resources).
2. Upload `tier.yaml`.
3. Set parameters: `Environment` (`dev` / `test` / `prod`), `InstanceType`, `MinSize`, `MaxSize`, `DBInstanceClass`.
4. Acknowledge the IAM capability (`CAPABILITY_IAM`) — required because the template creates an IAM role and instance profile.
5. Create the stack and wait for `CREATE_COMPLETE` (takes ~10–15 minutes).
6. Open the **Outputs** tab. Browse to `LoadBalancerURL` and append `/db` to verify app-to-database connectivity (`Database reachable from app tier`).
7. To inspect instances without SSH, use **Systems Manager → Session Manager**.

### Outputs

| Output | What it gives you |
|---|---|
| `LoadBalancerURL` | Public URL of the web tier — try `/` (app page) and `/db` (connectivity test) |
| `InternalAppEndpoint` | Internal ALB DNS (not reachable from the internet) |
| `DatabaseEndpoint` | RDS MySQL endpoint (private) |
| `DatabaseSecretArn` | Secrets Manager ARN holding the auto-generated DB credentials |

---
## ✅ Best Practices Applied

### Part A(a) — Static site

1. **Origin Access Control (OAC):** The basic tutorial approach makes the S3 bucket public. I refactored the code to make the S3 bucket **completely private** and used the modern CloudFront OAC to securely authenticate and read from the bucket.
2. **Encryption & Versioning:** I added Server-Side Encryption (AES256) to the S3 bucket to secure data at rest, and enabled Bucket Versioning to allow for content rollback.
3. **HTTPS Enforcement:** I configured the CloudFront distribution to use `redirect-to-https` and enforced a minimum TLS protocol version (`TLSv1.2_2021`), securing data in transit.
4. **SPA Routing:** I added `CustomErrorResponses` so that 403 and 404 errors redirect to `/index.html` with a 200 status code, which is a best practice for modern static sites.

### Part A(b) — Three-tier

5. **Region-agnostic AMIs:** The AMI is resolved at deploy time via the public SSM parameter `/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64`, so the stack works in any region and always picks the newest Amazon Linux 2023 image. (The original template hard-coded a `us-east-1` AMI ID.)
6. **No SSH, SSM only:** Removed the `KeyName` parameter and the `0.0.0.0/0 → :22` rule. Instances use an IAM instance profile with `AmazonSSMManagedInstanceCore` and are accessed via Session Manager — no inbound management ports, no key pairs to leak.
7. **NAT egress for private subnets:** Added an EIP + NAT Gateway + private route table. Without this, private instances cannot reach the internet for `dnf` updates or the SSM service — a common trap in "minimal" three-tier templates.
8. **Secrets Manager for the DB password:** Replaced the plaintext `DBPassword` parameter with `ManageMasterUserPassword: true`. RDS generates and rotates the master password in Secrets Manager; the ARN is exposed as a stack output.
9. **Tier-chained security groups:** Each tier accepts traffic only from the tier immediately above it — `Internet → PublicALB → Web → InternalALB → App → DB`. No security group accepts traffic from `0.0.0.0/0` except the public ALB on port 80.
10. **Encrypted storage everywhere:** RDS uses `StorageEncrypted: true` on `gp3`. EC2 root volumes are `gp3`, 8 GB, encrypted, with `DeleteOnTermination: true`.
11. **IMDSv2 enforced:** Both launch templates set `MetadataOptions.HttpTokens: required`, which blocks IMDSv1 and mitigates SSRF-style credential theft.
12. **Auto Scaling instead of fixed instances:** Two ASGs with target-tracking CPU policies at 50%, ELB health checks, and rolling update policies — replacing the two hard-coded `AWS::EC2::Instance` resources.
13. **Conditional Multi-AZ for prod:** `MultiAZ: !If [IsProd, true, false]` — dev/test stay cheap, prod gets a standby.
14. **RDS durability:** `DeletionPolicy: Snapshot` and `UpdateReplacePolicy: Snapshot` retain a final snapshot if the stack is deleted. `BackupRetentionPeriod: 1`, `AutoMinorVersionUpgrade: true`, and CloudWatch log exports (`error`, `slowquery`) round it out.
15. **Built-in connectivity check:** The app tier exposes a /db endpoint that opens a TCP connection to the RDS endpoint on port 3306. Browsing to LoadBalancerURL/db and seeing “Database reachable from app tier” confirms that the request passed through the public ALB, nginx, the internal ALB and the app tier, and that the app tier can reach the database over the network (so the security group rules and routing work). It does not authenticate or run a query, so it tests network connectivity only, not database health.
16. **Load balancer hardening:** The public ALB enables `routing.http.drop_invalid_header_fields.enabled`. Both target groups use `/health` with tuned thresholds and a 30-second deregistration delay for fast rolling updates.
17. **Commented code & UI metadata:** The `tier.yaml` template is sectioned with comment banners per tier and uses `AWS::CloudFormation::Interface` `Metadata` so parameters are grouped logically in the console.

### Illities Analysis

**Part A(a) — Static site**
- **Security:** Data is encrypted at rest (AES256) and in transit (TLS 1.2 minimum, HTTPS redirect). The S3 bucket is not publicly accessible; only the specific CloudFront distribution can read it via OAC.
- **Scalability:** Amazon CloudFront is a CDN that automatically scales to handle any amount of traffic. S3 and CloudFront are managed services — no servers to provision or scale manually.
- **Performance:** CloudFront caches the website content at edge locations globally, reducing latency for users regardless of their geographic location.

**Part A(b) — Three-tier**
- **Security:** No SSH; management via SSM. Private subnets have no inbound internet path. Security groups chain strictly tier-to-tier. RDS password managed and rotated by Secrets Manager. Storage encrypted at rest (EBS + RDS). IMDSv2 enforced. Public ALB drops invalid headers.
- **Scalability:** Web and app tiers scale horizontally on CPU (target 50%). The stateless app tier means any instance can serve any request. ASG rolling updates replace instances without downtime.
- **Performance:** Each tier scales independently, so a busy web tier doesn’t force the app tier to grow with it. The web tier uses nginx purely as a lightweight reverse proxy, passing requests to the internal ALB. Traffic between the app tier and the database stays inside the VPC, so it takes a short private network path rather than crossing the internet. Both Auto Scaling Groups use target tracking on average CPU at 50%, which adds instances when load rises and leaves headroom before the instances saturate. Load balancers spread requests across both Availability Zones, and the instances use gp3 volumes.
- **Reliability / operability:** `DeletionPolicy: Snapshot` on RDS; CloudWatch log exports for `error` and `slowquery`; ELB health checks; `/db` self-test endpoint; parameter groups in the console.

---

## 📸 Screenshots

### 1. Live Website (Part A(a))
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

### 4. Three-Tier Stack (Part A(b))
----- to be done ------

---

## 🔗 Important

- **Collaborator:** Ruth.Lennon@atu.ie has been added to this repository.
- **Code commits:** All commits are my own work.
- **Lab write-up:** This README.md serves as my lab write-up.