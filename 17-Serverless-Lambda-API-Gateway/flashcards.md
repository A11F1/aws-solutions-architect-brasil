# Flashcards — Module 11: Serverless, Lambda and API Gateway

> **Format:** Q = Question | A = Answer  
> **Total:** 25 flashcards

---

**Q:** What are the 3 Lambda invocation types?  
**A:** **Synchronous** (waits for a response — API GW, ALB, SDK); **Asynchronous** (doesn't wait — S3 Events, SNS, EventBridge); **Event Source Mapping** (Lambda polls — SQS, Kinesis, DynamoDB Streams)

---

**Q:** What is Lambda concurrency and what is the default limit per Region?  
**A:** The number of Lambda instances running simultaneously. Default: **1,000 concurrent executions per Region** (can be increased via support). Each synchronous invocation consumes 1 unit of concurrency while it runs.

---

**Q:** What is Provisioned Concurrency in Lambda?  
**A:** Pre-warms Lambda instances, eliminating cold starts. Instances stay "warm" and ready to respond immediately. It has an additional cost (paid per hour of provisioned concurrency). Useful for latency-sensitive applications.

---

**Q:** What happens to SQS messages when Lambda returns an error in an Event Source Mapping?  
**A:** Lambda retries automatically. After exhausting the attempts (configurable maxReceiveCount on the SQS DLQ), the message goes to the **Dead Letter Queue (DLQ)**. Lambda does not handle the SQS DLQ directly — the DLQ is configured on SQS itself.

---

**Q:** What is a Lambda Layer?  
**A:** A ZIP file containing dependencies (libraries, custom runtime) shared across multiple functions. Up to 5 layers per function. Maximum total size (code + layers): 250 MB unzipped. Avoids repeating dependencies in every deployment package.

---

**Q:** What is the difference between Lambda Proxy Integration and Custom Integration in API Gateway?  
**A:** **Proxy Integration:** API GW passes the entire HTTP request to Lambda (headers, body, query params) as the `event` object. Lambda returns the complete response (statusCode, headers, body). **Custom Integration:** API GW transforms the request/response via mapping templates (VTL). Proxy is simpler; custom offers more control.

---

**Q:** What is Lambda SnapStart and which runtime does it apply to?  
**A:** A feature to reduce cold start on Java 11+. When publishing a new version, AWS initializes the function and takes a snapshot of the state. New invocations restore the snapshot instead of initializing from scratch — reducing cold start from seconds to milliseconds.

---

**Q:** How does Lambda access resources in a private VPC?  
**A:** By configuring Lambda with **VPC Configuration** (VPC ID, subnets, security group). AWS creates an ENI (Elastic Network Interface) in the specified subnet. Lambda gets a private IP in the VPC and can access RDS, ElastiCache, etc. Requires: a NAT Gateway for Lambda to have internet access.

---

**Q:** What are the 3 API types in API Gateway?  
**A:** **REST API** (full functionality, staging, caching, throttling per stage); **HTTP API** (simpler, cheaper, lower latency, supports native JWT/OAuth); **WebSocket API** (persistent bidirectional connections, for chat and real-time notifications).

---

**Q:** What is throttling in API Gateway and how does it work?  
**A:** Limiting requests to protect the backend. Limits: **10,000 requests/second** per account (default) + **5,000 concurrent burst**. It can be configured separately per stage/method. When exceeded: HTTP 429 (Too Many Requests). The account limit can be increased via support.

---

**Q:** What does a Lambda authorizer check in API Gateway?  
**A:** **Token-based:** receives a token (JWT, OAuth) in the Authorization header and returns an IAM policy (Allow/Deny). **Request-based:** receives headers and query params and can use any authentication logic. The result is cached by TTL (up to 3600s) for performance.

---

**Q:** What is a Lambda Alias and how does it differ from a Version?  
**A:** **Version:** an immutable snapshot of code + configuration with a unique ARN (e.g., `function:1`). **Alias:** a named pointer (e.g., `PROD`) that points to one or two versions. Aliases support **canary deployment** (aliasName:weight to split traffic — e.g., 90% v1, 10% v2).

---

**Q:** What is the maximum Lambda execution time?  
**A:** **15 minutes** (900 seconds). For longer processes: use Step Functions (workflow orchestration), ECS/Fargate, or EC2.

---

**Q:** How does API Gateway support CORS?  
**A:** By configuring CORS in API Gateway to respond to pre-flight OPTIONS requests with the headers `Access-Control-Allow-Origin`, `Access-Control-Allow-Headers`, `Access-Control-Allow-Methods`. For Proxy Integration, the Lambda must also return the CORS headers in its responses.

---

**Q:** What is AWS SAM (Serverless Application Model)?  
**A:** A framework that simplifies deploying serverless applications. Simplified CloudFormation syntax: `AWS::Serverless::Function`, `AWS::Serverless::Api`. Commands: `sam build`, `sam deploy`, `sam local invoke` (local testing with Docker). Generates a CloudFormation stack under the hood.

---

**Q:** What is the Lambda Execution Role?  
**A:** The IAM Role assumed by the Lambda function during execution. It defines which AWS APIs the function can call (e.g., `dynamodb:PutItem`, `s3:GetObject`). The function assumes the Role automatically — no explicit credentials are needed in the code.

---

**Q:** What happens during a Lambda cold start?  
**A:** First invocation (or after a period of no use): AWS needs to allocate infrastructure, download the code, initialize the runtime, and run the code outside the handler. Adds latency (milliseconds to seconds, higher for Java). Solutions: Provisioned Concurrency, SnapStart (Java), keeping initialization code outside the handler.

---

**Q:** What native integrations does API Gateway have besides Lambda?  
**A:** HTTP Integration (forwards to an external HTTP endpoint), AWS Service Integration (calls AWS services directly — e.g., SQS, DynamoDB without an intermediary Lambda), Mock Integration (returns a static response), VPC Link (integrates with resources in a private VPC via NLB).

---

**Q:** What is an API Gateway Deployment Stage and what is it for?  
**A:** A deployed snapshot of the API with a unique URL (e.g., `https://api-id.execute-api.us-east-1.amazonaws.com/prod`). Enables multiple environments: dev, staging, prod. Each stage has independent settings: throttling, cache, stage variables (passed to Lambda/integration), logging.

---

**Q:** How does Lambda process multiple SQS messages simultaneously?  
**A:** Event Source Mapping processes messages in **batches** (up to 10,000). Lambda receives a batch of messages (configurable batch size). An error on any message in the batch can cause all of them to be returned to the queue. Solution: **Partial Batch Response** (individually report which messages failed).

---

**Q:** What are Lambda Destinations and when should you use them instead of a DLQ?  
**A:** For **asynchronous** invocations: configures success and failure destinations (SQS, SNS, EventBridge, another Lambda). More expressive than a DLQ (includes invocation context + original input + output/error). A DLQ only captures failed messages without the result. Lambda Destinations is the modern standard.

---

**Q:** What is the size limit of the Lambda invocation payload?  
**A:** **Synchronous:** 6 MB (request payload) + 6 MB (response payload). **Asynchronous:** 256 KB (event payload). For larger payloads: save the data to S3 and pass the S3 key as the event.

---

**Q:** What is an API Gateway Usage Plan?  
**A:** Defines the usage limit per API key: throttling (requests/second, burst) and quota (requests per day, week, or month). Lets you monetize APIs (different plans for different customer tiers), control usage per customer, and prevent abuse.

---

**Q:** What is Lambda@Edge?  
**A:** Runs Lambda code at CloudFront Points of Presence (edge locations) — close to the user. Use cases: URL rewrite, A/B testing, authentication at the edge, customizing headers. Limitations vs regular Lambda: lower timeout (5s viewer, 30s origin), lower max memory, no VPC access, limited Regions.

---

**Q:** How does a Lambda Function URL differ from API Gateway?  
**A:** Function URL: a dedicated HTTPS endpoint directly for the function (no API Gateway). Simpler, lower latency, no API Gateway cost. Limitations: no advanced throttling, no cache, no multiple stages, no native WAF integration (except via CloudFront). Use for simple webhooks or functions with occasional traffic.
