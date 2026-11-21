# Questions — Introduction to SAA-C03

## Question 1
**Domain:** Design Resilient Architectures  
**Difficulty:** Easy

An analyst is starting to study for the SAA-C03 and wants to prioritize the highest-weighted domain when planning the first reviews. Which domain should receive the most initial attention?

- A) Design Secure Applications and Architectures
- B) Design Cost-Optimized Architectures
- C) Design Resilient Architectures
- D) Design High-Performing Architectures

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: C**

Design Resilient Architectures represents 30% of the exam, the highest weight among the four domains. That makes it a good starting point for strategic review. Option D is close, but High-Performing is 28%. Option A is 24% and B is 18%.

**Key concept:** official domain weights of the SAA-C03
</details>

## Question 2
**Domain:** Design High-Performing Architectures  
**Difficulty:** Medium

A team used to the CLF-C02 is moving its study plan to the SAA-C03. The technical manager wants to understand the main difference in depth between the exams. Which statement best describes this difference?

- A) The SAA-C03 only tests service names, while the CLF-C02 tests scenarios
- B) The SAA-C03 requires architectural decisions with trade-offs, while the CLF-C02 is more introductory
- C) The SAA-C03 does not cover security, only compute and networking
- D) The CLF-C02 and the SAA-C03 have exactly the same focus, differing only in language

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

The SAA-C03 requires contextual choices between services and architectural patterns, considering availability, security, performance, and cost. The CLF-C02 works more on conceptual recognition of services. Option A reverses the roles. Option C is false because security carries significant weight. Option D ignores the real difference in depth.

**Key concept:** difference between an introductory exam and an associate exam
</details>

## Question 3
**Domain:** Design Cost-Optimized Architectures  
**Difficulty:** Medium

A startup has a lean team and needs to launch an API with the least possible operational effort. During a review for the SAA-C03, an architect uses this scenario to explain how the exam thinks. Which approach is most aligned with AWS logic in exam questions?

- A) Always prioritize EC2 to have full control of the environment
- B) Prioritize managed services when they meet the requirement with less administration
- C) Avoid any serverless service because of timeout limits
- D) Choose the option with the highest theoretical performance, regardless of cost

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

In scenarios that ask for the least operational effort, AWS normally favors managed services such as Lambda, API Gateway, RDS, DynamoDB, S3, and ECS Fargate, as long as they meet the technical requirements. Option A increases the operational load. Option C generalizes a real limit, but out of context. Option D ignores the balance between cost and need.

**Key concept:** least operational overhead
</details>

## Question 4
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Easy

Which set best represents the most recurring services on the SAA-C03 and that, therefore, should receive in-depth study in the first weeks?

- A) Snowmobile, Braket, Ground Station, and GameLift
- B) EC2, S3, VPC, IAM, RDS, DynamoDB, Lambda, and Route 53
- C) Device Farm, CodeArtifact, Honeycode, and WorkSpaces
- D) AWS Wickr, IQ, Managed Blockchain, and Panorama

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

These services appear with high frequency because they form the core of the architectural decisions tested. The other options list services that may appear occasionally, but do not represent the center of the exam.

**Key concept:** prioritizing study by frequency of appearance
</details>

## Question 5
**Domain:** Design Resilient Architectures  
**Difficulty:** Medium

During a study group, a student claims that a Multi-AZ architecture already solves any business continuity requirement. Which answer best corrects this claim?

- A) Multi-AZ covers an Availability Zone failure, but not necessarily a regional disaster
- B) Multi-AZ is equivalent to Multi-Region
- C) Multi-AZ only applies to serverless applications
- D) Multi-AZ eliminates the need for backups

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: A**

Multi-AZ increases availability within a region, protecting against Availability Zone failures. However, a regional event may still require a multi-region strategy. Option B confuses concepts. Option C is false. Option D ignores that backup and DR are still necessary.

**Key concept:** difference between regional HA and multi-region DR
</details>

## Question 6
**Domain:** Design High-Performing Architectures  
**Difficulty:** Medium

An exam question describes a global HTTP application that needs to reduce latency for end users and also cache static content at edge locations. Which service tends to be the best answer?

- A) AWS Global Accelerator
- B) Amazon CloudFront
- C) AWS Direct Connect
- D) Amazon EventBridge

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

CloudFront is the AWS CDN and offers caching at edge locations, making it the most suitable choice for globally distributed HTTP content. Global Accelerator improves global routing for TCP/UDP applications and also HTTP, but it is not an edge cache. Direct Connect is private connectivity. EventBridge does not apply.

**Key concept:** CloudFront vs Global Accelerator
</details>

## Question 7
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A scenario asks for automatic rotation of database credentials and native integration with KMS. Which service is usually most aligned with the question statement?

- A) AWS Secrets Manager
- B) AWS Systems Manager Parameter Store Standard
- C) Amazon S3 Versioning
- D) Amazon CloudWatch Logs

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: A**

Secrets Manager is the strongest choice when the requirement includes sensitive secrets and automatic rotation. Parameter Store can store secrets, but rotation is not its main native differentiator. S3 Versioning and CloudWatch Logs do not solve this problem.

**Key concept:** Secrets Manager vs Parameter Store
</details>

## Question 8
**Domain:** Design Cost-Optimized Architectures  
**Difficulty:** Medium

When reading a question that highlights a tight budget and unpredictable load, which signal in the statement most strongly suggests evaluating serverless or on-demand elastic technologies?

- A) A requirement to use dedicated hardware
- B) A requirement for low organizational change
- C) A requirement for billing that matches consumption and low idle time
- D) A requirement for perpetual on-premises licensing

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: C**

When the problem emphasizes variable consumption and reduced idle time, services with on-demand billing and native elasticity become a better fit. Dedicated hardware goes in the opposite direction. Low organizational change may favor lift-and-shift, but does not address consumption optimization. On-premises licensing does not apply.

**Key concept:** consumption-driven cost
</details>

## Question 9
**Domain:** Design High-Performing Architectures  
**Difficulty:** Hard

A company needs to store highly relational data with ACID transactions and read at scale for distributed users. In an introductory architecture question, which combination best shows the kind of reasoning the exam expects?

- A) Replace everything with DynamoDB because it always scales better
- B) Use Amazon RDS or Aurora for transactional consistency and evaluate read replicas for reads
- C) Use Amazon S3 as the main transactional database
- D) Use Amazon CloudFront for database replication

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

The exam rewards choosing a managed relational database when the workload calls for transactions and a relational model, complemented with read-scaling strategies when appropriate. Option A oversimplifies and ignores requirements. Option C does not offer relational transactional semantics. Option D is not a database solution.

**Key concept:** aligning the data requirement with the right service
</details>

## Question 10
**Domain:** Design Resilient Architectures  
**Difficulty:** Medium

In an architecture question, the company wants to reduce coupling between producers and consumers, absorb spikes, and reprocess messages in case of failure. Which service commonly appears as the central answer?

- A) Amazon SQS
- B) Amazon Route 53
- C) Amazon Inspector
- D) AWS Artifact

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: A**

SQS is a classic decoupling mechanism with buffering, burst absorption, and reprocessing through visibility timeout, retention, and DLQ. Route 53 is DNS. Inspector performs vulnerability analysis. Artifact provides access to compliance reports.

**Key concept:** queue-based decoupling pattern
</details>

## Question 11
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Hard

A question states that a company wants to restrict the maximum permissions that development teams can delegate, even when local administrators create additional policies. Which concept should come to mind?

- A) Amazon CloudFront signed cookies
- B) Permission Boundaries
- C) S3 Multipart Upload
- D) RDS Read Replicas

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

Permission Boundaries define the effective ceiling of permissions an IAM identity can receive. This is a classic governance concept and usually arises in scenarios with controlled delegation. The other options address entirely different problems.

**Key concept:** permissions governance in IAM
</details>

## Question 12
**Domain:** Design Cost-Optimized Architectures  
**Difficulty:** Easy

A team is choosing between two valid alternatives. Both meet the technical requirement, but one of them eliminates operating system maintenance, patching, and idle capacity. By the exam's logic, which one tends to be preferred?

- A) The alternative with the greatest operational effort
- B) The alternative with more manual components
- C) The managed alternative, if the functional requirement is met
- D) The alternative with dedicated hardware by default

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: C**

This is a recurring pattern in the SAA-C03. If the managed solution meets the scenario without compromising critical requirements, it tends to be preferred because it reduces operations and risk. The others increase cost or complexity without a clear need.

**Key concept:** preference for managed services on the exam
</details>

## Question 13
**Domain:** Design High-Performing Architectures  
**Difficulty:** Medium

Which interpretation is most correct when a question mentions "near real-time analytics" over continuous events produced by applications?

- A) There may be a need for streaming, such as Kinesis, instead of isolated batch processing
- B) A traditional data warehouse with weekly loads should always be used
- C) The statement points directly to S3 Glacier Deep Archive
- D) The requirement excludes any managed service

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: A**

"Near real-time" is an indicator that data needs to flow and be processed with low latency, which often points to streaming or event-driven solutions. Weekly batch does not meet it. Glacier is archiving. Managed services are not excluded.

**Key concept:** keywords in the question statement
</details>

## Question 14
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A company wants to record API calls made in the AWS account for auditing and investigation purposes. Which service should be remembered first?

- A) Amazon CloudTrail
- B) Amazon CloudWatch Synthetics
- C) AWS Budgets
- D) Amazon Macie

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: A**

CloudTrail records API events and is central to auditing. CloudWatch Synthetics tests endpoints. Budgets monitors cost. Macie analyzes sensitive data in S3. All of them may appear on the exam, but only CloudTrail directly meets the described requirement.

**Key concept:** auditing actions in the AWS account
</details>

## Question 15
**Domain:** Design Resilient Architectures  
**Difficulty:** Hard

A candidate is reviewing a question in which all options work, but only one meets the requirement with the best combination of resilience, operational simplicity, and cost. Which mental approach is most appropriate for solving this type of problem?

- A) Always choose the most complex architecture, since it tends to be more robust
- B) Choose the cheapest option in isolation, without considering operations
- C) Evaluate the main requirement, constraints, and trade-offs before deciding
- D) Ignore words such as "securely", "cost-effective", and "minimal changes" because they are confusing

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: C**

This is the core of the SAA-C03. The exam measures the ability to evaluate trade-offs given explicit and implicit requirements. Excessive complexity is not an automatic prize. Lowest cost in isolation can create risk or unworkable operations. The words in the statement are decisive and should never be ignored.

**Key concept:** trade-off-driven architectural reasoning
</details>
