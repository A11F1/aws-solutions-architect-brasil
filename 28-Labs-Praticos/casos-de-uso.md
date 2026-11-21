# Real-World Use Cases — Hands-On Labs (Module 18)

## Case 1 — Debug: Lambda Cannot Access RDS

**Context:** The development team deployed a Lambda function that must write data to an RDS MySQL database inside a private VPC. The Lambda works locally but times out in production.

**Step-by-Step Diagnosis:**
```
SYMPTOM: Lambda invocation timeout (no explicit connection error)

STEP 1 — Check the Lambda VpcConfig:
aws lambda get-function-configuration \
  --function-name MyFunction \
  --query 'VpcConfig'

Expected result:
{
  "SubnetIds": ["subnet-aaa", "subnet-bbb"],
  "SecurityGroupIds": ["sg-lambda"],
  "VpcId": "vpc-xxx"
}
If SubnetIds is empty → Lambda is not in the VPC → Add VpcConfig

STEP 2 — Check the subnet type (must be PRIVATE):
aws ec2 describe-subnets \
  --subnet-ids subnet-aaa \
  --query 'Subnets[*].[SubnetId,MapPublicIpOnLaunch,AvailabilityZone]'

If MapPublicIpOnLaunch=True → public subnet → use a private subnet

STEP 3 — Check Security Groups:
Lambda SG: does it allow outbound to port 3306 to the RDS SG?
aws ec2 describe-security-groups --group-ids sg-lambda
→ Verify: Outbound rule: TCP 3306 → sg-rds (or CIDR)

RDS SG: does it allow inbound from the Lambda SG?
aws ec2 describe-security-groups --group-ids sg-rds  
→ Verify: Inbound rule: TCP 3306 → sg-lambda

STEP 4 — Check the Lambda IAM Role:
aws lambda get-function-configuration \
  --function-name MyFunction --query 'Role'
→ Role must have: AWSLambdaVPCAccessExecutionRole (to create the ENI)

STEP 5 — Check the Lambda timeout (must be > connection time):
aws lambda get-function-configuration \
  --function-name MyFunction --query 'Timeout'
→ Default is 3s. An RDS cold connection can take 1-2s. Increase to 30s.

SOLUTION APPLIED:
1. Lambda was in a PUBLIC subnet (no access to the private RDS)
2. Fix: move Lambda to private subnets with a NAT GW
3. Increase timeout from 3s to 30s
```

---

## Case 2 — Debug: S3 Returning 403 Forbidden

**Context:** An ECS application receives a 403 when trying to PutObject into an S3 bucket. It works on the developer's machine but not on AWS.

**Diagnosis:**
```
STEP 1 — Identify which identity is making the request:
On the local machine: uses the dev's personal credentials (with full permissions)
On ECS: uses the task's IAM Role (may lack permissions)

See which role the container is using:
curl http://169.254.170.2/v2/credentials  (only from inside the ECS container)
→ Returns temporaryCredentials with RoleArn

STEP 2 — Check the Task Role:
aws ecs describe-task-definition --task-definition MyTask \
  --query 'taskDefinition.taskRoleArn'
→ If null → no Task Role → no S3 permissions

Fix: create an IAM Role for the task with:
{
  "Effect": "Allow",
  "Action": ["s3:PutObject", "s3:GetObject"],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
Assign it as taskRoleArn in the task definition

STEP 3 — Check the Bucket Policy:
aws s3api get-bucket-policy --bucket my-bucket
→ If there is an explicit Deny for the role → fix the bucket policy
→ If Condition: aws:SourceVpc → Lambda/ECS must be in the correct VPC

STEP 4 — Check Block Public Access:
aws s3api get-public-access-block --bucket my-bucket
→ Not the problem when using an IAM Role (it is not public access)

STEP 5 — Check KMS (if the bucket uses SSE-KMS):
Does the role have kms:GenerateDataKey and kms:Decrypt permissions on the key?
aws kms list-grants --key-id arn:aws:kms:...:key/xxx

SOLUTION:
Task Role did not exist → create it and attach it with the correct permissions
```

---

## Case 3 — Deploying with Terraform: State Lock and Team Conflicts

**Context:** Team A and Team B both run `terraform apply` at the same time. Team B gets a state lock error. After unlocking, both had made conflicting changes to the state.

**Concept and Resolution:**
```
RECOMMENDED SETUP (S3 Backend + DynamoDB Lock):
# versions.tf
terraform {
  backend "s3" {
    bucket         = "terraform-states-prod"
    key            = "api-service/terraform.tfstate"  
    region         = "us-east-1"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:..."  # SSE-KMS
    dynamodb_table = "terraform-locks"  # for state locking
  }
}

# DynamoDB table (created manually or with a bootstrap script):
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

WHEN THE STATE IS STUCK LOCKED (apply was interrupted):
terraform force-unlock LOCK_ID
# LOCK_ID is shown in the error message

BEST PRACTICES FOR TEAMS:
1. Each team has its own state file (different keys in S3)
2. Or: use Terraform workspaces
3. Or: use Terraform Cloud/Enterprise (native locking)
4. A CI/CD pipeline (GitHub Actions) ensures only 1 apply runs at a time

DEBUGGING CONFLICTS:
terraform state list  (see which resources are in the state)
terraform state show aws_instance.web  (see the current state of the resource)
terraform import aws_instance.web i-1234567890  (import an existing resource)
```

---

## Case 4 — SAM: Lambda with a Dependency Layer

**Context:** A Python Lambda needs the pandas library (100 MB), which exceeds the 50 MB zip upload limit. The deployment is failing.

**Solution with SAM Layers:**
```
PROJECT STRUCTURE:
lambda-project/
  template.yaml
  src/
    app.py  (main code)
  layers/
    pandas-layer/
      python/
        requirements.txt  (pandas==2.1.0, numpy==1.26.0)

template.yaml:
Resources:
  PandasLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      LayerName: pandas-layer
      ContentUri: layers/pandas-layer/
      CompatibleRuntimes:
        - python3.12
    Metadata:
      BuildMethod: python3.12  # SAM build compiles dependencies

  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Handler: app.lambda_handler
      Runtime: python3.12
      Layers:
        - !Ref PandasLayer
      MemorySize: 512  # pandas needs enough memory
      Timeout: 60

COMMANDS:
sam build  (builds layer + code)
sam deploy --guided  (first time)

VERIFY:
aws lambda list-layers
aws lambda get-layer-version --layer-name pandas-layer --version-number 1

IMPORTANT LIMIT:
- Code + layers unzipped: maximum 250 MB
- If pandas is still > 250 MB: use a Lambda container image (up to 10 GB)
```

---

## Case 5 — Debugging an ECS Fargate Task That Stops Immediately

**Context:** An ECS Fargate task starts but stops within seconds with a STOPPED status. The console shows a generic reason. The team does not know what is wrong.

**Full Diagnosis:**
```
STEP 1 — See the stoppedReason:
aws ecs describe-tasks \
  --cluster my-cluster \
  --tasks arn:aws:ecs:...:task/TASK_ID \
  --query 'tasks[0].{stoppedReason:stoppedReason,status:lastStatus,containers:containers[*].{name:name,reason:reason,exitCode:exitCode}}'

COMMON CAUSES + SOLUTIONS:

A) "CannotPullContainerError: ... no such host"
   → ECR is in a private VPC without an endpoint
   Solution: Create a VPC Endpoint for ECR (ecr.api, ecr.dkr, s3)
   OR: Use a public subnet with a NAT GW

B) "CannotPullContainerError: 403 Forbidden"  
   → Task Execution Role lacks ECR permissions
   Solution: Add AmazonECRReadOnly to the Task Execution Role

C) exitCode: 1, reason: "Essential container exited"
   → The app crashed. Check logs in CloudWatch:
   aws logs tail /ecs/my-task --follow
   Check: environment variables, Secrets Manager secrets

D) "OutOfMemoryError: Container killed"
   → memoryReservation too low
   Solution: Increase memory in the task definition

E) "ResourceInitializationError: unable to pull secrets"
   → Secrets Manager or SSM Parameter Store is unreachable
   Solution: VPC Endpoint for secretsmanager or a route to the internet

STEP 2 — See detailed logs:
aws logs describe-log-groups | grep ecs
aws logs get-log-events \
  --log-group-name /ecs/my-task \
  --log-stream-name ecs/container-name/TASK_ID

STEP 3 — Test locally:
docker run -e DATABASE_URL=... -e SECRET=... my-image:latest
→ Reproducing the error locally is much faster
```
