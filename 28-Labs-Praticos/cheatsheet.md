# Cheatsheet — Hands-On Labs (Module 18)

## AWS CLI Commands — Quick Reference

### Lambda
```bash
# Create function
aws lambda create-function \
  --function-name my-function \
  --runtime python3.12 \
  --role arn:aws:iam::ACCOUNT:role/lambda-role \
  --handler app.lambda_handler \
  --zip-file fileb://function.zip

# Invoke (synchronous)
aws lambda invoke --function-name my-function output.json

# Invoke (asynchronous)
aws lambda invoke --function-name my-function \
  --invocation-type Event output.json

# View recent logs
aws logs tail /aws/lambda/my-function --follow
```

### S3
```bash
# Create bucket
aws s3api create-bucket --bucket my-bucket --region us-east-1

# Upload
aws s3 cp file.txt s3://my-bucket/

# Sync directory
aws s3 sync ./local/ s3://my-bucket/prefix/

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# Block public access
aws s3api put-public-access-block \
  --bucket my-bucket \
  --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

### EC2 / Auto Scaling
```bash
# List running instances
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name]" \
  --output table

# Create Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --min-size 1 --max-size 5 --desired-capacity 2 \
  --launch-template LaunchTemplateId=lt-xxx,Version='$Latest' \
  --vpc-zone-identifier "subnet-aaa,subnet-bbb"
```

### CloudWatch
```bash
# Create CPU > 80% alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "High-CPU" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=AutoScalingGroupName,Value=my-asg \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:ACCOUNT:alerts

# Publish a custom metric
aws cloudwatch put-metric-data \
  --namespace "MyApp/Performance" \
  --metric-name "OrdersPerSecond" \
  --value 42 --unit Count
```

### DynamoDB
```bash
# Create table
aws dynamodb create-table \
  --table-name Orders \
  --attribute-definitions AttributeName=orderId,AttributeType=S \
  --key-schema AttributeName=orderId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

# Insert item
aws dynamodb put-item \
  --table-name Orders \
  --item '{"orderId": {"S": "001"}, "status": {"S": "pending"}}'
```

---

## Terraform — Common Resources

### Project Structure
```
project/
  main.tf          # main resources
  variables.tf     # variable declarations
  outputs.tf       # outputs
  terraform.tfvars # variable values
  versions.tf      # required_providers
```

### Versions Block (standard)
```hcl
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

### Most Tested Resources on the Exam

| Terraform Resource | AWS Service |
|------------------|-------------|
| `aws_vpc` + `aws_subnet` | VPC / Subnets |
| `aws_security_group` | Security Groups |
| `aws_instance` | EC2 |
| `aws_lambda_function` | Lambda |
| `aws_s3_bucket` + `aws_s3_bucket_versioning` | S3 |
| `aws_dynamodb_table` | DynamoDB |
| `aws_db_instance` | RDS |
| `aws_elasticache_cluster` | ElastiCache |
| `aws_ecs_cluster` + `aws_ecs_service` | ECS |
| `aws_iam_role` + `aws_iam_role_policy_attachment` | IAM |
| `aws_cloudwatch_metric_alarm` | CloudWatch |

---

## Terraform — Essential Commands

| Command | What it does |
|---------|-----------|
| `terraform init` | Initializes providers, downloads plugins |
| `terraform validate` | Validates HCL syntax |
| `terraform plan` | Previews changes without applying |
| `terraform plan -out=tfplan` | Saves the plan to apply later |
| `terraform apply` | Applies changes (asks for confirmation) |
| `terraform apply -auto-approve` | Applies without confirmation |
| `terraform apply tfplan` | Applies a saved plan |
| `terraform destroy` | Destroys all infrastructure |
| `terraform state list` | Lists resources in the state |
| `terraform state show aws_instance.web` | Details of a specific resource |
| `terraform output` | Displays defined outputs |
| `terraform fmt` | Formats HCL files |
| `terraform graph` | Generates a dependency graph |

---

## SAM — Essential Commands

```bash
# Initialize a SAM project
sam init

# Local build
sam build

# Test locally
sam local invoke FunctionName
sam local start-api  # local API GW on port 3000

# Deploy (guided)
sam deploy --guided

# Deploy with saved parameters
sam deploy

# View logs
sam logs -n FunctionName --stack-name my-stack --tail

# Delete stack
sam delete --stack-name my-stack
```

---

## Debug — Checklist by Problem

### Lambda cannot access RDS
```
1. Are Lambda and RDS in the same VPC? (Lambda needs VpcConfig)
2. Does the Lambda Security Group allow outbound to port 5432/3306?
3. Does the RDS Security Group allow inbound from the Lambda SG?
4. Is the Lambda in private subnets (not public)?
5. Do the private subnets have a NAT Gateway for internet (if needed)?
6. Does the Lambda IAM Role have the basic permission (AWSLambdaVPCAccessExecutionRole)?
7. Is the Lambda timeout sufficient (DB connection can take a while)?
```

### S3 returning 403 Forbidden
```
1. Is the Bucket Policy blocking it? (check aws s3api get-bucket-policy)
2. Is Block Public Access active while a public ACL is being used?
3. Does the user/role IAM Policy have s3:GetObject?
4. Does the object exist? (aws s3 ls s3://bucket/key)
5. Is the bucket in a different region than the client expects?
6. KMS key: does the user have decrypt permission on the key?
7. Is a VPC Endpoint policy blocking it? (if accessing via a VPC Gateway Endpoint)
```

### ECS Task does not start (STOPPED)
```
1. Check stoppedReason: aws ecs describe-tasks --cluster X --tasks Y
2. Common causes:
   - CannotPullContainerError: private ECR without IAM or VPC endpoint
   - OutOfMemory: insufficient memory defined in the task definition
   - ResourceInitializationError: missing execution role
3. CloudWatch Logs: /ecs/<task-family> — check application errors
4. Does the Task Role have the required permissions (e.g., s3:GetObject)?
5. Does the ECS Security Group allow outbound to RDS/external services?
```

---

## Cleanup Order in Terraform (`terraform destroy`)

Dependencies that can cause errors if deleted in the wrong order:
```
1. ECS Services (before the Cluster)
2. ALB / Target Groups (before subnets/SGs)
3. RDS / ElastiCache (before the subnet group and SGs)
4. Lambda (may have ENIs in the VPC)
5. NAT Gateway (before the EIP can be released)
6. Subnets (before the VPC)
7. VPC (last — everything inside it must be removed first)
```

> `terraform destroy` usually resolves the order through dependencies. Problems occur with resources created outside Terraform.

---

## IAM — Simulate Policy (permission debugging)

```bash
# Simulate whether an action is allowed
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT:role/my-role \
  --action-names s3:PutObject \
  --resource-arns arn:aws:s3:::my-bucket/*
  
# Expected result: "EvalDecision": "allowed"
```

---

## Cost Estimate — Typical Lab

| Resource | Estimated Cost (2h lab) |
|---------|------------------------|
| EC2 t3.micro | ~$0.01 |
| NAT Gateway | ~$0.09/h + $0.045/GB |
| RDS db.t3.micro Multi-AZ | ~$0.06/h |
| Lambda (1M req) | Free Tier |
| S3 (< 5GB) | Free Tier |
| ECS Fargate (0.25vCPU/0.5GB) | ~$0.01/h |

> **Tip:** Always destroy resources when you finish. NAT Gateway is billed hourly even without traffic.

---

## Exam Tips

| Clue in the Question | Expected Answer |
|-----------------|------------------|
| "test Lambda locally before deploying" | SAM local invoke / sam local start-api |
| "infrastructure as code, reproducible" | Terraform or CloudFormation |
| "Lambda needs RDS in a private VPC" | Lambda VpcConfig + RDS SG opened |
| "preview changes before applying" | terraform plan |
| "Lambda cannot reach the internet in a VPC" | Missing NAT Gateway in the private subnet |
| "store secrets for Lambda to access" | Secrets Manager (not environment variables) |
| "serverless deployment with simplified YAML" | SAM (template.yaml) |
| "infrastructure state shared across teams" | Terraform state in S3 + DynamoDB lock |
| "debug S3 403 error" | Check bucket policy + Block Public Access + IAM |
| "ECS task stops right after starting" | Check stoppedReason + CloudWatch Logs /ecs/ |
