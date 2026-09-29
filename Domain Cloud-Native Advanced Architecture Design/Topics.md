# AWS Certified Solutions Architect - Professional (SAP-C03) Study Topics

## 1. Advanced Compute & Serverless Architectures

### EC2 Enterprise Scaling & Placement
* **EC2 Placement Groups:** Cluster, Spread, Partition
* **Auto Scaling Groups:** Mixed instance types, Spot pools, EC2 Launch Templates
* **Capacity Reservations**

### Serverless Event-Driven Orchestration
* **AWS Step Functions:** Standard vs. Express Workflows, Distributed Map, error handling, sagas
* **AWS Lambda Advanced Patterns:** Concurrency limits, cold start mitigation, Lambda Layers, VPC integration, Provisioned Concurrency

### API Management
* **Amazon API Gateway:** REST vs. HTTP APIs, private endpoints, custom authorizers, usage plans, throttling, caching
* **AWS AppSync:** GraphQL APIs, subscriptions, conflict resolution

### Messaging & Decoupling
* **Amazon EventBridge:** Event Bus, rules, Schema Registry, Pipes, cross-account/cross-Region routing
* **Amazon SQS:** FIFO vs. Standard, Dead Letter Queues (DLQ), message deduplication
* **Amazon SNS:** Fan-out pattern, message filtering

---

## 2. Advanced Containerization & Microservices

### Container Orchestration
* **Amazon ECS:** EC2 vs. Fargate capacity providers, task definitions, task placement strategies
* **Amazon EKS:** Managed node groups, Fargate, service accounts, VPC CNI plugin, IAM Roles for Service Accounts (IRSA)

### Service Networking & Discovery
* Service mesh and inter-service connectivity via **Amazon VPC Lattice** and **Amazon ECS Service Connect**

### Container Security & Registries
* **Amazon ECR:** Image scanning, cross-account replication, lifecycle policies
* Container image signing and security verification

---

## 3. Generative AI & AI/ML Integrations

### Amazon Bedrock Architecture
* Foundation Model (FM) selection
* Model customization (fine-tuning, Continued Pre-training)
* Provisioned Throughput

### Agentic AI & RAG Patterns
* **Retrieval-Augmented Generation (RAG):** Vector databases (Amazon OpenSearch Serverless, Amazon Aurora PostgreSQL `pgvector`)
* Amazon Bedrock Agents and Amazon Bedrock AgentCore

### Guardrails & Governance
* **Bedrock Guardrails:** Content filtering, PII masking, safety controls, human-in-the-loop (HITL) workflows

### AI Observability
* Monitoring model performance, latency, hallucination detection metrics, and token usage

---

## 4. Advanced Storage & Purpose-Built Databases

### Object & Block Storage at Scale
* **Amazon S3:** Intelligent-Tiering, Object Lambda, Cross-Region/Same-Region Replication (CRR/SRR), S3 Express One Zone
* **Amazon EBS:** gp3, io2 Block Express, Multi-Attach, Fast Snapshot Restore

### High-Performance & Shared File Systems
* **Amazon FSx Variants:** FSx for Windows File Server, FSx for Lustre, FSx for NetApp ONTAP, FSx for OpenZFS
* **Amazon EFS:** Performance/throughput modes, cross-account mounts

### Relational & Distributed Databases
* **Amazon Aurora:** Global Databases, Aurora Serverless v2, read replicas, multi-master
* **Amazon RDS:** Multi-AZ DB Cluster, Proxy connection pooling

### NoSQL & In-Memory Datastores
* **Amazon DynamoDB:** Global Tables, Streams, On-Demand vs. Provisioned, DynamoDB Accelerator (DAX)
* **Amazon ElastiCache:** Redis OSS/Memcached, cluster mode, cross-AZ failover
* **Amazon MemoryDB for Redis**





