1. Identity & Access Management at Scale

Organization-Level Identity Management: AWS IAM Identity Center (formerly AWS SSO) with external Identity Providers (SAML 2.0 / SCIM integration with Okta, Azure AD, Ping), Multi-Factor Authentication (MFA) enforcement.

IAM Delegation & Access Boundaries: AWS STS cross-account IAM roles, Session Policies, Permissions Boundaries, Attribute-Based Access Control (ABAC) using tags vs. Role-Based Access Control (RBAC).

Policy Maintenance: IAM Access Analyzer (policy generation, external access validation, automated reasoning), IAM policy conditions (aws:PrincipalOrgID, aws:SourceVpce).

2. Multi-Account Organizational Governance

AWS Organizations & Landing Zones: Organizational Units (OUs) structure, AWS Control Tower (Landing Zone setup, Guardrails/Controls: Mandatory, Strongly Recommended, Elective), Delegated Administrator for member accounts.

Policy Maintenance at Scale: Service Control Policies (SCPs) for organization-wide permission guardrails, Tag Policies, Backup Policies, AI service opt-out policies.

Compliance & Security Services: AWS Security Hub (CSPM, security standards compliance), AWS Config (Custom rules, conformance packs, multi-account aggregator, auto-remediation), AWS GuardDuty (runtime threat detection, malware protection), AWS Macie (data privacy scanning).

3. Complex Network Security & Data Protection

Perimeter Security: AWS WAF (Web ACLs, managed rule groups, rate-based rules), AWS Shield Advanced (DDoS mitigation), AWS Network Firewall, AWS Firewall Manager.

Data Encryption & Key Governance: AWS KMS (symmetric/asymmetric keys, multi-Region keys, key policies, automatic rotation, envelope encryption), AWS CloudHSM, AWS Certificate Manager (ACM / Private CA).

Secrets & Parameter Management: AWS Secrets Manager (automatic secret rotation, multi-Region replication), AWS Systems Manager Parameter Store (secure string encryption).

Post-Quantum Cryptography: Post-quantum hybrid key exchange patterns in AWS KMS (e.g., ML-DSA, ML-KEM).

