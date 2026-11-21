# Flashcards — Module 17: Real-World Use Cases

> **Format:** Q = Question | A = Answer  
> **Total:** 25 flashcards

---

**Q:** What is the "Strangler Fig" pattern and how do you implement it on AWS?  
**A:** A pattern for gradually migrating from a monolith to microservices. New features are built as microservices; old features are migrated incrementally. On AWS: use API Gateway or Application Load Balancer to route traffic — `/nova-feature` → Lambda/ECS, `/legado/*` → EC2 monolith. Gradual, with no big-bang rewrite.

---

**Q:** How do you architect a highly available e-commerce application on AWS?  
**A:** Frontend: CloudFront + S3 (SPA). Backend: ALB + ECS/EC2 across 2+ AZs with Auto Scaling. Database: RDS Aurora Multi-AZ (writes) + Read Replicas (reads). Cache: ElastiCache (sessions, popular products). Cart/inventory: DynamoDB (high scale). Payments: SQS (exactly-once via deduplicationId). Images: S3 + CloudFront.

---

**Q:** What is the fan-out pattern and when do you use it on AWS?  
**A:** Fan-out: one event triggers multiple consumers in parallel. Implementation: SNS Topic → multiple SQS Queues (or Lambda). Each subscriber processes independently. Use when: one event needs to trigger multiple actions (order_placed → SQS_email + SQS_fulfillment + SQS_analytics + SQS_audit). Decouples the publisher from multiple consumers.

---

**Q:** How do you implement exactly-once processing for payments on AWS?  
**A:** SQS FIFO Queue with **MessageDeduplicationId** (hash of the transactionId). If the same transaction is sent twice with the same DeduplicationId within the 5-minute window, only one is delivered. In processing: an idempotent check in the database (check whether the transactionId was already processed before debiting).

---

**Q:** What architecture for an IoT pipeline with 100,000 devices?  
**A:** Devices → IoT Core (MQTT) → Rules Engine → multiple destinations: Kinesis Data Streams (streaming analytics), DynamoDB (latest readings), S3 (via Kinesis Firehose, historical data). For real-time alerts: Lambda Consumer on KDS. For historical analysis: Athena on S3. Visualization: QuickSight or Grafana.

---

**Q:** How do you use the CQRS (Command Query Responsibility Segregation) pattern on AWS?  
**A:** Separate writes and reads into different models/stores. Write path (Command): API Gateway → Lambda → DynamoDB (source of truth). Read path (Query): denormalized data in ElastiCache or OpenSearch for fast reads. Synchronization: DynamoDB Streams → Lambda → updates the read model. Write optimized for consistency; read optimized for performance.

---

**Q:** How do you architect a serverless Data Lake on AWS?  
**A:** Ingestion: Kinesis Firehose (streaming) + DataSync (batch). Storage: S3 with date partitioning. Catalog: Glue Crawler → Glue Data Catalog. Transform: Glue ETL Jobs (raw → curated → analytics zones). Query: Athena (SQL). Security: Lake Formation (column/row-level FGA). Visualization: QuickSight. Orchestration: Step Functions or MWAA.

---

**Q:** What is Event Sourcing and how do you implement it on AWS?  
**A:** Instead of saving the current state, save all the events that led to the state: `order_created`, `payment_processed`, `shipped`. The current state is rebuilt by replaying events. AWS: DynamoDB Streams or Kinesis as the event log. EventBridge to publish events. Advantage: full auditing, temporal queries (state at any point in time), debugging.

---

**Q:** How do you implement authentication with Cognito in a SPA + S3?  
**A:** (1) Cognito User Pool for authentication (username/password, social OAuth). (2) Cognito Identity Pool to exchange the User Pool token for temporary AWS credentials (assumed IAM Role). (3) The frontend uses the temporary credentials to call API Gateway (authenticated via a Cognito Authorizer) or access S3 directly. No authentication backend to manage.

---

**Q:** What architecture for a video streaming platform (Netflix-like)?  
**A:** Upload: S3 + pre-signed URL. Transcoding: S3 Event → SQS → EC2/ECS with MediaConvert or Elemental. CDN: CloudFront with S3 origins (processed video). DRM: CloudFront Signed URLs or Signed Cookies. Metadata: DynamoDB or Aurora. Recommendations: SageMaker Personalize. Analytics: Kinesis → Redshift/Athena.

---

**Q:** How do you implement multi-tenancy with per-tenant isolation on AWS?  
**A:** Strategies: **(1) Silo model:** one AWS account per tenant (maximum isolation, high administrative cost). **(2) VPC per tenant:** shared account, separate VPCs. **(3) S3 prefixes:** `s3://bucket/tenant-id/data/` with IAM conditions `Condition: {"StringLike": {"s3:prefix": "${aws:PrincipalTag/TenantId}/*"}}`. **(4) DynamoDB partition key** = tenantId. The choice depends on the isolation requirement.

---

**Q:** What is the typical serverless architecture pattern on the SAA-C03 exam?  
**A:** API GW → Lambda → DynamoDB (simple CRUD). Variations: + S3 pre-signed URLs for direct upload. + Cognito for auth. + SQS to decouple Lambda from heavy processing. + EventBridge for scheduling. + Step Functions for multi-step workflows. + Lambda@Edge for CDN personalization. + RDS Proxy for Lambda connecting to RDS (connection pooling).

---

**Q:** How do you detect fraud in real time with ML on AWS?  
**A:** Transactions → API GW → Lambda → Amazon Fraud Detector (or SageMaker endpoint) → result in <100ms. For historical transactions: Kinesis Data Streams → Lambda (calls the FM endpoint). Store decisions: DynamoDB. Periodically retrain models: Kinesis → S3 → SageMaker Training Job → update the endpoint. Audit trail: CloudTrail + DynamoDB.

---

**Q:** What is the "Cache-Aside" pattern and how do you implement it with ElastiCache?  
**A:** The application manages the cache manually. Read: check ElastiCache → hit: return the data. Miss: fetch from RDS → save to ElastiCache → return. Write: update RDS → **invalidate** the cache (delete the key). The opposite of Write-Through (cache always updated on write). Cache-Aside: lazy loading, less memory used, eventually consistent.

---

**Q:** How do you implement Blue/Green Deployment with zero downtime on AWS?  
**A:** Option 1: **ELB + ASG** — create a new Target Group (green) with the new version, gradually shift traffic in the ALB Listener Rule. Option 2: **CodeDeploy Blue/Green** — automates the process + automatic rollback. Option 3: **Lambda aliases + weighted routing** (10% v2 → 100% v2). Option 4: **Elastic Beanstalk** — swap environment URLs. Rollback: redirect traffic back.

---

**Q:** How do you architect high-performance product search in e-commerce?  
**A:** DynamoDB as the source of truth → DynamoDB Streams → Lambda → OpenSearch Service (indexing). The front end searches in OpenSearch (full-text, filters, autocomplete, facets). Benefit: OpenSearch returns in <10ms for complex queries. DynamoDB does not natively support full-text search. Data mapped with analyzed (tokenized) fields in OpenSearch.

---

**Q:** What are the main integration patterns between AWS services?  
**A:** **(1) Sync:** API Gateway → Lambda (client waits). **(2) Async:** API GW → Lambda → SQS → another Lambda (fire-and-forget). **(3) Fan-out:** SNS → multiple SQS. **(4) Streaming:** KDS → Lambda/KDA. **(5) Schedule:** EventBridge Scheduler → Lambda. **(6) Workflow:** Step Functions → multiple orchestrated services. Each pattern fits a use case.

---

**Q:** How do you ensure HIPAA compliance in an AWS architecture?  
**A:** (1) Use HIPAA-eligible services (EC2, RDS, S3, Lambda, etc. — see the BAA). (2) Sign a Business Associate Agreement (BAA) with AWS. (3) Encryption: KMS CMKs, encryption at rest and in transit. (4) Audit logging: CloudTrail + CloudWatch Logs. (5) Access control: IAM with the least-privilege principle. (6) Network: private VPC + VPN/DX for access. (7) Encrypted data backup.

---

**Q:** How do you implement a DLQ (Dead Letter Queue) in a serverless architecture?  
**A:** **Asynchronous** invocation (SNS, S3 Events): the DLQ is configured directly on the Lambda function (SNS retries 2x, Lambda retries 2x). Failed messages go to an SQS DLQ. **Event Source Mapping** (SQS): the DLQ is configured on the SQS queue (not on Lambda). SQS will retry up to maxReceiveCount, then move to the SQS DLQ. Preferred: use Lambda Destinations for asynchronous invocations (more flexible).

---

**Q:** What architecture for a multi-region backend application with low RTO (<5 min) and RPO (<1 min)?  
**A:** Active-Active: Route 53 with health checks + Latency/Failover Routing. Stateless API in both Regions (ECS/Lambda). Database: Aurora Global Database (replication <1s, automatic failover in ~1 min). Cache: ElastiCache Global Datastore (cross-region replica). S3 CRR for assets. Sessions: DynamoDB Global Tables. CloudFront with origins in multiple Regions.

---

**Q:** What is the Saga pattern and when do you use it on AWS?  
**A:** Manage distributed transactions across microservices. Each service performs a local transaction and publishes an event. If a later step fails: compensating transactions undo the previous steps. AWS: Step Functions (orchestration — knows the global state) or choreography via SNS/EventBridge (each service reacts to events). Use when: you need eventual consistency in multi-service transactions.

---

**Q:** How do you use SSM Parameter Store for application configuration?  
**A:** Parameter hierarchy: `/app/prod/database/password` (SecureString with KMS). Lambda/ECS fetches parameters at startup via the SDK. Advantages over env vars: (1) versioning; (2) encrypted secrets; (3) auditable (CloudTrail); (4) change without redeploy (Lambda can read on cold start). For automatic rotation: Secrets Manager.

---

**Q:** How do you implement serverless image processing on AWS?  
**A:** Upload: S3 pre-signed URL (directly from the browser to S3, without going through the backend). Upload trigger: S3 Event → Lambda. Lambda processes: Pillow/Sharp → resize/thumbnail → saves the result to S3. CDN: CloudFront distributes the processed images. For heavy processing: S3 Event → SQS → Lambda (throttled) or EC2 Spot via ASG. Metadata: DynamoDB.

---

**Q:** What is the typical architecture for a real-time notification system?  
**A:** Backend → EventBridge (or SNS) → multiple channels: (1) Email: SES. (2) SMS: SNS → SMS. (3) Push notifications: SNS → GCM/APNs. (4) In-app real-time: API GW WebSocket API + Lambda + DynamoDB (connection tracking). (5) Slack: Lambda → Slack API. Fan-out pattern for multiple simultaneous channels without coupling the backend.

---

**Q:** How does the exam test knowledge of multi-tier (3-tier) architectures?  
**A:** **Presentation:** CloudFront + S3 (SPA) or ALB. **Application:** EC2 ASG or ECS in private subnets (no direct internet access). **Data:** RDS in a private data subnet (accepts connections only from the application tier's SG). The exam tests: correct SG flow, that databases sit in private subnets, that the ALB sits in a public subnet, that application EC2 instances do not need a public IP.
