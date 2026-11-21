# Questions — Module 17: Real-World Use Cases

> **SAA-C03 Domain**: All domains (multi-service)  
> **Difficulty**: High

---

**1.** A video streaming company needs an architecture that supports 1 million concurrent users, serves videos with low latency globally, and scales automatically. Which AWS services make up the ideal architecture?

- A) EC2 + ELB + RDS
- B) CloudFront + S3 (static content) + ALB + Auto Scaling Group + Aurora (metadata)
- C) API Gateway + Lambda + DynamoDB
- D) Global Accelerator + EC2 + RDS Multi-AZ

<details><summary>Answer</summary>

**B** — CloudFront caches the videos at edge locations globally (low latency, reduces load on the origin). S3 stores the video files (unlimited scale, low cost). ALB + ASG scales the application tier (metadata, authentication, playlists). Aurora for transactional data with Read Replicas for high read throughput.

</details>

---

**2.** An e-commerce platform needs to process payments ensuring each transaction is processed exactly once, even in case of a retry. Which architecture guarantees this?

- A) API Gateway → Lambda → SQS Standard → processor Lambda
- B) API Gateway → Lambda → SQS FIFO → processor Lambda with an idempotency key
- C) API Gateway → Lambda → DynamoDB → SNS
- D) API Gateway → direct Lambda (explicit retry on the client)

<details><summary>Answer</summary>

**B** — SQS FIFO with MessageDeduplicationId = exactly-once delivery. In the processor, use the transaction ID as an Idempotency Key in DynamoDB (conditional put: only writes if it does not exist) to guarantee exactly-once processing even if the message is redelivered. Double protection: SQS FIFO (dedup) + DynamoDB conditional write.

</details>

---

**3.** A fintech startup needs an architecture that detects fraud in transactions in real time (latency < 100ms), processes 10,000 transactions per second, and maintains a complete audit trail. Which architecture implements this?

- A) S3 + Athena for batch fraud analysis (review after the day ends)
- B) API GW → Lambda → SageMaker endpoint (real-time ML models) → DynamoDB (result) + Kinesis → S3 + Athena (audit)
- C) RDS Aurora with fraud detection stored procedures
- D) EC2 with a fraud detection application running 24/7

<details><summary>Answer</summary>

**B** — For real time with ML: a SageMaker real-time endpoint returns a prediction in <100ms. The transaction also goes to Kinesis Firehose → S3 for auditing in Athena. DynamoDB stores the prevention results. Lambda orchestrates everything, scaling automatically to 10,000 TPS.

</details>

---

**4.** A multi-tenant SaaS company needs to isolate each customer's (tenant's) data in S3, ensuring that one tenant never accesses another's data. Which mechanism implements this?

- A) A single S3 bucket with per-tenant prefixes and a generic bucket policy
- B) One S3 bucket per tenant with a bucket policy + Cognito Identity Pool with per-tenant assume role
- C) Shared S3 with per-object ACLs for each tenant
- D) S3 Object Ownership with ACLs enabled per object

<details><summary>Answer</summary>

**B** — Strong isolation: bucket per tenant + Cognito Identity Pool (federates users to IAM roles specific to each tenant). The tenant bucket's bucket policy allows access only from the tenant's role. No cross-tenant access is possible. Alternative (A) with prefixes is easier to implement but less secure (a single misconfiguration could expose all tenants).

</details>

---

**5.** A company wants to migrate from a monolithic architecture to microservices with no downtime, using the Strangler Fig pattern. How do you implement this on AWS?

- A) Create all the microservices first and do a full cutover over a weekend
- B) ALB with path-based routing: new routes go to microservices, legacy routes go to the monolith; migrate gradually
- C) Route 53 with weighted routing splitting traffic between the monolith and microservices
- D) B is the correct Strangler Fig pattern; traffic is migrated incrementally per domain

<details><summary>Answer</summary>

**D** — Strangler Fig: ALB as the facade. New features and migrated routes go to microservices (ECS/Lambda); legacy features still go to the monolith. Gradually, all routes are migrated and the monolith is "strangled" (shut down). Route 53 weighted routing would be more suited to blue/green of the entire monolith, not incremental migration.

</details>

---

**6.** An IoT company has 100,000 devices sending telemetry every 10 seconds. The data needs to be processed in real time for alerts and stored for historical analytics. Which architecture should be used?

- A) IoT Core → Lambda (per message) → DynamoDB + S3
- B) IoT Core → Kinesis Data Streams → Lambda (real-time alerts) + Kinesis Firehose → S3 (storage) + Athena (analytics)
- C) EC2 MQTT broker → SQS → Lambda
- D) API Gateway → Lambda → DynamoDB Streams → Analytics

<details><summary>Answer</summary>

**B** — IoT Core: managed MQTT/HTTPS for 100k devices. Kinesis Data Streams absorbs 100k ÷ 10s = 10k messages/second. Lambda processes the stream for real-time alerts. Kinesis Firehose in parallel delivers to S3 in Parquet. Athena performs historical analytics. A serverless, managed, and scalable solution.

</details>

---

**7.** A marketplace company needs to notify 5 different teams (email, mobile push, audit, analytics, CRM) when an order is created. What is the most suitable pattern?

- A) A REST API that calls the 5 systems in sequence
- B) SNS topic (order created) → 5 SQS queues (one per system) → each system consumes its own queue (fan-out pattern)
- C) EventBridge with 5 rules pointing to each system
- D) B and C are both valid; SNS fan-out is simpler; EventBridge is better for complex conditional rules

<details><summary>Answer</summary>

**D** — SNS Fan-out: SNS → multiple SQS (or Lambda directly) in parallel. Simple and direct. EventBridge: better when different events need different filters/rules (e.g., only notify analytics for orders > $500). For the simple case described, SNS fan-out is sufficient.

</details>

---

**8.** A company needs a product search system that supports full-text search, category/price filters, and autocompletion. The catalog has 10 million products. Which AWS service should be used?

- A) DynamoDB with a GSI for category filters
- B) Amazon OpenSearch Service with product indexes
- C) Amazon Aurora with Full-Text Search
- D) Amazon Kendra

<details><summary>Answer</summary>

**B** — OpenSearch: a full-text search engine with compound filters, autocompletion (completion suggester), fuzzy matching, and relevance scoring. 10 million documents is feasible in a properly sized OpenSearch cluster. DynamoDB has no full-text search. Kendra is for enterprise document search, not a product catalog.

</details>

---

**9.** A company wants to build a serverless data lake where data engineers can do ETL in Python, analysts can run SQL queries, and data scientists can train ML models, all on the same data in S3. Which architecture?

- A) A permanent EMR cluster for all workloads
- B) S3 + Glue Data Catalog + Glue Studio (ETL) + Athena (SQL) + SageMaker (ML)
- C) Redshift as the center for all data
- D) Aurora + QuickSight for all cases

<details><summary>Answer</summary>

**B** — S3 as universal storage (raw and processed data). Glue Data Catalog: shared metastore. Glue Studio: serverless Python/Spark ETL. Athena: SQL over S3 (serverless). SageMaker: ML over S3 data. Each persona uses the ideal tool, and all access the same data in S3 via the Glue Catalog.

</details>

---

**10.** A web application needs to authenticate users with Google, Facebook, and email/password, and then access DynamoDB and S3 directly from the browser (client-side) without going through the backend. Which service implements this?

- A) IAM Users for each application user
- B) Amazon Cognito: User Pools (auth) + Identity Pools (temporary AWS credentials for client-side access)
- C) API Gateway with a Lambda Authorizer to validate OAuth tokens
- D) AWS SSO for web application users

<details><summary>Answer</summary>

**B** — Cognito User Pool: signup/login with email/password + federation with Google, Facebook (social). Identity Pool: exchanges the Cognito (or social) token for temporary AWS credentials (STS assume role). The browser can then access S3 and DynamoDB directly with the temporary credentials, without exposing permanent credentials.

</details>

---

**11.** A media company needs to process videos uploaded by users (transcoding to multiple formats and resolutions). The processing is heavy (30-60 minutes per video) and the volume is unpredictable. Which architecture?

- A) Lambda for transcoding (maximum timeout 15 min — insufficient)
- B) S3 event → SQS → ECS Fargate tasks (scaling based on queue depth) for transcoding
- C) On-Demand EC2 with Auto Scaling
- D) AWS Elemental MediaConvert (a managed service specific to transcoding)

<details><summary>Answer</summary>

**D** — AWS Elemental MediaConvert: a managed service specific to video transcoding. Supports multiple input/output formats, resolution presets, DRM, and captions. S3 → MediaConvert → S3. Serverless, pay per minute of transcoded output. Alternative: S3 → SQS → Fargate tasks with FFmpeg (B) — more flexible but more complex.

</details>

---

**12.** A company needs a cloud development environment where developers write, run, and debug code directly from the browser, without setting up local environments. Which AWS service?

- A) AWS Cloud9 (browser-based IDE with EC2 in the background)
- B) AWS CloudShell
- C) Amazon WorkSpaces
- D) AWS CodeStar

<details><summary>Answer</summary>

**A** — AWS Cloud9: a full IDE in the browser with a code editor, integrated terminal, debugger, and support for Lambda development. Runs on EC2 (or SSM for existing instances). Real-time collaboration. CloudShell (B) is only a terminal with no IDE. WorkSpaces (C) is a full virtual desktop.

</details>

---

**13.** A blog platform has predictable traffic spikes (article launches) lasting 2 hours. Most of the content is static. What is the most cost-effective architecture?

- A) Auto Scaling Group with CPU-based scaling
- B) CloudFront caching static content (articles, images) → ALB origin + small ASG (only for dynamic content)
- C) Lambda for all content (including dynamic HTML generation)
- D) EC2 with high memory for local in-memory caching

<details><summary>Answer</summary>

**B** — CloudFront absorbs most of the static content traffic without reaching the origin. Only non-cacheable requests (search, comments, user-specific) reach the backend. The ASG can be minimal. During spikes, CloudFront scales automatically (global infrastructure). Cost-effective because it removes most of the load from the backend.

</details>

---

**14.** A company needs to ensure it can restore its application in a new Region in under 1 hour after a regional failure. Which combination of strategies should be implemented?

- A) Daily EBS snapshots + AMIs copied to a second Region
- B) Multi-region AMIs + Aurora Global DB + S3 CRR + Route 53 health check failover + IaC (CloudFormation)
- C) Only S3 CRR for data backup
- D) AWS Backup with cross-region backup for all resources

<details><summary>Answer</summary>

**B** — For RTO < 1 hour: pre-copied AMIs (EC2 ready to launch), Aurora Global DB (replicated DB with RPO < 1s), S3 CRR (replicated data), Route 53 automatic failover, and CloudFormation to quickly provision the infrastructure. AWS Backup (D) is good for backups but does not coordinate an orchestrated failover.

</details>

---

**15.** A healthcare company needs to store medical records with strict controls: only authorized doctors access their patients' records, auditing of all access, encryption with customer-managed keys, and HIPAA compliance. Which architecture?

- A) RDS MySQL with a user per doctor
- B) S3 with AES256 + IAM policies per doctor
- C) DynamoDB (data) + KMS CMK (encryption) + IAM with resource-based conditions + CloudTrail Data Events (auditing) + AWS Artifact (HIPAA certification)
- D) Aurora with SSL + Cognito User Pools

<details><summary>Answer</summary>

**C** — Complete HIPAA compliance: DynamoDB with a KMS CMK (customer-managed key = full control of encryption); IAM with conditions (`dynamodb:LeadingKeys` to restrict by the doctor-patient's patient_id); CloudTrail Data Events: records all reads in DynamoDB; AWS Artifact: obtains the BAA (Business Associate Agreement) documents required for HIPAA.

</details>
