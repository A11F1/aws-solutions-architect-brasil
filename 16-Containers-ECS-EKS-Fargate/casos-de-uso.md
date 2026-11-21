# Use Cases — Module 10: Containers (ECS, EKS, Fargate)

## Case 1: Microservices with ECS Fargate + ALB + Service Connect

**Scenario:** A fintech startup with 5 microservices. A small team wants containers without managing infrastructure. Secure inter-service communication with observability.

**Architecture:**
```
Internet → Route 53 → ALB (api.fintech.com)
                        ├── /payments/*  → ECS Service: PaymentService (Fargate)
                        ├── /accounts/*  → ECS Service: AccountService (Fargate)
                        └── /auth/*      → ECS Service: AuthService (Fargate)

Internal communication (ECS Service Connect):
  PaymentService → http://accountservice:8080/validate
  PaymentService → http://authservice:8080/verify
  [Service Connect = internal DNS, automatic retries, metrics in CloudWatch]

IAM (Task Roles):
  PaymentService  → DynamoDB (payments table) + SQS (payment events)
  AccountService  → RDS Aurora (accounts DB)
  AuthService     → DynamoDB (sessions) + Secrets Manager (JWT keys)

ECR: separate repository per service
Secrets: Secrets Manager → injected via Task Definition (Execution Role)
Logs: awslogs driver → CloudWatch Logs Groups per service
```

---

## Case 2: EKS with Managed Node Groups for a Data Platform

**Scenario:** A company with a DevOps team experienced in Kubernetes. It needs a data platform with Spark, Kafka, and containerized REST APIs.

**Architecture:**
```
EKS Cluster (us-east-1)
  ├── Node Group: general (m6i.xlarge, 4 nodes, On-Demand)
  │     └── Pods: API services, monitoring, ingress controllers
  │
  ├── Node Group: compute-intensive (c6i.4xlarge, 0-20 nodes, Spot)
  │     └── Pods: Spark executors, Kafka brokers
  │
  └── Fargate Profile: namespace=monitoring
        └── Pods: Prometheus, Grafana (serverless)

AWS Add-ons:
  ├── AWS Load Balancer Controller → ALB per Kubernetes Ingress
  ├── Amazon VPC CNI → pods with VPC IPs
  ├── AWS EBS CSI Driver → PVCs for Kafka persistent storage
  └── ADOT (AWS Distro for OpenTelemetry) → traces to X-Ray

IRSA:
  ServiceAccount: spark-sa → IAM Role → S3 (data lake read/write)
  ServiceAccount: kafka-sa → IAM Role → MSK (access to the Kafka cluster)
```

---

## Case 3: Blue/Green Deployment on ECS with CodeDeploy

**Scenario:** A critical API with a requirement for zero-downtime deployments and rollback in < 5 minutes in case of a problem.

**Process:**
```
Git Push → CodePipeline
  ├── CodeBuild: docker build → ECR push (v2.0)
  └── CodeDeploy (ECS Blue/Green):
        1. Creates a new "Green" task set with image v2.0
        2. ALB gradually shifts a % of traffic to Green
           [Canary: 10% for 5 min → if ok → 90% → 100%]
        3. Health Check period: 5 minutes
        4. Success: terminates Blue tasks
        ---- OR ----
        4. Failure detected → CodeDeploy reverts ALB to Blue in < 1 min
        5. Green tasks terminated automatically

ECS Service configuration:
  Deployment Type: CODE_DEPLOY
  ALB: 2 target groups (Blue + Green)
  CodeDeploy App: appspec.yml defines Lambda hooks for smoke tests
```

---

## Case 4: ML Workload with EKS + Karpenter + GPU Nodes

**Scenario:** An ML team needs to train models with GPUs (p3.8xlarge) that sit idle between experiments. Cost is critical.

**Architecture:**
```
EKS Cluster
  └── Karpenter (node provisioner) ← instead of Cluster Autoscaler
        ├── NodeClass: GPU (p3.8xlarge Spot, max 10 nodes)
        └── NodePool: ml-training (namespace=ml, tolerations=gpu)

Training Flow:
1. Data Scientist submits a Kubernetes Job (spec: GPU request)
2. Karpenter detects a Pending pod (no node available)
3. Karpenter provisions a GPU Spot node in ~30 seconds
4. Pod starts ML training
5. Job completes → pod terminated
6. Karpenter terminates the node after 2 min of idleness

IRSA for S3 access:
  ServiceAccount: ml-training → IAM Role → S3 (datasets, model artifacts)
  
FSx for Lustre (CSI Driver):
  PVC → PersistentVolume → FSx for Lustre (integrated with S3)
  [High read speed of datasets during training]
```

**Savings:** Spot GPU instance = 70% discount. Karpenter terminates nodes within minutes of inactivity vs Cluster Autoscaler, which waited 10+ minutes.

---

## Case 5: Migration to ECS with App Runner and a CI/CD Pipeline

**Scenario:** A startup migrates from on-premises VMs to containers with no Kubernetes or ECS experience. It needs simple CI/CD.

**Simplified Architecture (App Runner):**
```
GitHub Repo (main branch)
  └── CodePipeline:
        ├── CodeBuild: docker build → ECR
        └── App Runner Deploy: updates the service with the new image
              └── App Runner Service:
                    ├── Auto-scaling: min=1, max=10 containers
                    ├── Automatic HTTPS (managed ACM)
                    ├── Custom domain: api.startup.io
                    └── Source: ECR image

VPC Connector (App Runner + VPC):
  App Runner → VPC Connector → RDS (private subnet)
               [App Runner originally has no VPC access; VPC Connector solves it]

App Runner limits (future motivation for ECS):
  ✗ No full VPC control
  ✗ No multiple containers per service
  ✗ No persistent volumes
  ✗ HTTP workloads only
  → When these limits are reached: migrate to ECS Fargate
```

**Recommended progression:**
```
App Runner (start) → ECS Fargate (when you need control) → EKS (when you need K8s)
```
