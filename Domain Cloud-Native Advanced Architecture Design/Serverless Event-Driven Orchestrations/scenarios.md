# Serverless Event-Driven Architecture on AWS

### 1. Asynchronous Task Processing Pattern
* **Topic:** Asynchronous File Processing Pipeline (S3 to EventBridge/SQS to Lambda)
* **Scenario / Use Case:** Media file processing (video encoding, thumbnail generation, PDF rendering) triggered upon uploading files to Amazon S3.
1. **Assumptions:**
   * Files uploaded to S3 are non-atomic and vary in size up to 5 GB.
   * Direct S3-to-Lambda integration can exhaust Lambda concurrency limits during unexpected spike bursts.
2. **Pattern Considerations:**
   * Use **Amazon EventBridge S3 Event Notifications** (or direct S3 Event Notifications to SQS) to decouple file uploads from compute processing.
   * Place an **Amazon SQS FIFO or Standard Queue** between S3 Event notifications and Lambda to throttle execution concurrency using `ReservedConcurrentExecutions` or `MaximumConcurrency`.
   * Ensure Lambda functions are idempotent using file checksums/S3 ETag keys stored in Amazon DynamoDB to avoid double-processing during retries.
3. **Troubleshooting:**
   * **Issue:** Missing S3 notifications or unconsumed messages piling up in SQS.
   * **Diagnosis:** Check SQS queue visibility timeout (must be $\ge 6 \times$ Lambda timeout) and inspect DLQ (Dead-Letter Queue) redrive policies. Verify S3 bucket notification permissions for EventBridge/SQS.

---

### 2. Fan-Out Message Distribution Pattern
* **Topic:** Order Fulfillment Fan-Out (SNS / EventBridge to SQS Queues)
* **Scenario / Use Case:** An E-commerce checkout system where a single "Order Placed" event must independently trigger Payment Processing, Inventory Reservation, and Email Notifications.
1. **Assumptions:**
   * Downstream microservices operate at different processing speeds and scale independently.
   * A failure in the Notification service should not block Payment or Inventory workflows.
2. **Pattern Considerations:**
   * Publish events to an **Amazon SNS Topic** or **Amazon EventBridge Custom Event Bus**.
   * Subscribe multiple dedicated **Amazon SQS Queues** (one per downstream service consumer) to the topic/bus.
   * Apply SNS/EventBridge **filter policies** so each consumer service receives only relevant payload sub-attributes.
3. **Troubleshooting:**
   * **Issue:** Downstream consumer services not receiving events or receiving incorrect payloads.
   * **Diagnosis:** Inspect SNS subscription raw message delivery settings and EventBridge pattern syntax. Validate subscription policy JSON for missing `sns:Subscribe` or `sqs:SendMessage` cross-service permissions.

---

### 3. Saga Pattern for Distributed Transactions
* **Topic:** Distributed Long-Running Orchestration (Step Functions Express/Standard)
* **Scenario / Use Case:** Booking a vacation package consisting of Flight, Hotel, and Car Rental, where failures require executing compensating undo actions.
1. **Assumptions:**
   * Distributed services (Flight API, Hotel API, Rental API) lack cross-database ACID transaction support.
   * Failures midway require rolling back previous state changes across remote microservices.
2. **Pattern Considerations:**
   * Implement **AWS Step Functions** to orchestrate execution flows via State Machines.
   * Use **Standard Workflows** for long-running executions with audit requirements or **Express Workflows** for high-volume, short-duration event transformations.
   * Define explicit `Catch` and `Retry` blocks per step, mapping errors to compensating undo Lambda functions (e.g., `CancelHotel` when `BookFlight` fails).
3. **Troubleshooting:**
   * **Issue:** Execution stuck indefinitely or compensating actions fail midway.
   * **Diagnosis:** Enable **AWS X-Ray** tracing across Step Functions state executions. Check execution history logs in CloudWatch Logs to identify state input payload mismatch or missing execution role permissions.

---

### 4. Claim-Check Pattern for Large Payloads
* **Topic:** Large Message Processing (SQS / EventBridge with S3)
* **Scenario / Use Case:** Ingesting large telemetry or enterprise data payloads exceeding the 256 KB payload limit of SQS/EventBridge.
1. **Assumptions:**
   * Incoming messages contain structured JSON or raw data exceeding 256 KB up to several megabytes.
2. **Pattern Considerations:**
   * **Producer Step:** Save the large payload directly into an Amazon S3 bucket and receive an object key URI.
   * **Event Step:** Send an event message containing only the S3 metadata/URI pointer (the "claim check") via SQS or EventBridge.
   * **Consumer Step:** The consumer Lambda function retrieves the message, fetches the complete payload from S3 using the pointer URI, processes it, and deletes or archives the S3 object.
3. **Troubleshooting:**
   * **Issue:** Consumer Lambda fails with `403 AccessDenied` or `404 NoSuchKey` when reading from S3.
   * **Diagnosis:** Check if S3 object upload completed before event publishing (eventual consistency/race condition). Ensure Lambda IAM execution role includes `s3:GetObject` permissions for the bucket path.

---

### 5. Circuit Breaker Pattern for Serverless Resilience
* **Topic:** Downstream Dependency Protection (Lambda + DynamoDB / ElastiCache)
* **Scenario / Use Case:** Protecting a legacy third-party REST API or non-scalable database (e.g., RDS) from being overwhelmed by serverless spikes.
1. **Assumptions:**
   * The downstream external endpoint has rate limits or slow responses during peak traffic.
   * Continual Lambda retries cause compounding latency and exhaustion of system resources.
2. **Pattern Considerations:**
   * Store circuit state (`CLOSED`, `OPEN`, `HALF-OPEN`) and error counts in **Amazon DynamoDB** or **Amazon ElastiCache for Redis**.
   * On invocation, Lambda checks the circuit state:
     * If `OPEN`, throw an immediate fallback error or return cached/mocked responses without making HTTP calls.
     * If `CLOSED`, proceed with the API call. If error rate threshold is breached within a rolling time window, flip state to `OPEN`.
3. **Troubleshooting:**
   * **Issue:** Circuit remains permanently `OPEN` or continuously flaps state between `OPEN` and `CLOSED`.
   * **Diagnosis:** Adjust the reset timeout duration and state failure threshold values in DynamoDB. Ensure race conditions are handled by setting DynamoDB Conditional Writes (`attribute_exists`, `atomic counters`).

---

### 6. Transactional Outbox Pattern
* **Topic:** Reliable Event Emission (DynamoDB Streams + EventBridge Pipe)
* **Scenario / Use Case:** Guaranteeing dual-writes (persisting application state into a database while reliably publishing an event to EventBridge) without distributed transactions.
1. **Assumptions:**
   * Network hiccups could cause state to be written to the database while event emission fails.
2. **Pattern Considerations:**
   * Save both business entity changes and outbox event records into an **Amazon DynamoDB** table inside a single atomic `TransactWriteItems` operation.
   * Enable **DynamoDB Streams** on the table to capture change events asynchronously.
   * Use **Amazon EventBridge Pipes** directly configured with the DynamoDB Stream as source and an EventBridge Event Bus or SQS as target without writing custom consumer Lambda glue code.
3. **Troubleshooting:**
   * **Issue:** Events are delayed or missing from the EventBridge Target.
   * **Diagnosis:** Check DynamoDB Stream iterator age metrics in CloudWatch. Verify EventBridge Pipes IAM role has `dynamodb:GetRecords`, `dynamodb:GetShardIterator`, and `events:PutEvents` rights.

---

### 7. Queue-Based Load Leveling Pattern
* **Topic:** API Throttling & Protection (API Gateway to SQS direct integration)
* **Scenario / Use Case:** High-throughput flash sales or ticketing APIs receiving burst requests that exceed backend processing throughput.
1. **Assumptions:**
   * Incoming traffic bursts can reach 50,000 requests/sec, while downstream processing services can only handle 1,000 requests/sec.
2. **Pattern Considerations:**
   * Direct service integration between **Amazon API Gateway** and **Amazon SQS** (bypassing Lambda at the API ingestion level).
   * API Gateway converts incoming HTTP requests straight into SQS `SendMessage` API calls and immediately returns HTTP 202 Accepted.
   * Consumer Lambda processes messages asynchronously from SQS using controlled batch sizes (`BatchSize`) and batching windows (`MaximumBatchingWindowInSeconds`).
3. **Troubleshooting:**
   * **Issue:** API Gateway returns HTTP 500 errors during burst ingestion.
   * **Diagnosis:** Verify API Gateway execution role credentials for SQS. Inspect API Gateway stage throttling/quota limits and SQS queue depth in CloudWatch metrics.

---

### 8. Event Streaming and Real-Time Analytics Pattern
* **Topic:** High-Velocity Event Ingestion (Kinesis Data Streams to Lambda / Managed Flink)
* **Scenario / Use Case:** Ingesting clickstream telemetry or IoT sensor data continuously for real-time dashboard analytics.
1. **Assumptions:**
   * Data flows continuously at high volume with strict sequential processing per device/user ID.
2. **Pattern Considerations:**
   * Use **Amazon Kinesis Data Streams** with Partition Keys set to unique device/user IDs to ensure ordering within shards.
   * Attach **AWS Lambda** as an Event Source Mapping using `ParallelizationFactor` (up to 10 concurrent executions per shard) to scale compute horizontally.
   * Configure `BisectBatchOnFunctionError` and `MaximumRecordAgeInSeconds` to prevent poison-pill records from blocking stream processing.
3. **Troubleshooting:**
   * **Issue:** High `GetRecords.IteratorAgeMilliseconds` metric on Kinesis stream.
   * **Diagnosis:** Scale out the stream by splitting Kinesis shards or increase `ParallelizationFactor` in Lambda Event Source Mapping. Optimize Lambda runtime execution time per record.

---

### 9. Scheduled Event & Cron Execution Pattern
* **Topic:** Serverless Batch Maintenance (EventBridge Scheduler to Lambda / Step Functions)
* **Scenario / Use Case:** Nightly database indexing, billing processing, or generating recurring automated reports.
1. **Assumptions:**
   * Recurring tasks require flexible schedule expressions (Cron/Rate) with high precision delivery.
2. **Pattern Considerations:**
   * Use **Amazon EventBridge Scheduler** instead of legacy EventBridge Rules for fine-grained schedule windows and flexible timezone support.
   * Set target directly to **AWS Step Functions** (for complex batch jobs) or **AWS Lambda** (for short tasks).
   * Configure retry policies with maximum event age and dead-letter queue (DLQ) destinations for failed schedule executions.
3. **Troubleshooting:**
   * **Issue:** Scheduled tasks do not trigger at expected execution times.
   * **Diagnosis:** Check target IAM invocation permissions in EventBridge Scheduler. Validate timezone settings and Daylight Saving Time (DST) cron configurations in the schedule expression.

---

### 10. Webhook Ingestion with Event Validation Pattern
* **Topic:** Third-Party Event Integration (API Gateway to EventBridge)
* **Scenario / Use Case:** Processing external webhooks safely (e.g., Stripe, GitHub, Shopify notifications).
1. **Assumptions:**
   * External webhooks can send invalid signatures, bad schemas, or malicious payloads.
2. **Pattern Considerations:**
   * Front ingestion endpoint with **Amazon API Gateway** with an **API Gateway Request Validator** matching JSON Schema models.
   * Pass validated requests to a lightweight **Lambda Authorizer / Signature Verification Function** that validates HMAC request headers against secrets stored in **AWS Secrets Manager**.
   * Validated events are published directly to an **Amazon EventBridge Custom Event Bus** for routing to downstream microservices.
3. **Troubleshooting:**
   * **Issue:** Webhook sender receives HTTP 504 Gateway Timeout errors.
   * **Diagnosis:** Decouple signature verification and processing; verify signature quickly in Lambda and publish to EventBridge asynchronously rather than synchronously processing business logic inside the webhook HTTP response path.

---

### 11. Command Query Responsibility Segregation (CQRS) Pattern
* **Topic:** Read/Write Decoupled Architecture (DynamoDB Streams + EventBridge + OpenSearch / Aurora Serverless)
* **Scenario / Use Case:** High-write transaction applications requiring complex multi-attribute search and analytics queries on read path.
1. **Assumptions:**
   * Write transactions hit a highly scalable key-value store (DynamoDB), but read models require relational/full-text search features (OpenSearch/Aurora).
2. **Pattern Considerations:**
   * Write commands modify state directly in **Amazon DynamoDB**.
   * **DynamoDB Streams** trigger an asynchronous event processor (Lambda or EventBridge Pipes).
   * Stream events project updated read-optimized views into **Amazon OpenSearch Serverless** or **Amazon Aurora Serverless v2**.
3. **Troubleshooting:**
   * **Issue:** Read model is significantly out of sync with write model (eventual consistency lag).
   * **Diagnosis:** Monitor Lambda Event Source Mapping batch size, failure retries, and write throttling metrics on target read databases.

---

### 12. FIFO Ordering & Deduplication Pattern
* **Topic:** Strict Order Sequence Processing (SQS FIFO + Lambda)
* **Scenario / Use Case:** Financial ledger updates, stock trading orders, or sequential inventory balance updates where sequence order is non-negotiable.
1. **Assumptions:**
   * Events must be processed strictly in the exact order they were generated.
   * Duplicate events must be filtered out within deduplication windows.
2. **Pattern Considerations:**
   * Use **Amazon SQS FIFO Queues** with explicit `MessageGroupId` attributes (grouping events belonging to the same entity/account) and `MessageDeduplicationId`.
   * Configure Lambda Event Source Mapping with single concurrency per Message Group ID to prevent parallel out-of-order execution.
3. **Troubleshooting:**
   * **Issue:** Queue throughput is throttled or consumer latency increases.
   * **Diagnosis:** Ensure high cardinality of `MessageGroupId` keys. Using a single `MessageGroupId` caps throughput at 300 msg/sec (or up to 7000 msg/sec with high throughput FIFO mode enabled).

---

### 13. Dynamic Router / Content-Based Routing Pattern
* **Topic:** Dynamic Payload Routing (EventBridge Rules / Step Functions Choice State)
* **Scenario / Use Case:** Routing incoming messages to different localized processing services based on event attributes (e.g., country code, customer tier).
1. **Assumptions:**
   * Processing logic differs substantially across message categories or regulatory regions.
2. **Pattern Considerations:**
   * Use **Amazon EventBridge Rules** with content filtering patterns on event detail JSON fields (e.g., `{"detail": {"region": ["US", "EU"]}}`).
   * Route directly to dedicated SQS queues or Step Function targets per region/tier without intermediary routing Lambda functions.
3. **Troubleshooting:**
   * **Issue:** Events fall through without matching any EventBridge rule.
   * **Diagnosis:** Implement a catch-all Rule in EventBridge with target pointing to a DLQ or CloudWatch Log Group to capture unmatched events. Verify JSON casing and data types in pattern expressions.

---

### 14. Priority Queue Processing Pattern
* **Topic:** Multi-Tier Task Scheduling (Multi-SQS Queues + Lambda Scaling)
* **Scenario / Use Case:** Processing workloads where Premium/VIP customer tasks must bypass Standard tier processing backlogs.
1. **Assumptions:**
   * Higher priority tasks must execute with lower latency during resource constraints.
2. **Pattern Considerations:**
   * Establish two distinct queues: `High-Priority.fifo` and `Standard-Priority.fifo`.
   * Configure producer microservices to route events to queues based on customer tier SLA metadata.
   * Set up dedicated Lambda event source mappings with higher reserved concurrency allocated to the `High-Priority` queue consumer.
3. **Troubleshooting:**
   * **Issue:** Standard queue messages experience starvation under high load.
   * **Diagnosis:** Configure fallback polling logic or allocate minimum baseline concurrency for standard queue consumers to preserve service baseline requirements.

---

### 15. Cross-Account Event Bus Federation Pattern
* **Topic:** Enterprise Multi-Account Architecture (EventBridge Cross-Account Routing)
* **Scenario / Use Case:** Centralized logging, auditing, or event sharing across decentralized AWS organization accounts.
1. **Assumptions:**
   * Individual business units operate in separate AWS accounts managed under AWS Organizations.
2. **Pattern Considerations:**
   * Central "Event Hub" account hosts an **Amazon EventBridge Bus** with EventBusPolicy granting `events:PutEvents` access to Spoke accounts.
   * Spoke accounts deploy EventBridge rules targeting the Central Bus ARN.
   * Central Bus routes cross-account events to security monitoring or analytical consumers.
3. **Troubleshooting:**
   * **Issue:** `AccessDeniedException` when publishing events from Spoke to Central account.
   * **Diagnosis:** Verify Resource-based Policy attached to Central EventBus permits the Spoke Account ID. Ensure target rules in Spoke accounts include an IAM role granting `events:PutEvents` on the target ARN.

---

### 16. Dead-Letter Queue (DLQ) & Redrive Pattern
* **Topic:** Asynchronous Error Recovery & Replay (SQS / SNS / EventBridge / Lambda DLQs)
* **Scenario / Use Case:** Capturing, analyzing, and replaying failed processing messages without data loss.
1. **Assumptions:**
   * Transient network errors or corrupt payload instances cause consumer invocation failures.
2. **Pattern Considerations:**
   * Configure SQS `RedrivePolicy` with `maxReceiveCount` (e.g., 3 to 5 attempts) pointing to a dedicated Dead-Letter Queue (DLQ).
   * Attach an **AWS Lambda function** or **SQS Start Message Move Task** (Redrive) to inspect and re-inject fixed messages back to the main queue after bug remediation.
3. **Troubleshooting:**
   * **Issue:** Infinite reprocessing loop between primary queue and DLQ.
   * **Diagnosis:** Validate that fixed target code addresses root exceptions before triggering redrive tasks. Ensure unparseable poison-pill messages are archived to S3 instead of re-queued indefinitely.

---

### 17. Strangler Fig Migration Pattern
* **Topic:** Legacy Monolith Deconstruction (API Gateway / EventBridge Pipes)
* **Scenario / Use Case:** Incrementally migrating legacy monolithic application logic to serverless event-driven microservices without downtime.
1. **Assumptions:**
   * Monolith remains live while microservices are extracted step-by-step.
2. **Pattern Considerations:**
   * Place **Amazon API Gateway** in front of the application domain.
   * Configure path-based routing rules: unmigrated endpoints route to legacy ALB/EC2 infrastructure; migrated path endpoints route to serverless EventBridge or Lambda functions.
   * Emit domain events via **Amazon EventBridge Pipes** directly from database streams to notify decoupled serverless consumers as legacy database models change.
3. **Troubleshooting:**
   * **Issue:** Inconsistent session state or double-writes during transition phase.
   * **Diagnosis:** Use feature flags and ensure read/write paths for specific entity domains are cut over atomically at the API Gateway routing level.

---

### 18. Database Change Data Capture (CDC) Pattern
* **Topic:** Relational DB to Event Streaming (Aurora Database Activity Streams / MSK / DynamoDB Streams)
* **Scenario / Use Case:** Publishing real-time events based on data updates inside Aurora or DynamoDB tables to update search indexes or caches.
1. **Assumptions:**
   * Direct code modifications to legacy application source to add message publishing logic are unfeasible or risky.
2. **Pattern Considerations:**
   * Enable **DynamoDB Streams** or **Amazon Aurora Native Database Streams/CDC**.
   * Use **Amazon EventBridge Pipes** with filtering rules to capture insert/modify operations.
   * Transform raw stream payload into standard CloudEvents specification formats using EventBridge Pipes Input Transformers before emitting to target event buses.
3. **Troubleshooting:**
   * **Issue:** High pipeline processing latency or missing schema attributes in downstream stream consumers.
   * **Diagnosis:** Verify DB stream retention settings (DynamoDB Streams retain for 24 hours). Ensure schema evolution changes are backward-compatible.

---

### 19. Cache-Aside Serverless Read Pattern
* **Topic:** Low-Latency Read Cache (API Gateway + Lambda + ElastiCache Serverless / DynamoDB DAX)
* **Scenario / Use Case:** Accelerating high-frequency database queries (e.g., product catalog lookup, user profiles) under intense read traffic.
1. **Assumptions:**
   * Read performance requires sub-millisecond responses, and database queries are costly.
2. **Pattern Considerations:**
   * Lambda first queries **Amazon ElastiCache Serverless (Redis)** or **DynamoDB Accelerator (DAX)**.
   * On **Cache Hit**: return cached item immediately.
   * On **Cache Miss**: read item from primary DynamoDB / Aurora table, store query result in ElastiCache with appropriate TTL (Time-To-Live), and return response.
3. **Troubleshooting:**
   * **Issue:** Cache stampede (thundering herd problem) on cache expiration or cold start.
   * **Diagnosis:** Implement probabilistic early expiration (XFetch algorithm) or locking mechanisms in Lambda code before executing database fallback queries.

---

### 20. Event Sourcing Pattern
* **Topic:** Immutable Audit Trail State Rebuilding (Kinesis / DynamoDB + EventBridge)
* **Scenario / Use Case:** Financial ledgers or healthcare tracking systems requiring full historical record reconstruction of state changes over time.
1. **Assumptions:**
   * System state should not be updated in-place; all changes are stored as immutable sequential events.
2. **Pattern Considerations:**
   * Store raw state transition events sequentially in **Amazon DynamoDB** (append-only table pattern) or **Amazon Kinesis Data Streams**.
   * Publish events to **Amazon EventBridge** to update current-state read views asynchronously.
   * Periodically create state snapshots in S3 to accelerate snapshot replay rebuilding routines.
3. **Troubleshooting:**
   * **Issue:** Replaying full event history to reconstruct state takes excessive time as event store grows.
   * **Diagnosis:** Implement snapshotting routines (e.g., nightly state aggregations) and start state rebuilds from the latest verified snapshot point rather than genesis events.

---

### 21. Real-Time Notification & WebSocket Push Pattern
* **Topic:** Bidirectional Event Delivery (API Gateway WebSockets + DynamoDB + EventBridge)
* **Scenario / Use Case:** Real-time dashboards, chat apps, or live order status push updates to browser clients.
1. **Assumptions:**
   * Frontend applications need instant server-to-client updates without continuous HTTP short polling.
2. **Pattern Considerations:**
   * Manage persistent connections using **Amazon API Gateway WebSocket API**.
   * Store active connection IDs inside an **Amazon DynamoDB** table via `$connect` / `$disconnect` route Lambda handlers.
   * Backend events emitted on **Amazon EventBridge** trigger a delivery Lambda function that queries active connection IDs from DynamoDB and pushes messages via API Gateway Management API (`@connections.postToConnection`).
3. **Troubleshooting:**
   * **Issue:** `410 GoneException` returned during WebSocket message post calls.
   * **Diagnosis:** Handle stale connections gracefully; when API Gateway returns `410 GoneException`, execute a DynamoDB cleanup call to remove the stale `connectionId`.

---

### 22. Heavy Batch File ETL Pipeline Pattern
* **Topic:** Distributed Large-Scale Processing (S3 Event -> EventBridge Pipes -> AWS Glue / Step Functions Map State)
* **Scenario / Use Case:** Ingesting daily multi-gigabyte CSV/Parquet extract files from enterprise third parties.
1. **Assumptions:**
   * Individual files contain millions of rows exceeding Lambda 15-minute execution runtime limits.
2. **Pattern Considerations:**
   * S3 file landing triggers **AWS Step Functions** with a **Distributed Map State**.
   * Step Functions natively reads CSV/JSON files directly from S3, splits data into chunks, and launches child executions concurrently (up to 10,000 parallel workers) using child Lambda functions or ECS Fargate tasks.
3. **Troubleshooting:**
   * **Issue:** Exceeding target service API limits or database connection pools during distributed Map processing.
   * **Diagnosis:** Configure `MaxConcurrency` controls on the Step Functions Distributed Map state to throttle parallel downstream worker executions.

---

### 23. Edge Compute Event Enrichment Pattern
* **Topic:** Global Request Customization (CloudFront + CloudFront Functions / Lambda@Edge)
* **Scenario / Use Case:** Dynamic geo-routing, localized payload modification, and real-time JWT token verification at the AWS edge network.
1. **Assumptions:**
   * Ingress request evaluation latencies must be under 10ms globally.
2. **Pattern Considerations:**
   * Use **CloudFront Functions** for lightweight HTTP request/response header manipulations, URL rewrites, and simple validations (sub-millisecond execution runtime).
   * Use **Lambda@Edge** for heavy operations requiring external network calls or database lookups (e.g., secret retrieval or complex request authorization).
3. **Troubleshooting:**
   * **Issue:** Elevated latency or execution failures at edge locations.
   * **Diagnosis:** Check CloudFront edge log groups in `us-east-1` (Lambda@Edge logs stream to CloudWatch Logs region nearest to the edge location). Keep package sizes minimal to optimize cold starts.

---

### 24. Event Archive and Replay Pattern
* **Topic:** Disaster Recovery & Event Auditing (EventBridge Archive & Replay)
* **Scenario / Use Case:** Replaying production events into dev/staging environments or recovering from downstream database corruption.
1. **Assumptions:**
   * Historical events need to be retained for compliance and system testing.
2. **Pattern Considerations:**
   * Enable **EventBridge Event Archiving** on custom or default event buses.
   * Set retention policies (e.g., indefinite or specific days) and schema match filters.
   * Initiate an **EventBridge Replay** specifying a start and end time window to re-process archived events through existing target rules during bug recovery.
3. **Troubleshooting:**
   * **Issue:** Replayed events trigger unintended side effects (e.g., duplicate customer emails or charges).
   * **Diagnosis:** Ensure consumer functions verify event header attributes (e.g., `replay-name` metadata) to suppress external side-effect operations during replay tasks.

---

### 25. Secure Multi-Tenant Event Isolation Pattern
* **Topic:** Multi-Tenant Event Separation (EventBridge / SQS with IAM Policy ABAC)
* **Scenario / Use Case:** SaaS platforms processing events for multiple customer tenants on shared serverless infrastructure.
1. **Assumptions:**
   * Tenant A must strictly never read, intercept, or process events generated by Tenant B.
2. **Pattern Considerations:**
   * Inject standard envelope metadata in event detail (e.g., `detail.tenantId`).
   * Apply **Attribute-Based Access Control (ABAC)** IAM policies and EventBridge rules restricting targets based on tag matching (`aws:PrincipalTag/TenantId` matching `detail.tenantId`).
   * Use separate SQS KMS Customer Managed Keys (CMKs) per high-security tenant to ensure cryptographic storage isolation.
3. **Troubleshooting:**
   * **Issue:** IAM authorization failures when consumers process events.
   * **Diagnosis:** Check principal session tags passed by STS when assuming runtime roles. Validate KMS Key Policy permissions for cross-tenant KMS alias calls.

---

### 26. External SaaS Integration Sync Pattern
* **Topic:** Partner System Event Ingestion (EventBridge Partner Event Sources)
* **Scenario / Use Case:** Ingesting events directly from third-party SaaS vendors (e.g., Zendesk, Auth0, PagerDuty, MongoDB Atlas) without custom webhook infrastructure.
1. **Assumptions:**
   * SaaS partner natively supports Amazon EventBridge Partner integrations.
2. **Pattern Considerations:**
   * Create a Partner Event Source association inside EventBridge.
   * Associate the source with a dedicated Event Bus in the target AWS account.
   * Define EventBridge rules matching partner event schemas to invoke internal compute or orchestration targets directly.
3. **Troubleshooting:**
   * **Issue:** Event bus remains in `PENDING` state after creation.
   * **Diagnosis:** Ensure the partner event source setup is explicitly accepted in the AWS Management Console / CloudFormation within the required time window.

---

### 27. GraphQL Subscriptions Serverless Pattern
* **Topic:** Event-Driven Real-Time Data GraphQL Push (AWS AppSync + EventBridge / DynamoDB Streams)
* **Scenario / Use Case:** Mobile client apps requiring dynamic real-time subscription feeds across granular GraphQL fields.
1. **Assumptions:**
   * System requires managed serverless GraphQL APIs with native subscription handling.
2. **Pattern Considerations:**
   * Deploy **AWS AppSync** as the GraphQL API gateway.
   * Microservices emit domain events to **Amazon EventBridge**.
   * EventBridge targets an AppSync **None Data Source** mutation, triggering GraphQL `$subscription` pushes automatically to connected client devices over WebSockets.
3. **Troubleshooting:**
   * **Issue:** AppSync subscription clients fail to receive real-time field mutations.
   * **Diagnosis:** Verify GraphQL schema auth directives (`@aws_iam`, `@aws_cognito_user_pools`). Ensure mutation target payload schema matches subscription filter requirements exactly.

---

### 28. Deadman's Switch / Heartbeat Monitoring Pattern
* **Topic:** Silent System Absence Detection (Step Functions Timers / DynamoDB TTL)
* **Scenario / Use Case:** Alerting administrators if an IoT device or external data feed stops sending expected heartbeat signals.
1. **Assumptions:**
   * Lack of an event occurrence within a specified time window represents a critical failure condition.
2. **Pattern Considerations:**
   * On receiving a heartbeat event, update an **Amazon DynamoDB** item with an updated `expiration_time` attribute.
   * Enable **DynamoDB Time-To-Live (TTL)** on the table.
   * If a device stops sending signals, the TTL expires, generating a DynamoDB Stream deletion event that invokes an alert notification Lambda/SNS pipeline.
3. **Troubleshooting:**
   * **Issue:** Alerts are delayed by hours after expected heartbeat failures.
   * **Diagnosis:** DynamoDB TTL deletions are asynchronous and can take up to 48 hours during background processes. For strict sub-minute SLA requirements, use **AWS Step Functions Wait states** or **EventBridge Scheduler** callbacks instead.

---

### 29. Rate-Limiting & Token Bucket Pattern
* **Topic:** Serverless API Ingress Throttling (API Gateway Usage Plans / DynamoDB Atomic Counters)
* **Scenario / Use Case:** Protecting downstream APIs from abuse while monetizing tiered API consumption.
1. **Assumptions:**
   * Clients are bound to strict request quotas per minute/day based on subscription tier.
2. **Pattern Considerations:**
   * Attach **API Gateway Usage Plans** and API Keys to define burst and rate limits at the gateway layer.
   * For dynamic rate-limiting inside Lambda compute, use **Amazon ElastiCache for Redis** or **DynamoDB atomic updates** (`ADD` expression) implementing a Token Bucket algorithm.
3. **Troubleshooting:**
   * **Issue:** API Gateway returns unexpected HTTP 429 Too Many Requests errors to legitimate traffic.
   * **Diagnosis:** Inspect burst vs rate limit settings in Usage Plans. Verify that client requests correctly include the `x-api-key` header value.

---

### 30. Canary & Blue/Green Serverless Deployment Pattern
* **Topic:** Zero-Downtime Event Consumer Deployments (AWS SAM / CodeDeploy + Lambda Aliases)
* **Scenario / Use Case:** Upgrading production Lambda function code or event source integrations safely without dropping events.
1. **Assumptions:**
   * Deploying breaking changes risks failing active production event processing streams.
2. **Pattern Considerations:**
   * Deploy serverless updates using **AWS SAM** or **AWS CloudFormation** configured with **AWS CodeDeploy**.
   * Set deployment preference types (e.g., `Canary10Percent10Minutes` or `Linear10PercentEvery1Minute`).
   * Configure CloudWatch Alarms monitoring Lambda error rates and latency metrics; if triggered during deployment, CodeDeploy automatically rolls back traffic alias pointing to the previous version.
3. **Troubleshooting:**
   * **Issue:** CodeDeploy deployment hangs or fails to complete traffic shift.
   * **Diagnosis:** Check pre/post-traffic Lambda validation hooks. Ensure CloudWatch alarms are not configured in an `INSUFFICIENT_DATA` state.

---

### 31. Idempotent Consumer Processing Pattern
* **Topic:** Event Duplicate Elimination (Lambda + DynamoDB / AWS Powertools)
* **Scenario / Use Case:** Preventing duplicate order charges or duplicate database records when messaging providers deliver events "at-least-once".
1. **Assumptions:**
   * Messaging networks (SQS Standard, SNS, EventBridge) guarantee at-least-once delivery, leading to duplicate event processing.
2. **Pattern Considerations:**
   * Extract unique event identifier (e.g., `eventId` or business key) in Lambda execution runtime.
   * Write event state to **Amazon DynamoDB** using conditional expression (`attribute_not_exists(eventId)`).
   * If the key exists, abort processing gracefully and return previous execution result (leveraging utilities like **AWS Lambda Powertools Idempotency module**).
3. **Troubleshooting:**
   * **Issue:** Incomplete event invocations leave lock records stuck in `IN_PROGRESS` status permanently.
   * **Diagnosis:** Configure appropriate TTL values on DynamoDB idempotency records so transient lock records auto-expire if Lambda panics or times out midway.

---

### Summary Verification Matrix

| Pattern Output | Core Event Services | Primary Resilience Control | Source Reference |
| :--- | :--- | :--- | :--- |
| **1. Asynchronous Task** | S3, EventBridge, SQS, Lambda | Visibility Timeout & Reserved Concurrency | AWS Decision Guides, Well-Architected |
| **2. Fan-Out Distribution** | SNS, EventBridge, SQS | Message Filtering & Decoupled Queues | AWS Well-Architected Framework |
| **3. Saga Orchestration** | Step Functions, Lambda | Retries, Catches & Compensating State Steps | AWS Serverless mistakes & patterns guide |
| **4. Claim-Check** | S3, SQS, Lambda | Offload large payload references | AWS Serverless Architectural Patterns |
| **5. Circuit Breaker** | Lambda, DynamoDB | Dynamic Circuit State Flips | AWS Lambda Anti-Patterns Documentation |
| **6. Transactional Outbox** | DynamoDB Streams, EventBridge Pipes | Dual-write elimination via stream capture | AWS Serverless Event-Driven Patterns |
| **7. Load Leveling** | API Gateway, SQS, Lambda | Asynchronous HTTP 202 Ingestion Buffer | AWS Serverless Architecture Guidance |
| **8. Real-Time Streaming** | Kinesis Streams, Lambda | ParallelizationFactor & BisectBatch onError | AWS Compute Decision Guides |
| **9. Scheduled Batch** | EventBridge Scheduler, Step Functions | Timezone-aware precision cron triggers | AWS Decision Guides & Well-Architected |
| **10. Webhook Ingestion** | API Gateway, Lambda, EventBridge | Edge schema validation & async decoupling | AWS Event-Driven Best Practices |
| **11. CQRS Pattern** | DynamoDB, EventBridge Pipes, OpenSearch | Asynchronous stream view materialization | AWS Serverless Reference Architectures |
| **12. FIFO Ordering** | SQS FIFO, Lambda | Group ID partitioning & strict deduplication | AWS SQS & Lambda Developer Guide |
| **13. Dynamic Router** | EventBridge Rules, SQS | Content-filtering rule targets without Lambda | AWS EventBridge Architectural Patterns |
| **14. Priority Queue** | SQS, Lambda | Concurrency reservation per priority stream | AWS Compute & Queue Best Practices |
| **15. Cross-Account Federation**| EventBridge Cross-Account Bus | Resource-based Bus Policies | AWS Well-Architected Multi-Account |
| **16. DLQ & Redrive** | SQS, Lambda | MaxReceiveCount & Automated Redrive Tasks | AWS Well-Architected Security & Error |
| **17. Strangler Fig** | API Gateway, EventBridge Pipes | Path-based routing & event-driven decoupling | AWS Microservices Migration Guide |
| **18. Change Data Capture** | DynamoDB Streams, EventBridge Pipes | Low-code stream event transformation | AWS EventBridge Pipes Specifications |
| **19. Cache-Aside Read** | API Gateway, Lambda, ElastiCache | TTL caching & lazy loading fallback | AWS In-Memory Caching Design Patterns |
| **20. Event Sourcing** | Kinesis, DynamoDB, EventBridge | Immutable log replay & snapshot aggregation | AWS Event-Driven Architecture Guide |
| **21. Real-Time Push** | API Gateway WebSockets, DynamoDB | Connection tracking & `@connections` API | AWS Serverless Developer Reference |
| **22. Heavy ETL Pipeline** | S3, Step Functions Map State | Distributed concurrent execution workers | AWS Step Functions Distributed Map Docs |
| **23. Edge Compute** | CloudFront, CloudFront Functions | Sub-10ms global edge payload customization | AWS Edge Services Documentation |
| **24. Event Archive/Replay** | EventBridge Archive, Event Bus | Selective timeline re-processing capabilities | AWS EventBridge Developer Guide |
| **25. Multi-Tenant Isolation** | EventBridge, SQS, KMS | Attribute-Based Access Control (ABAC) | AWS Security Best Practices Guide |
| **26. SaaS Integration** | EventBridge Partner Sources | Native third-party event bus routing | AWS EventBridge Integration Partners |
| **27. GraphQL Subscriptions** | AppSync, EventBridge, Lambda | Direct WebSocket pushed field updates | AWS AppSync Architectural Patterns |
| **28. Heartbeat Monitor** | DynamoDB TTL, Step Functions | Asynchronous absence detection alerting | AWS Serverless Design Patterns |
| **29. Ingress Rate Limiting**| API Gateway Usage Plans, ElastiCache | Usage Plan burst allocations & Token Buckets | AWS API Gateway Security Guidance |
| **30. Canary Deployment** | SAM, CodeDeploy, CloudWatch Alarms | Automated traffic shifting & metric rollback | AWS Serverless Application Model |
| **31. Idempotent Consumer** | Lambda, DynamoDB, AWS Powertools | Conditional Lock writes & TTL expiration | AWS Serverless Best Practices |
