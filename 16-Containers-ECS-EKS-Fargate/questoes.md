# Exam Questions — Module 10: Containers (ECS, EKS, Fargate)

<!-- SAA-C03 Domain: Design High-Performing Architectures / Design Resilient Architectures -->

---

**1.** A team wants to run containers on AWS without managing EC2 servers. Which combination allows this with ECS?

- A) ECS with EC2 Launch Type
- B) ECS with Fargate Launch Type
- C) ECS with External Launch Type
- D) ECS with manually configured Auto Scaling Groups

<details>
<summary>Answer</summary>
**B — ECS with Fargate**
Fargate is serverless: you define the task (CPU, RAM, image), and AWS manages the entire underlying host. No EC2 to provision, patch, or size. The EC2 Launch Type requires you to register EC2 instances in the cluster (you manage the fleet). External is for on-prem servers (ECS Anywhere).
</details>

---

**2.** What is the difference between Task Role and Task Execution Role in ECS?

- A) Task Role is for the ECS agent; Execution Role is for the container
- B) Task Role is for the application inside the container; Execution Role is for ECS to manage the task (pull ECR image, push logs)
- C) Both are equivalent — the distinction is only naming
- D) Task Role applies to the EC2 Launch Type; Execution Role to Fargate

<details>
<summary>Answer</summary>
**B — Correct**
**Task Role**: IAM policies for what the **application** can do (e.g., read S3, write DynamoDB). Obtained via the Task Metadata Endpoint (the task's IMDS). **Task Execution Role**: for the ECS and Fargate agent: pull the image from ECR, send logs to CloudWatch, read Secrets Manager/SSM Parameter Store. Both are needed when the application accesses AWS and uses ECR.
</details>

---

**3.** A container in ECS with the Fargate launch type needs each task to have its own Security Group for granular network control. Which network mode should be enabled?

- A) bridge mode
- B) host mode
- C) awsvpc mode
- D) none mode

<details>
<summary>Answer</summary>
**C — awsvpc mode**
With `awsvpc`, each task receives **a dedicated ENI (Elastic Network Interface)** with a private VPC IP. This lets you associate individual Security Groups per task. It is the **only mode supported by Fargate**. `bridge` uses dynamic ports of the EC2 host. `host` uses the host network directly.
</details>

---

**4.** A company uses the ECS EC2 launch type and wants to distribute traffic among multiple containers on the same EC2 instance, with different dynamically assigned ports. Which configuration on the ALB?

- A) Application Load Balancer with target type "instance" and dynamic port mapping
- B) Network Load Balancer with fixed IP targets
- C) Application Load Balancer with target type "ip" in awsvpc mode
- D) Classic Load Balancer with sticky sessions

<details>
<summary>Answer</summary>
**A — ALB with dynamic port mapping (bridge mode)**
In ECS EC2 with bridge mode, no fixed port is defined (or hostPort=0 is used) — ECS assigns dynamic host ports. The ALB with target type "instance" uses the ECS service to automatically register the (instance, dynamic port) pairs. With awsvpc, target type "ip" is used (each task has its own unique IP).
</details>

---

**5.** Which ECS component defines the containers (Docker image, CPU, RAM, environment variables, volumes, IAM) of a task?

- A) ECS Service
- B) ECS Cluster
- C) Task Definition
- D) Capacity Provider

<details>
<summary>Answer</summary>
**C — Task Definition**
The Task Definition is the JSON blueprint that defines: Docker image(s), allocated CPU/RAM, port mapping, mounted volumes, environment variables, Task Role, Task Execution Role, log driver, etc. It is versioned. The ECS Service uses the Task Definition to launch and keep N tasks running.
</details>

---

**6.** A company uses Amazon EKS and wants to run pods without managing worker nodes (serverless). Which configuration should it use?

- A) EKS Managed Node Groups with Auto Scaling
- B) EKS Self-managed Nodes with Spot instances
- C) EKS with Fargate Profiles
- D) EKS with AWS Batch integration

<details>
<summary>Answer</summary>
**C — EKS Fargate Profiles**
Fargate Profiles on EKS define which pods (by namespace/labels) should run on Fargate. AWS provisions and manages the compute; you do not see or manage EC2 nodes. Managed Node Groups manage EC2 nodes automatically but you still choose types, AMIs, etc. Fargate = zero node management.
</details>

---

**7.** What is the approximate hourly cost of the control plane of an EKS cluster?

- A) Free — only the worker nodes are charged
- B) $0.10/hour per EKS cluster (~$72/month)
- C) $0.50/hour per cluster
- D) Charged by the number of running pods

<details>
<summary>Answer</summary>
**B — $0.10/hour per cluster (~$72/month)**
EKS charges $0.10 per hour for the managed control plane (API server, etcd, etc.) — this is fixed regardless of the number of nodes/pods. ECS has no cluster cost; only compute (EC2 or Fargate). This is an important differentiator when choosing ECS vs EKS for small workloads.
</details>

---

**8.** A Docker image was pushed to ECR. How do you ensure that only the 10 most recent images per repository are retained (automatically deleting old images)?

- A) S3 Lifecycle Policy integrated with ECR
- B) ECR Lifecycle Policy with an image count rule
- C) Lambda with CloudWatch Events cleaning images weekly
- D) ECR Image Scanning with auto-delete of old images

<details>
<summary>Answer</summary>
**B — ECR Lifecycle Policy**
ECR Lifecycle Policies let you create rules such as: keep only the N most recent images per tag prefix, delete untagged images after X days, etc. Applied automatically by ECR with no additional compute. Image Scanning detects vulnerabilities (CVEs) — it does not manage retention.
</details>

---

**9.** What is IRSA (IAM Roles for Service Accounts) in EKS?

- A) It lets EKS pods use IAM roles without needing hardcoded credentials
- B) A service to federate IAM identities across multiple EKS clusters
- C) An authentication mechanism for EKS nodes with the control plane
- D) It allows cross-account access between EKS clusters

<details>
<summary>Answer</summary>
**A — IAM Roles for individual pods**
IRSA lets you associate an IAM Role with a Kubernetes Service Account. Pods that use that Service Account automatically receive temporary AWS credentials (via OIDC). This is the equivalent of the Task Role in ECS — without putting credentials in environment variables or using the entire node's role for the pod.
</details>

---

**10.** A company wants to use ECS or EKS. The team has little experience but needs to integrate with third-party Kubernetes Helm Charts. Which should it choose?

- A) ECS — simpler and no cluster cost
- B) EKS — native Kubernetes support and ecosystem (Helm, Operators, etc.)
- C) App Runner — eliminates the need for orchestration
- D) EC2 with Docker Compose

<details>
<summary>Answer</summary>
**B — EKS**
If the explicit requirement is Kubernetes compatibility (Helm Charts, Operators, kubectl, native RBAC), EKS is the choice. ECS is not Kubernetes and does not support Helm. If the priority were simplicity without Kubernetes, ECS (especially + Fargate) would be better. The presence of Helm Charts as a requirement points to EKS.
</details>

---

**11.** How does automatic task scaling work in ECS based on SQS queue metrics?

- A) ECS Service Auto Scaling with a Target Tracking Policy based on ApproximateNumberOfMessagesVisible
- B) A Lambda function that invokes ECS RunTask as the queue grows
- C) SQS Auto Scaling Group trigger for the ECS Cluster
- D) CloudWatch Alarm → SNS → ECS TaskSet update

<details>
<summary>Answer</summary>
**A — ECS Service Auto Scaling with Target Tracking**
ECS Service Auto Scaling supports Target Tracking, Step Scaling, and Scheduled Scaling. For SQS: create a CloudWatch Custom Metric with `ApproximateNumberOfMessagesVisible / number_of_tasks`, and configure Target Tracking to keep that ratio at a target (e.g., 10 messages per task). ECS scales tasks automatically based on the queue load.
</details>

---

**12.** Which AWS service is best suited for quickly deploying a containerized web application with zero infrastructure configuration (no ECS, ALB, etc.)?

- A) ECS + Fargate + ALB (manual)
- B) AWS Copilot CLI
- C) AWS App Runner
- D) Elastic Beanstalk with Docker

<details>
<summary>Answer</summary>
**C — AWS App Runner**
App Runner is a PaaS for web containers: you provide the ECR image (or source code) and App Runner creates and manages all the infrastructure (load balancer, auto-scaling, HTTPS, custom domain). Zero ECS/ALB configuration. Elastic Beanstalk with Docker is more flexible but more complex. Copilot automates ECS/Fargate but still exposes the components.
</details>

---

**13.** What is the recommended secure mechanism for an ECS container to access database credentials stored in Secrets Manager?

- A) Inject the credential as a hardcoded environment variable in the Task Definition
- B) Mount an EFS volume with the credentials file
- C) Reference the Secrets Manager ARN in the Task Definition — ECS automatically injects it as an env var or file
- D) A Lambda sidecar container that fetches the credential and passes it via a shared volume

<details>
<summary>Answer</summary>
**C — Referencing Secrets Manager in the Task Definition**
ECS supports injecting secrets directly from the Task Definition: in the `secrets` field, reference the ARN of Secrets Manager (or SSM Parameter Store). The ECS Execution Role pulls the value at task launch and injects it as an environment variable. No credential is hardcoded. C is the recommended AWS standard.
</details>

---

**14.** What differentiates an ECS Capacity Provider from simply using Auto Scaling Groups manually on the ECS EC2 Launch Type?

- A) The Capacity Provider automatically manages EC2 instance scaling based on pending tasks, without manual intervention
- B) The Capacity Provider is exclusive to Fargate and does not work with EC2
- C) There is no difference — a Capacity Provider is just an alias for an ASG
- D) The Capacity Provider eliminates the need for Task Definitions

<details>
<summary>Answer</summary>
**A — Automatic cluster scaling management**
Capacity Provider with Managed Scaling: automatically scales the cluster's EC2 Auto Scaling Group based on pending tasks (optimized bin packing). Without a CP, you manage it manually or configure simple scaling policies. With a CP, ECS "knows" how many instances are needed to accommodate all tasks and scales proactively before tasks become pending.
</details>

---

**15.** A company wants to check for CVE vulnerabilities in Docker images in ECR before they are deployed. Which configuration?

- A) SonarQube integrated with ECR
- B) ECR Image Scanning (Basic via Clair or Enhanced via Amazon Inspector)
- C) GuardDuty with ECR integration
- D) A CodeBuild step that runs Trivy manually

<details>
<summary>Answer</summary>
**B — ECR Image Scanning**
ECR offers two types of scanning: **Basic** (uses open-source Clair, checks OS CVEs) enabled on-push or manually. **Enhanced Scanning** (via Amazon Inspector) checks CVEs in the OS and in programming languages (Python packages, Node.js modules, etc.) with results in the Inspector Console and EventBridge events for automation. Amazon Inspector → ECR = continuous image scanning.
</details>
