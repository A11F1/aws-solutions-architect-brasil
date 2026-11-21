# Flashcards — Module 18: Hands-On Labs

> **Format:** Q = Question | A = Answer  
> **Total:** 25 flashcards

---

**Q:** How do you create a Lambda Function with DynamoDB permissions via the CLI?  
**A:**
```bash
# 1. Create IAM Role
aws iam create-role --role-name LambdaDynamoRole \
  --assume-role-policy-document file://trust-lambda.json

# 2. Attach policy
aws iam attach-role-policy --role-name LambdaDynamoRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess

# 3. Deploy Lambda
aws lambda create-function --function-name MyFunc \
  --runtime python3.12 --role arn:aws:iam::ACCOUNT:role/LambdaDynamoRole \
  --handler lambda_function.lambda_handler --zip-file fileb://function.zip
```

---

**Q:** How do you create a VPC with a public and a private subnet via Terraform?  
**A:**
```hcl
resource "aws_vpc" "main" { cidr_block = "10.0.0.0/16" }
resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id; cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true; availability_zone = "us-east-1a"
}
resource "aws_subnet" "private" {
  vpc_id = aws_vpc.main.id; cidr_block = "10.0.2.0/24"
  availability_zone = "us-east-1b"
}
resource "aws_internet_gateway" "igw" { vpc_id = aws_vpc.main.id }
```

---

**Q:** How do you check whether an EC2 instance in a private subnet has internet connectivity?  
**A:** The private instance needs a **NAT Gateway** in the public subnet with a route in the private subnet: `0.0.0.0/0 → NAT GW`. To test: `curl -I https://aws.amazon.com` (from inside the instance via SSM Session Manager). Without a NAT GW: timeout. With a NAT GW + correct route: HTTP 200.

---

**Q:** What is the AWS CLI command to upload a file to S3 with SSE-KMS encryption?  
**A:**
```bash
aws s3 cp file.txt s3://my-bucket/file.txt \
  --sse aws:kms \
  --sse-kms-key-id arn:aws:kms:us-east-1:123456:key/key-id
```
Verify:
```bash
aws s3api head-object --bucket my-bucket --key file.txt \
  --query 'ServerSideEncryption'
```

---

**Q:** How do you test an API Gateway + Lambda locally before deploying?  
**A:**
```bash
# Install SAM CLI
pip install aws-sam-cli

# Invoke Lambda locally
sam local invoke "FunctionName" -e events/event.json

# Start local API (port 3000)
sam local start-api

# Test
curl -X POST http://localhost:3000/endpoint \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

---

**Q:** Which Terraform command previews changes before applying?  
**A:**
```bash
terraform plan -out=tfplan    # shows changes + saves the plan
terraform show tfplan         # views the saved plan
terraform apply tfplan        # applies exactly the saved plan
```
`-out=tfplan` ensures `apply` applies exactly what was reviewed in `plan`, with no surprises from state changes between plan and apply.

---

**Q:** How do you configure an Auto Scaling Group to scale based on CPU via the CLI?  
**A:**
```bash
# Create scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name MyASG \
  --policy-name cpu-scale-out \
  --scaling-adjustment 1 \
  --adjustment-type ChangeInCapacity

# Create a CloudWatch Alarm that triggers the policy
aws cloudwatch put-metric-alarm \
  --alarm-name High-CPU \
  --metric-name CPUUtilization --namespace AWS/EC2 \
  --threshold 70 --comparison-operator GreaterThanThreshold \
  --statistic Average --period 300 --evaluation-periods 2 \
  --alarm-actions arn:aws:autoscaling:...policy/cpu-scale-out
```

---

**Q:** How do you enable versioning on S3 and list the versions of an object?  
**A:**
```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# List versions of an object
aws s3api list-object-versions \
  --bucket my-bucket --prefix file.txt \
  --query 'Versions[*].[VersionId,LastModified,IsLatest]'

# Restore a specific version
aws s3api copy-object --bucket my-bucket \
  --copy-source "my-bucket/file.txt?versionId=abc123" \
  --key file.txt
```

---

**Q:** How do you create a Multi-AZ RDS instance with Terraform?  
**A:**
```hcl
resource "aws_db_instance" "main" {
  identifier        = "prod-db"
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.t3.medium"
  allocated_storage = 20
  username          = "admin"
  password          = var.db_password
  multi_az          = true
  skip_final_snapshot = false
  final_snapshot_identifier = "prod-db-final"
  backup_retention_period = 7
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
}
```

---

**Q:** How do you check a Lambda function's logs in CloudWatch via the CLI?  
**A:**
```bash
# List log streams (invocations)
aws logs describe-log-streams \
  --log-group-name /aws/lambda/MyFunction \
  --order-by LastEventTime --descending --max-items 5

# View logs of the latest invocation
aws logs get-log-events \
  --log-group-name /aws/lambda/MyFunction \
  --log-stream-name "2024/01/01/[$LATEST]abc123" \
  --query 'events[*].message'
```

---

**Q:** How do you create a DynamoDB table with a GSI via Terraform?  
**A:**
```hcl
resource "aws_dynamodb_table" "orders" {
  name         = "Orders"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "orderId"
  range_key    = "createdAt"
  attribute { name = "orderId"   type = "S" }
  attribute { name = "createdAt" type = "S" }
  attribute { name = "userId"    type = "S" }

  global_secondary_index {
    name            = "UserIdIndex"
    hash_key        = "userId"
    range_key       = "createdAt"
    projection_type = "ALL"
  }
}
```

---

**Q:** How do you configure CORS in API Gateway via Terraform?  
**A:**
```hcl
resource "aws_api_gateway_method" "options" {
  rest_api_id   = aws_api_gateway_rest_api.api.id
  resource_id   = aws_api_gateway_resource.resource.id
  http_method   = "OPTIONS"
  authorization = "NONE"
}
resource "aws_api_gateway_integration" "options" {
  http_method = aws_api_gateway_method.options.http_method
  type        = "MOCK"
  request_templates = { "application/json" = "{\"statusCode\": 200}" }
}
# + method_response and integration_response with CORS headers
```

---

**Q:** How do you create a CloudWatch Alarm to monitor Lambda errors and send them to SNS?  
**A:**
```bash
# Create SNS topic
aws sns create-topic --name lambda-errors-alert

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "LambdaErrors-MyFunction" \
  --metric-name Errors --namespace AWS/Lambda \
  --dimensions Name=FunctionName,Value=MyFunction \
  --statistic Sum --period 60 \
  --threshold 5 --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:ACCOUNT:lambda-errors-alert
```

---

**Q:** How do you clean up Terraform resources to avoid costs?  
**A:**
```bash
# Destroy all resources in the state
terraform destroy

# Check what will be destroyed beforehand
terraform plan -destroy

# Force destruction of an S3 bucket with objects (force_destroy=true in the .tf)
# or via the CLI:
aws s3 rm s3://my-bucket --recursive
aws s3api delete-bucket --bucket my-bucket

# Check for remaining resources in the account
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Environment,Values=lab
```

---

**Q:** How do you diagnose why a Lambda cannot reach an RDS instance in a VPC?  
**A:** Checklist: (1) Does the Lambda have a VPC config with a private subnet? (2) Does the Lambda Security Group have outbound on port 3306? (3) Does the RDS Security Group have inbound on port 3306 **from the Lambda's SG** (not from the CIDR)? (4) Do the Lambda's subnets route to a NAT GW (to call AWS APIs)? (5) Is RDS in the same VPC as the Lambda? (6) Is the RDS endpoint correct (not the IP)?

---

**Q:** How do you create an ECS Fargate Service with Terraform?  
**A:**
```hcl
resource "aws_ecs_service" "app" {
  name            = "my-app"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 2
  launch_type     = "FARGATE"
  network_configuration {
    subnets          = [aws_subnet.private.id]
    security_groups  = [aws_security_group.app.id]
    assign_public_ip = false
  }
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 8080
  }
}
```

---

**Q:** How do you use the AWS CLI to test an IAM Policy without applying it?  
**A:**
```bash
# Simulates whether an action is allowed for the principal
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT:role/MyRole \
  --action-names "s3:GetObject" \
  --resource-arns "arn:aws:s3:::my-bucket/*"

# Result: allowed/denied + which policy blocked it
# Also useful: iam simulate-custom-policy to test a JSON policy directly
```

---

**Q:** How do you deploy a Lambda application with SAM and create an API Gateway automatically?  
**A:**
```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  HelloFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.12
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```
```bash
sam build && sam deploy --guided
```

---

**Q:** What is the CLI command to list running EC2 instances by region?  
**A:**
```bash
# Running instances in the default region
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# All regions (bash script)
for region in $(aws ec2 describe-regions --query 'Regions[*].RegionName' --output text); do
  echo "Region: $region"
  aws ec2 describe-instances --region $region --filters "Name=instance-state-name,Values=running" --query 'Reservations[*].Instances[*].InstanceId' --output text
done
```

---

**Q:** How do you configure an S3 Lifecycle Policy to transition to Glacier via the CLI?  
**A:**
```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket \
  --lifecycle-configuration '{
    "Rules": [{
      "ID": "move-to-glacier",
      "Status": "Enabled",
      "Filter": {"Prefix": "logs/"},
      "Transitions": [
        {"Days": 30, "StorageClass": "STANDARD_IA"},
        {"Days": 90, "StorageClass": "GLACIER"}
      ],
      "Expiration": {"Days": 365}
    }]
  }'
```

---

**Q:** How do you debug a 403 Forbidden error on S3?  
**A:** Checklist: (1) Is the Bucket Policy blocking it? `aws s3api get-bucket-policy --bucket bucket`. (2) Is Block Public Access enabled? `aws s3api get-public-access-block --bucket bucket`. (3) Does the role/user IAM Policy have `s3:GetObject`? `aws iam simulate-principal-policy`. (4) Object ACL? `aws s3api get-object-acl --bucket bucket --key key`. (5) KMS key: does the role have `kms:Decrypt`?

---

**Q:** What is the command to manually scale an ECS Service?  
**A:**
```bash
# Increase desired count to 5
aws ecs update-service \
  --cluster my-cluster \
  --service my-service \
  --desired-count 5

# Check the deployment
aws ecs describe-services \
  --cluster my-cluster \
  --services my-service \
  --query 'services[0].[runningCount,desiredCount,deployments]'
```

---

**Q:** How do you configure CloudFront with an S3 origin using OAC (Origin Access Control)?  
**A:**
```hcl
resource "aws_cloudfront_origin_access_control" "oac" {
  name = "my-oac"
  origin_access_control_origin_type = "s3"
  signing_behavior = "always"
  signing_protocol = "sigv4"
}
# In the CloudFront distribution: origin with origin_access_control_id = oac.id
# Bucket Policy: allow only cloudfront.amazonaws.com with the condition
# "aws:SourceArn": "arn:aws:cloudfront::ACCOUNT:distribution/DISTID"
```

---

**Q:** How do you check the estimated cost of resources provisioned by Terraform?  
**A:** Option 1: **Infracost** (open-source tool): `infracost breakdown --path .` — shows the estimated monthly cost per Terraform resource. Option 2: **AWS Pricing Calculator** (manual). Option 3: **Terraform Cloud** has built-in cost estimation. Option 4: after `terraform apply`, check AWS Cost Explorer using the project's tags.

---

**Q:** How do you create a Secret in Secrets Manager and retrieve it with Lambda?  
**A:**
```bash
# Create secret
aws secretsmanager create-secret \
  --name prod/db/credentials \
  --secret-string '{"username":"admin","password":"supersecret"}'
```
```python
# Lambda Python
import boto3, json
def lambda_handler(event, context):
    sm = boto3.client('secretsmanager')
    secret = sm.get_secret_value(SecretId='prod/db/credentials')
    creds = json.loads(secret['SecretString'])
    # use creds['username'] and creds['password']
```
The Lambda Role needs `secretsmanager:GetSecretValue`.
