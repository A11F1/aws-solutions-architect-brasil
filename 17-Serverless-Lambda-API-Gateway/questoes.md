# Questions — Module 11: Serverless Lambda and API Gateway

> **SAA-C03 Domain**: Design High-Performing Architectures | Secure Architectures  
> **Difficulty**: Medium-High

---

**1.** A Lambda function is invoked by an API Gateway and needs to access an RDS database in a private subnet. The function starts timing out. What is the most likely cause?

- A) The Lambda does not have IAM permission to access RDS
- B) The Lambda in a VPC has no outbound route to the internet, but the RDS endpoint requires external DNS resolution
- C) The Lambda in a VPC cannot connect to RDS because it is not in the same subnet
- D) The Lambda needs an Elastic IP to access resources inside the VPC

<details><summary>Answer</summary>

**A** — A Lambda in a VPC needs permission in the RDS Security Group for the database port (e.g., MySQL 3306). The most likely cause of the timeout is the RDS SG not allowing traffic from the Lambda's SG. Note: the Lambda in a VPC can be in a different subnet than RDS as long as the SG allows it and there is a route between subnets of the same VPC.

</details>

---

**2.** A company has a Lambda function that consumes messages from an SQS queue. The function has a 30-second timeout. Messages start being processed more than once. Which queue setting should be adjusted?

- A) Increase the Message Retention Period
- B) Set the Visibility Timeout to at least 6x the Lambda timeout (180 seconds)
- C) Enable Long Polling with a wait time of 20 seconds
- D) Reduce the Batch Size to 1

<details><summary>Answer</summary>

**B** — The SQS queue's Visibility Timeout must be ≥ 6× the Lambda timeout. If the Lambda takes 30s to process, the VT must be ≥ 180s. If the VT expires before the Lambda finishes, the message reappears in the queue and another worker will process it, causing duplication.

</details>

---

**3.** An API Gateway REST API has a Lambda backend that sometimes raises exceptions. The company wants to return HTTP 500 for internal errors and HTTP 400 for validation errors. Which configuration allows this?

- A) Lambda Proxy Integration with a custom return in the handler
- B) Lambda Custom Integration with Integration Response mapping error regex to status codes
- C) API Gateway Usage Plans with an error quota
- D) CloudWatch Alarms to detect errors and invoke a fallback Lambda

<details><summary>Answer</summary>

**A** — With Lambda Proxy Integration, the Lambda has full control over the response status code. The handler returns `{ statusCode: 400/500, body: ... }` and API Gateway passes exactly that status code. With Lambda Custom Integration (B), you can use an error message regex, but it is more complex.

</details>

---

**4.** A Lambda function needs to process 1,000 messages per second from an SQS Standard queue. Which configuration maximizes processing throughput?

- A) Increase the Lambda timeout to 15 minutes
- B) Configure multiple event source mappings for the same queue
- C) Increase the Batch Size (up to 10,000) and increase the Lambda's reserved concurrency
- D) Migrate to SQS FIFO for higher throughput

<details><summary>Answer</summary>

**C** — A larger Batch Size = fewer Lambda invocations for the same volume, reducing overhead. Increasing concurrency allows multiple batches to be processed in parallel. SQS FIFO has lower throughput (300 TPS without batching), not higher. One Event Source Mapping per queue.

</details>

---

**5.** The company needs a Lambda function to keep a connection to an RDS database between invocations to reduce reconnection overhead. How should this be implemented correctly?

- A) Use `/tmp` to store the connection object
- B) Initialize the connection in the initialization code outside the handler; the connection persists in the execution environment between warm invocations
- C) Use a Layer to share the connection between Lambda functions
- D) Use ElastiCache to store the connection string

<details><summary>Answer</summary>

**B** — Code outside the handler runs only when the execution environment is initialized (cold start) and persists across "warm" invocations within the same environment. Initializing `db_connection = get_db_connection()` outside the handler = reconnecting only on cold starts. `/tmp` is file storage, not for Python objects.

</details>

---

**6.** A company uses API Gateway + Lambda for its API. Traffic increases 10x during promotional events and throttling starts to occur. What should be done to protect the backend while allowing a reasonable burst?

- A) Enable API Gateway Cache with a 5-minute TTL
- B) Configure Usage Plans with throttling per API Key and increase the Lambda concurrency limit
- C) Migrate from REST API to HTTP API (lower cost)
- D) Add SQS between API Gateway and Lambda to buffer requests

<details><summary>Answer</summary>

**D** — For extreme spikes: API Gateway → SQS → Lambda processes at the pace it can handle. This eliminates throttling because the API returns 200 immediately and the Lambda consumes the queue. For synchronous APIs (immediate response required), increasing Lambda concurrency and configuring throttling in API GW are the alternatives.

</details>

---

**7.** A Lambda function processes images from S3 but frequently exceeds the processing time. The processed image must be saved back to S3. Which Lambda configuration is most suitable?

- A) Increase memory to 10 GB and timeout to 15 minutes; use /tmp for intermediate processing
- B) Use a Lambda container image (up to 10 GB) with a processing library
- C) Split into two Lambdas: one for download, another for processing, coordinated by Step Functions
- D) Use EC2 Spot for heavy image processing

<details><summary>Answer</summary>

**A** — Lambda supports up to 10 GB of memory and a 15-minute timeout with up to 10 GB of `/tmp`. For processing large images, increase memory (which also increases CPU) and use `/tmp` for temporary work. A container image (B) solves the package size problem, not the timeout.

</details>

---

**8.** An architect needs to create an HTTPS endpoint that triggers a Lambda every time a CSV file is dropped into S3. What is the simplest path?

- A) API Gateway + Lambda + S3 Event Notification
- B) S3 Event Notification directly triggers the Lambda (no API Gateway)
- C) EventBridge Rule for an S3 event → Lambda
- D) SQS + S3 Event Notification → Lambda ESM

<details><summary>Answer</summary>

**B** — S3 can invoke Lambda directly via an S3 Event Notification configured on the bucket (no additional cost, no API GW needed). It is the simplest path. EventBridge (C) works but is more complex for this simple case.

</details>

---

**9.** An API Gateway REST API has 50 endpoints and processes 100,000 requests per day. The team wants to reduce cost. What should they analyze?

- A) Migrate all endpoints to HTTP API (~71% cost reduction per request vs REST API)
- B) Enable caching on GET endpoints with rarely changing data
- C) Move the API inside a VPC (VPC-only endpoint)
- D) A and B are valid complementary strategies

<details><summary>Answer</summary>

**D** — HTTP API costs less per request (and supports most use cases); caching reduces backend calls (and total cost). The migration should evaluate which REST API features are needed (native WAF, request validation, usage plans — not available in HTTP API).

</details>

---

**10.** A Java Lambda has 8-second cold starts causing a poor experience for API users. What is the most efficient solution to eliminate cold starts without changing the runtime?

- A) Increase the Lambda memory from 512 MB to 10 GB
- B) Lambda SnapStart — available for Java 11+ (snapshots of the initialized runtime)
- C) Add the Lambda to a placement group
- D) Use a Scheduled EventBridge rule to "pre-warm" the Lambda by calling it every minute

<details><summary>Answer</summary>

**B** — SnapStart for Java 11+: AWS takes a snapshot of the execution environment after the init phase and restores it instead of initializing from scratch, reducing cold start to sub-second. "Pre-warming" (D) works but is a hack; Provisioned Concurrency is the official alternative if SnapStart is not available.

</details>

---

**11.** A company uses Lambda to process asynchronous financial transactions. They need to ensure no transaction is lost even if the Lambda fails. What should be configured?

- A) Increase the Lambda timeout to 15 minutes
- B) Configure Lambda Destinations with an On Failure destination to an SQS DLQ
- C) Enable Enhanced Monitoring on Lambda
- D) The Lambda is configured with SQS as an ESM — SQS itself retains messages until maxReceiveCount

<details><summary>Answer</summary>

**D** — With SQS as an ESM, messages stay in the queue until they are successfully processed or reach maxReceiveCount. After maxReceiveCount attempts, they go to the DLQ. This ensures no message is lost. Lambda Destinations (B) are for direct asynchronous invocations (not ESM).

</details>

---

**12.** An API Gateway exposes sensitive data and the company requires all requests to be authenticated with JWT tokens from its existing identity provider (not Cognito). Which configuration should be used?

- A) Cognito User Pool Authorizer (would require migrating the Identity Provider)
- B) Lambda Authorizer with a bearer token — the Lambda validates the JWT and returns an IAM policy
- C) IAM Authorization with SigV4
- D) API Key with a Usage Plan

<details><summary>Answer</summary>

**B** — Lambda Authorizer with TOKEN type: receives the JWT in the Authorization header, the Lambda validates the token against the existing Identity Provider (no migration) and returns an IAM policy (Allow/Deny). A cache of up to 3600s avoids verification on every request.

</details>

---

**13.** A Lambda needs to access a secret in Secrets Manager. What is the most secure way to grant this permission?

- A) Create an IAM user with access to Secrets Manager and embed the credentials in the Lambda code
- B) Add the `secretsmanager:GetSecretValue` permission to the Lambda's execution role
- C) Store the secret in a Lambda environment variable (encrypted with KMS)
- D) Use the VPC to access Secrets Manager without IAM

<details><summary>Answer</summary>

**B** — Lambda IAM Role (Execution Role): the Lambda automatically assumes the role during execution. Adding `secretsmanager:GetSecretValue` to the role is the secure standard. Never use an IAM user with hardcoded credentials (A). An environment variable (C) is not as dynamic as fetching directly from Secrets Manager.

</details>

---

**14.** A company has a batch job that must run every hour and process data from S3. The job takes 5-10 minutes. Which solution is the most cost-effective?

- A) EC2 t3.medium running 24/7 with cron
- B) EventBridge Rule (rate: 1 hour) → Lambda with a 15-minute timeout
- C) ECS Fargate Task triggered by EventBridge
- D) B or C depending on the need for CPU/memory beyond Lambda's limits

<details><summary>Answer</summary>

**D** — If the job fits in Lambda (≤ 15 min, ≤ 10 GB memory): EventBridge → Lambda is simpler and cheaper. If it needs more resources or > 15 min: EventBridge → ECS Fargate Task (you pay only for the task duration, no idle cost). Both are serverless and charge only for execution time.

</details>

---

**15.** An architecture has: API Gateway → Lambda A → Lambda B (synchronous call). Lambda B occasionally fails. How can the architecture be made more resilient?

- A) Configure automatic retry in Lambda A's code
- B) Replace the synchronous Lambda A → B call with Lambda A → SQS → Lambda B (async with retry)
- C) Add a CloudWatch Alarm to detect errors in Lambda B
- D) Configure Provisioned Concurrency on Lambda B

<details><summary>Answer</summary>

**B** — Synchronous Lambda-to-Lambda calls create coupling: if B fails, A fails. Introducing SQS decouples the two: A publishes to the queue and returns; SQS + Lambda B is asynchronous with automatic retry (up to maxReceiveCount attempts) and a DLQ for permanent failures. This improves resilience and scalability.

</details>

---

## Multiple-Response Questions

**Bonus 1.** Which of the following statements about Lambda are TRUE? (Select 2)

- A) Lambda supports up to a 15-minute timeout and 10 GB of memory
- B) Lambda can be configured with Provisioned Concurrency to eliminate cold starts
- C) Lambda always has a cold start on the first 10 invocations
- D) The maximum Lambda timeout is 5 minutes for functions in a VPC

<details><summary>Answer</summary>

**A and B** — Maximum timeout = 15 min and maximum memory = 10 GB. Provisioned Concurrency eliminates cold starts by keeping pre-warmed environments. There is no 10-invocation limit for cold starts (C); Lambda in a VPC has the same 15-min timeout as outside a VPC (D).

</details>
