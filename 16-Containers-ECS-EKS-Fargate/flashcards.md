# Flashcards — Module 10: Containers (ECS, EKS, Fargate)

---

**Q:** What is the object hierarchy in ECS?
**A:** Cluster → Service → Task → Container. The Task Definition defines the blueprint (image, CPU, RAM, vars, volumes, IAM). The Service keeps N tasks running. The Cluster is the logical grouping of resources

---

**Q:** What is Task Role vs Task Execution Role in ECS?
**A:** Task Role: IAM role for the **application** inside the container (permission to access S3, DynamoDB, etc.). Task Execution Role: IAM role for the **ECS Agent** (pull the image from ECR, push logs to CloudWatch, read Secrets Manager/SSM)

---

**Q:** What is the only network mode supported by Fargate?
**A:** **awsvpc mode**. Each Fargate task receives a dedicated ENI with a private VPC IP. Allows individual Security Groups per task. Bridge and host modes only work on the EC2 Launch Type

---

**Q:** What is the advantage of awsvpc mode on ECS EC2?
**A:** Each task receives its own ENI with a VPC IP → can have an individual SG per task (not per instance). Facilitates granular access control and compatibility with ALB target type "ip". Downside: ENI limit per EC2 instance

---

**Q:** What are the CPU and RAM limits that can be configured on a Fargate task?
**A:** CPU: 0.25 vCPU to 16 vCPU. RAM: proportional to CPU (e.g., 0.25 vCPU → 0.5-2 GB; 16 vCPU → 32-120 GB). Billed per vCPU-second and GB-second while the task is active

---

**Q:** What are the 3 Launch Types available in ECS?
**A:** EC2 (you manage the cluster's EC2 instances), Fargate (serverless, AWS manages the host), External/ECS Anywhere (on-premises or edge servers registered as an ECS cluster)

---

**Q:** What is an ECS Capacity Provider?
**A:** Automatically manages the cluster's EC2 instance Auto Scaling Group based on pending tasks, optimizing bin packing. With "Managed Scaling" enabled, ECS scales instances up/down automatically before tasks become queued

---

**Q:** What is the control plane cost of an EKS cluster?
**A:** $0.10/hour per EKS cluster (~$72/month). ECS does not charge for the cluster (only for compute: EC2 or Fargate). Consider this fixed cost for small workloads on EKS

---

**Q:** What are EKS Managed Node Groups?
**A:** AWS manages the lifecycle of the EC2 worker nodes (provisioning, patching, zero-downtime updates, draining). You choose the instance type and AMI. Different from Fargate (no nodes) and Self-managed nodes (you manage everything manually)

---

**Q:** What is IRSA (IAM Roles for Service Accounts) in EKS?
**A:** Lets you associate IAM Roles with Kubernetes Service Accounts. Pods that use the Service Account receive temporary AWS credentials via OIDC (no hardcoding). Equivalent to the ECS Task Role but for Kubernetes pods — per-pod/deployment granularity

---

**Q:** What is the Amazon VPC CNI in EKS?
**A:** The default EKS network plugin. Allocates IPs directly from the VPC to each pod (no overlay network). Result: pods have native VPC IPs, communicate directly with other AWS resources, and can have SGs applied directly (EKS Security Groups for Pods)

---

**Q:** What is an ECR Lifecycle Policy and how does it work?
**A:** Rules that automatically delete images based on criteria: age (days), count (keep the N most recent), tag status (tagged/untagged). Examples: "delete untagged images older than 7 days" or "keep only the 5 most recent images per tag prefix"

---

**Q:** What is the difference between ECR Image Scanning Basic and Enhanced?
**A:** Basic (Clair): checks OS CVEs (OS packages). Enhanced (Amazon Inspector): checks CVEs in the OS AND in programming languages (Python, Node.js, Java packages). Enhanced = more detailed. Results available in the Inspector Console + EventBridge events

---

**Q:** How do ECS containers access secrets (DB credentials) securely?
**A:** In the Task Definition, `secrets` field: reference the ARN of Secrets Manager or SSM Parameter Store. The ECS Agent (via the Task Execution Role) fetches the secret at launch and injects it as an environment variable or file. No hardcoded credentials

---

**Q:** What is AWS App Runner?
**A:** A serverless PaaS for web containers. You provide an ECR image or source code → App Runner automatically creates the load balancer, HTTPS, auto-scaling, and custom domain. Zero ECS/Fargate/ALB configuration. Simpler than ECS, less control

---

**Q:** What is ECS Service Connect?
**A:** An AWS-managed Service Mesh for service-to-service communication inside an ECS cluster. Provides service discovery, routing, retries, timeouts, and observability (metrics, tracing) without configuring App Mesh or Envoy manually

---

**Q:** What is the scaling mode in EKS for pods and for nodes?
**A:** Pods: **HPA** (Horizontal Pod Autoscaler) — scales the number of pods based on CPU/RAM/custom metrics. Nodes: **Cluster Autoscaler** (scales node groups) or **Karpenter** (faster and more efficient node provisioning, AWS-native). Fargate nodes: scale automatically per pod

---

**Q:** What is an EKS Fargate Profile?
**A:** A selector that defines which pods run on Fargate (by namespace and labels). A pod matching the profile is automatically scheduled on Fargate (no EC2 instances). Pods that do not match go to node groups. Each Fargate pod = an isolated micro-VM

---

**Q:** How does authentication to Amazon ECR work for Docker pull?
**A:** Uses the AWS CLI: `aws ecr get-login-password --region ... | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com`. Token valid for 12 hours. Kubernetes/EKS: uses the ECR credential helper or IRSA for automatic renewal

---

**Q:** What is the AWS Copilot CLI?
**A:** A CLI that automates deployment of containerized applications on ECS/Fargate following best practices. Automatically creates: ECR, Task Definition, ECS Service, ALB, VPC, CodePipeline, SSM/Secrets. Abstracts complexity without removing control the way App Runner does
