# AWS Certified Solutions Architect - Professional (SAP-C03) Study Topics

## 1. Advanced Compute & Serverless Architectures

### EC2 Enterprise Scaling & Placement
* **EC2 Placement Groups:** Cluster, Spread, Partition
* **Auto Scaling Groups:** Mixed instance types, Spot pools, EC2 Launch Templates
* **Capacity Reservations**

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


## 1. Complex Multi-Account Networking & Hybrid Topologies

* **AWS Transit Gateway (TGW) Enterprise Routing:** Multi-TGW peering, route table isolation (Segmentation: Prod vs. Non-Prod vs. Shared Services), TGW Connect for SD-WAN integration, Route Analyzer.
* **Private Endpoint Management:** Large-scale AWS PrivateLink setup, Endpoint Services behind Network Load Balancers, Route 53 Private Hosted Zone sharing across multi-account VPCs via TGW or VPC Peering.
* **Hybrid Connectivity at Scale:** AWS Direct Connect (DX) with DX Gateways, Transit Virtual Interfaces (Transit VIF), Link Aggregation Groups (LAG), MACsec encryption, and BGP community routing.
* **Network Security & Perimeter Control:** AWS Network Firewall deployments (centralized vs. distributed inspection VPCs), AWS WAF enterprise rule management via AWS Firewall Manager, Route 53 Resolver DNS Firewall.


## 3. Containerization & Service Mesh Orchestration

* **Amazon EKS & ECS Scale Architectures:** ECS Capacity Providers (Fargate + Spot pools), EKS Managed Node Groups, Karpenter for real-time Kubernetes auto-scaling.
* **Service Networking:** Amazon VPC Lattice for zero-trust cross-account microservice networking, AWS App Mesh, and Amazon ECS Service Connect.
* **Container Identity & Security:** EKS IAM Roles for Service Accounts (IRSA), EKS Pod Identities, ECR cross-Region/cross-account image replication, and vulnerability scanning.

## 4. Generative AI, RAG & Purpose-Built Data Architectures

* **Amazon Bedrock Architecture:** Multi-tenant Bedrock deployment, Provisioned Throughput allocation, fine-tuning vs. Continued Pre-training, Bedrock Guardrails for PII/safety filtering.
* **Retrieval-Augmented Generation (RAG) at Scale:** Integration with vector databases (Amazon OpenSearch Serverless, Aurora PostgreSQL pgvector, Bedrock Knowledge Bases), embedding model pipelines, and hybrid search.
* **Agentic Workflows:** Amazon Bedrock Agents, multi-step tool execution, and human-in-the-loop (HITL) authorization integrations.




