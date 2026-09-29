# AWS Certified Solutions Architect – Professional (SAP-C03) Exam Scenarios Cheat Sheet

## Architectural Decision Matrix

| Scenario Requirement | Ideal Architecture / Solution Pattern | Key Distinctions & Anti-Patterns |
| :--- | :--- | :--- |
| **Centralized Egress Internet Traffic Control across 50+ VPCs** | AWS Transit Gateway + Central Inspection VPC with **AWS Network Firewall** or GWLB with 3rd party firewalls. Route all `0.0.0.0/0` outbound traffic from spoke VPCs to the TGW. | **Anti-Pattern:** Deploying individual NAT Gateways in every spoke VPC (costly and lacks centralized logging). |
| **Hybrid On-Prem DNS Resolution for Private AWS Hosted Zones** | Deploy **Route 53 Resolver Inbound Endpoints** for on-prem to AWS queries, and **Outbound Endpoints** with conditional forwarding for AWS to on-prem queries. | **Anti-Pattern:** Building custom EC2 BIND DNS proxies across VPCs (adds operational overhead). |
| **Secure Multi-Account API Access without exposing to Public Internet** | Private API Gateway + **Interface VPC Endpoints (AWS PrivateLink)** + Resource Policy restricting access to specific VPC Endpoints / VPCs. | **Anti-Pattern:** Using Public API Gateways with IP Whitelisting (still exposes public DNS endpoints). |
| **Zero-Downtime Database Migration with Minimal Latency** | **AWS Database Migration Service (DMS)** with Change Data Capture (CDC) enabled + **AWS Schema Conversion Tool (SCT)** for heterogeneous conversions. | **Anti-Pattern:** Dump and restore via `mysqldump` / `pg_dump` over VPN (causes unacceptable downtime). |
| **Centralized KMS Key Usage across Multiple Accounts** | **KMS Key Policy** in Account A granting `kms:Encrypt`, `kms:Decrypt`, and `kms:CreateGrant` permissions directly to Account B's IAM Role/Account Principal. | **Anti-Pattern:** Copying KMS keys across accounts (KMS key material cannot be directly exported or shared). |
| **Low-Latency Static & Dynamic Global Acceleration** | **AWS Global Accelerator** (for non-HTTP static Anycast IP requirements or raw TCP/UDP) or **Amazon CloudFront** (for Layer 7 edge caching & dynamic optimizations). | **Anti-Pattern:** Route 53 Latency-Based routing alone when clients require static IP whitelisting. |
| **S3 Enterprise Data Lake Access Governance** | **AWS Lake Formation** with fine-grained column/row-level access control integrated with IAM Identity Center. | **Anti-Pattern:** Managing hundreds of complex IAM policies and bucket policies manually per user/role. |
| **Near-Zero RTO/RPO Global Database Failover** | **Amazon Aurora Global Database** with storage-based replication (<1 second replication latency) and managed failover. | **Anti-Pattern:** RDS Cross-Region Read Replicas for critical databases where automated RTO demands < 1 minute. |

---

## High-Frequency SAP-C03 Scenario Traps & Solutions

### Scenario 1: The Multi-Account Audit Logging Pipeline
* **Requirement:** Aggregate CloudTrail logs, AWS Config data, and VPC Flow Logs from 100+ AWS accounts into a secure, tamper-proof, single-destination S3 bucket.
* **Solution Architecture:**
  1. Create an **Organizational CloudTrail** in the Management/Delegated Admin Account.
  2. Direct all logs to a centralized **Log Archive Account** S3 bucket.
  3. Attach an S3 Bucket Policy enforcing `aws:PrincipalOrgID` checks and requiring `s3:x-amz-acl: bucket-owner-full-control`.
  4. Enable **S3 Object Lock** in Compliance Mode on the destination bucket to enforce WORM rules.
  5. Enable **KMS Customer Managed Key (CMK)** encryption with a key policy granting access to `cloudtrail.amazonaws.com`.

### Scenario 2: Legacy Stateful Application Migration
* **Requirement:** Move a legacy Windows cluster requiring shared file access over SMB and Active Directory authentication to AWS with high availability.
* **Solution Architecture:**
  1. Deploy **Amazon FSx for Windows File Server** in a Multi-AZ deployment mode.
  2. Join the FSx file system to the existing on-premises Active Directory via **AWS Managed Microsoft AD** or Active Directory Connector.
  3. Mount storage targets to Windows EC2 instances distributed across multiple Availability Zones inside an Auto Scaling Group.

### Scenario 3: Real-Time Analytics Pipeline at Scale
* **Requirement:** Ingest millions of streaming data events per second with real-time processing and long-term archival in S3 for querying via Amazon Athena.
* **Solution Architecture:**
  1. Ingest streaming data using **Amazon Kinesis Data Streams**.
  2. Process real-time streaming data using **AWS Lambda** or **Amazon Managed Service for Apache Flink**.
  3. Buffer and convert stream data to columnar formats (Apache Parquet/ORC) using **Amazon Data Firehose**.
  4. Store the transformed data in an **Amazon S3** bucket partitioned by timestamp (`YYYY/MM/DD/HH`).
  5. Run **AWS Glue Crawlers** to update the Glue Data Catalog for efficient querying with **Amazon Athena**.

---

## Critical Exam Thresholds & Rules of Thumb

1. **Direct Connect Failover:**
   * **Active/Active:** Equal BGP costs (MED values) sent from on-prem.
   * **Active/Passive:** Use BGP Local Preference on the on-prem router, or set **AS Path Prepending** on the passive link.
2. **KMS Grants vs. Key Policies:** Use **Grants** when temporary access or delegate permission creation is required dynamically by programmatic services (e.g., EBS volume encryption via AWS Auto Scaling).
3. **AWS Organization Migration:** To move an account between Organizations, you must first remove it from the old Organization (requires full payment details on the member account), then send/accept an invite from the new Organization.

