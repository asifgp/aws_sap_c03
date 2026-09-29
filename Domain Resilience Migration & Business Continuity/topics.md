## 1. High Availability & Disaster Recovery (DR)

* **Resilience Engineering:** Designing against target RPO (Recovery Point Objective) and RTO (Recovery Time Objective).
* **Disaster Recovery Patterns:** Backup & Restore, Pilot Light, Warm Standby, Multi-Region Active-Active / Active-Passive failover.
* **Automated Resilience Validation:** Testing with AWS Fault Injection Service (FIS) (chaos engineering), assessment via AWS Resilience Hub, traffic management via AWS Application Recovery Controller (ARC).
* **Backup Management:** Centralized multi-account backups using AWS Backup, cross-account/cross-Region backup vault replication, Amazon S3 Versioning, and Object Lock.

## 2. Workload Migration Strategies & Execution

* **Migration Frameworks:** The 7 Rs Strategy (Rehost, Relocate, Replatform, Refactor/Rearchitect, Repurchase, Retain, Retire).
* **Discovery & Assessment:** AWS Application Discovery Service, AWS Migration Hub, Migration Evaluator.
* **Server & Data Migration Tools:** AWS Application Migration Service (MGN), AWS Database Migration Service (DMS - Change Data Capture / CDC, Schema Conversion Tool - SCT), AWS DataSync (S3, EFS, FSx transfers), AWS Snowball Edge for offline massive migrations.
* **Modernization & Refactoring:** Application modernization using the Strangler Fig pattern, monolith-to-microservices transformation.

## 1. Disaster Recovery (DR) & Multi-Region Architectures

### DR Strategies & RPO/RTO Metrics:
* **Backup and Restore:** (Hours/Days)
* **Pilot Light:** (Minutes)
* **Warm Standby:** (Seconds)
* **Multi-Region Active-Active / Active-Passive:** (Near Zero)

* **Cross-Region Replication Patterns:** Aurora Global Databases (storage-level replication, write forwarding), DynamoDB Global Tables (multi-Region active-active), Amazon S3 Cross-Region Replication (CRR) with KMS key mapping.
* **Multi-Region Failover Orchestration:** Route 53 Application Recovery Controller (ARC), Routing Controls, Health Checks, Global Accelerator Anycast IP failover.

## 2. Centralized Backup & Business Continuity

* **AWS Backup Enterprise Management:** Multi-account multi-Region backup policies, cross-account backup vault sharing, AWS Backup Vault Lock (WORM compliance), continuous backups with Point-in-Time Restore (PITR).
* **Resilience Validation:** Chaos engineering via AWS Fault Injection Service (FIS), resilience posture modeling via AWS Resilience Hub.


