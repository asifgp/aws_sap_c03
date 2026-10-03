
### Serverless Event-Driven Orchestration
* **AWS Step Functions:** Standard vs. Express Workflows, Distributed Map, error handling, sagas
* **AWS Lambda Advanced Patterns:** Concurrency limits, cold start mitigation, Lambda Layers, VPC integration, Provisioned Concurrency

## 2. Modern Application Architectures & Serverless Engineering

* **Advanced Event-Driven Patterns:** Amazon EventBridge (custom buses, cross-account event routing, EventBridge Pipes, API Destinations, Schema Registry), SQS FIFO deduplication and Dead Letter Queue (DLQ) automated replay.
* **Complex Workflow Orchestration:** AWS Step Functions (Standard vs. Express workflows, Distributed Map for high-throughput batch execution, Sagas pattern for distributed transactions, error handling/retries).
* **API Management Strategy:** API Gateway private endpoints, client certificates (mTLS), custom Lambda authorizers, usage plans, throttling algorithms (token bucket), and response caching strategies.

## 3. Things to recap:
* EventBridge Cross-Account Routing
* Priority Queue Processing Pattern: two distinct queues: Lambda event source mappings -> <High-Priority.fifo or Standard-Priority.fifo>.
* Dead-Letter Queue (DLQ) & Redrive Pattern: Asynchronous Error Recovery & Replay (SQS / SNS / EventBridge / Lambda DLQs)
Scenario / Use Case: Capturing, analyzing, and replaying failed processing messages without data loss.
Assumptions: Transient network errors or corrupt payload instances cause consumer invocation failures.
Pattern Considerations:
Configure SQS RedrivePolicy with maxReceiveCount (e.g., 3 to 5 attempts) pointing to a dedicated Dead-Letter Queue (DLQ).
Attach an AWS Lambda function or SQS Start Message Move Task (Redrive) to inspect and re-inject fixed messages back to the main queue after bug remediation.
Troubleshooting:
Issue: Infinite reprocessing loop between primary queue and DLQ.
Diagnosis: Validate that fixed target code addresses root exceptions before triggering redrive tasks. Ensure unparseable poison-pill messages are archived to S3 instead of re-queued indefinitely.

* Amazon Aurora Native Database Streams/CDC
* Amazon ElastiCache Serverless (Redis)
* Key Strategies to Improve Write Operations on DDB
  Batch Operations: Group multiple write requests into a single call using BatchWriteItem. This reduces network overhead and API call costs compared to individual PutItem or DeleteItem requests. A single batch handles up to 25 items or 16 MB of data.
  Write Sharding: Distribute high-volume write workloads across multiple partitions. Avoid hot partitions by appending a random suffix (e.g., numbers 1 to 100) or a calculated suffix (like an ID modulo) to your partition keys.
  Conditional Updates: Use ConditionExpressions to write or update data only when specific criteria are met. This prevents redundant writes and saves consumed Write Capacity Units (WCUs).
  Capacity Mode Tuning: Choose On-Demand mode for unpredictable or spiky workloads, or Provisioned mode with Auto Scaling for steady, predictable traffic. Pre-warm your tables if you expect a massive traffic surge.
  Attribute Minimization: Store only necessary attributes or pointer references (like storing large files in Amazon S3 and saving just the object URL in DynamoDB) to keep item sizes under 1 KB, reducing WCU consumption per write
* Step Functions Map State
* AWS Batch/AWS Step function
* EventBridge: Event Bus, Event Propagation, Subscription to Event Bus, Event Archiving, EventBridge Replay
* Attribute-Based Access Control (ABAC) IAM policies
* Partner Event Sourcing association inside EventBridge
* How to create a deadman's switch using DynamoDB TTL & event sourcing
* 


