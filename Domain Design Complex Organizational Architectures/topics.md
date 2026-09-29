## 1. Enterprise Multi-Account Structure & Governance

* **AWS Organizations Architecture:** Hierarchical Organizational Unit (OU) design strategies (Core OUs, Workload OUs, Sandbox OUs, Policy Staging OUs), account creation automation via Organizations API.
* **Service Control Policies (SCPs):** Complex SCP guardrails, policy evaluation logic (Explicit Deny vs. Allow), restrictive SCPs for Region restriction, service disabling, and compliance boundaries.
* **AWS Control Tower & Landing Zones:** Account Factory customization (AFC), Landing Zone drift detection and proactive/detective guardrails (Controls), AWS Control Tower controls for security standards.
* **Delegated Administration:** Configuring delegated administrators for services like AWS IAM Identity Center, AWS GuardDuty, AWS Security Hub, AWS Config, and AWS Backup to preserve root account isolation.

## 2. Multi-Account Identity & Access Management

* **IAM Identity Center (formerly AWS SSO):** External Identity Provider (IdP) integration via SAML 2.0 and SCIM provisioning (Okta, Entra ID, Ping Identity), Permission Sets configuration, ABAC vs. RBAC routing.
* **Cross-Account Access Patterns:** AWS STS assume-role mechanics, ExternalId condition key usage to prevent the confused deputy problem, cross-account resource policies (S3, KMS, SQS, EventBridge).
* **Advanced Policy Evaluation:** Policy evaluation logic combining SCPs, IAM Resource-based policies, IAM Permission Boundaries, Session Policies, and Endpoint Policies.
* **Least Privilege Maintenance:** IAM Access Analyzer for active policy generation, automated reasoning for S3 bucket public access checks, and un-used permission identification.

## 3. Service Sharing & Resource Management

* **AWS Resource Access Manager (RAM):** Sharing Transit Gateways, Subnets, License Manager configurations, Route 53 Resolver Rules, and Aurora DB clusters across accounts.
* **Service Catalog Enterprise Governance:** Curating portfolios of pre-approved CloudFormation templates, access control via launch constraints, multi-account sharing, and Tag Option Library.

