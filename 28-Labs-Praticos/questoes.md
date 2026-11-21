# Questions — Module 18: Hands-On Labs

> **SAA-C03 Domain**: Hands-On Verification  
> **Difficulty**: Practical

---

**1.** After creating a VPC with the command `aws ec2 create-vpc --cidr-block 10.0.0.0/16`, you try to launch an EC2 instance in the new VPC but it cannot access the internet. Which step is most likely missing?

- A) Configure the VPC DNS
- B) Create an Internet Gateway and attach it to the VPC, and add a 0.0.0.0/0 route in the route table pointing to the IGW
- C) Enable IPv6 on the VPC
- D) Create a NAT Gateway before creating instances

<details><summary>Answer</summary>

**B** — A newly created VPC has no IGW. For internet access: (1) create an IGW, (2) attach it to the VPC, (3) create or modify a route table adding a 0.0.0.0/0 → IGW route, (4) associate the route table with the public subnet, (5) the instance needs a public IP (auto-assign or EIP). All of these steps are required.

</details>

---

**2.** You created a Lambda function that tries to connect to an RDS database in a private subnet, but it times out. The Lambda is in the same VPC. What is the first thing to check?

- A) The timeout configured on the Lambda (too low)
- B) Whether the RDS Security Group allows port 3306 traffic from the Lambda's Security Group
- C) Whether the Lambda has enough memory
- D) Whether RDS is Multi-AZ

<details><summary>Answer</summary>

**B** — For a Lambda in a VPC to access RDS, the RDS SG needs an inbound rule allowing the database port (MySQL: 3306, PostgreSQL: 5432) from the Lambda's SG (referenced by Security Group ID, not by IP). Checking SGs is always the first step in connectivity problems.

</details>

---

**3.** You ran `terraform apply` and created AWS resources. When trying to destroy them with `terraform destroy`, you get an error saying the S3 bucket cannot be deleted because it is not empty. What is the correct solution in Terraform?

- A) Manually delete the objects in the S3 console before running `terraform destroy`
- B) Add `force_destroy = true` to the `aws_s3_bucket` resource and re-apply before destroying
- C) Use `terraform state rm aws_s3_bucket.name` to remove it from the state
- D) Create a lifecycle rule on the bucket to expire the objects

<details><summary>Answer</summary>

**B** — `force_destroy = true` on the `aws_s3_bucket` resource allows Terraform to delete the bucket even with objects in it. Run `terraform apply` to update the resource with this setting, then `terraform destroy`. Removing it from the state (C) leaves the resource orphaned in AWS (it is not deleted).

</details>

---

**4.** You want to test whether an IAM policy works correctly before attaching it to a user. Which AWS CLI tool should you use?

- A) `aws iam simulate-principal-policy`
- B) `aws iam get-policy`
- C) `aws iam list-attached-user-policies`
- D) `aws sts assume-role`

<details><summary>Answer</summary>

**A** — `aws iam simulate-principal-policy`: simulates the IAM authorization result for a specific principal performing specific actions on specific resources. It returns Allow or Deny without performing the action. Perfect for testing policies before applying them. `simulate-custom-policy` tests a JSON policy without a specific user.

</details>

---

**5.** You are creating a Terraform module for an S3 bucket. The module must accept the bucket name as a required variable and create the bucket with versioning enabled. What is the correct syntax?

- A) 
```hcl
resource "aws_s3_bucket" "main" {
  bucket = var.bucket_name
}
resource "aws_s3_bucket_versioning" "main" {
  bucket = aws_s3_bucket.main.id
  versioning_configuration { status = "Enabled" }
}
```
- B) `aws s3api create-bucket --bucket ${var.bucket_name} --versioning Enabled`
- C) `resource "s3" "bucket" { name = var.bucket_name; versioning = true }`
- D) None of the above

<details><summary>Answer</summary>

**A** — In Terraform AWS Provider 4.x+ syntax, `aws_s3_bucket_versioning` is a separate resource (no longer a block inside `aws_s3_bucket`). The configuration in (A) is the correct, modern one. Option (C) has invalid syntax.

</details>

---

**6.** After deploying an ECS Fargate application, you notice the containers keep restarting constantly. Where is the first place to investigate?

- A) VPC Flow Logs
- B) The container's CloudWatch Logs (the task definition must have a logConfiguration with the awslogs driver)
- C) AWS X-Ray traces
- D) CloudTrail for the ECS service

<details><summary>Answer</summary>

**B** — Containers in a restart loop usually indicate an application error or misconfiguration. The container log (stdout/stderr) in CloudWatch Logs is the first place to look. The task definition must have `logConfiguration: awslogs` pointing to a Log Group. Without it, logs are lost and debugging is impossible.

</details>

---

**7.** You are using the AWS CLI and get the error `An error occurred (AccessDenied) when calling the DescribeInstances operation`. How do you diagnose which permission is missing?

- A) Check `aws iam list-attached-user-policies` to see the user's policies
- B) Use `aws iam simulate-principal-policy --action-names ec2:DescribeInstances` to simulate the authorization
- C) Check `aws sts get-caller-identity` to confirm which identity is being used, then check that identity's policies
- D) C is the first step; then B to confirm the exact policy that is denying

<details><summary>Answer</summary>

**D** — Correct flow: first `aws sts get-caller-identity` to confirm you are authenticated as the correct identity (not as another role). Then `simulate-principal-policy` to understand which statement is denying the action. `list-attached-user-policies` does not show group policies or inline policies.

</details>

---

**8.** You created an RDS MySQL database with `--no-publicly-accessible` in a private subnet. The application on EC2 in the same VPC cannot connect. What should you check? (Select 2)

- A) The RDS Security Group allows inbound on port 3306 from the EC2 Security Group
- B) The EC2 instance is in the same VPC as RDS (or in VPCs connected by peering/TGW)
- C) RDS has an Elastic IP associated
- D) The RDS subnet has a route to the Internet Gateway

<details><summary>Answer</summary>

**A and B** — For EC2 → RDS connectivity in the same VPC: (1) both in the same VPC; (2) the RDS SG allows port 3306 from the EC2 SG. A private RDS needs neither an EIP (C) nor an IGW route (D) — those are for public access.

</details>

---

**9.** You want to query DynamoDB returning only items where `status = "active"` and `age > 25`. The table has `userId` as the partition key and `createdAt` as the sort key. Which DynamoDB operation should you use?

- A) GetItem with FilterExpression
- B) Query with FilterExpression
- C) Scan with FilterExpression (reads the entire table and filters)
- D) Batch GetItem

<details><summary>Answer</summary>

**C** — `status` and `age` are not keys of the table or of any described GSI/LSI. Query requires specifying the partition key. Scan reads the entire table and applies the FilterExpression (works but is expensive for large tables). The ideal solution would be to create a GSI with `status` as the partition key and then run a Query, but with the indexes described, Scan is the only option.

</details>

---

**10.** You created a Lambda function with code that works locally, but when invoking it on AWS you get `Runtime.ImportModuleError: No module named 'boto3'`. What is the probable cause?

- A) Boto3 must be installed separately in Python Lambdas
- B) The deployment zip does not include the dependencies correctly
- C) The selected Python runtime does not support boto3
- D) Boto3 is already included in the Lambda Python runtime; the problem is another module being imported

<details><summary>Answer</summary>

**D** — Boto3 IS already included in the Lambda Python runtime (pre-installed by AWS). If the error says `No module named 'boto3'`, it is possible the error actually comes from another module listed earlier, or that the zip was created incorrectly, overriding the runtime's boto3 with an incomplete local version. Check that the deployment package is correct.

</details>

---

**11.** After creating a CloudFront distribution with an S3 bucket as the origin, you can still access S3 objects directly via the bucket URL, bypassing CloudFront. How do you fix this?

- A) Configure the bucket policy to deny access to everyone except the CloudFront OAC (Origin Access Control)
- B) Disable static website hosting on S3
- C) Configure CORS on the S3 bucket
- D) Use S3 pre-signed URLs instead of CloudFront

<details><summary>Answer</summary>

**A** — OAC (Origin Access Control): creates an identity in CloudFront. The bucket policy is updated to allow only GetObject from the Principal `cloudfront.amazonaws.com` with an `AWS:SourceArn` condition for the distribution. Any direct access to the S3 URL returns 403.

</details>

---

**12.** You want to monitor how many 5xx responses your Lambda API is returning and create an alarm when it exceeds 10 in 5 minutes. Which CLI command sequence implements this?

- A) Create a Metric Filter on the Log Group + create an Alarm on the custom metric
- B) Use the standard API Gateway metrics (`5XXError`) directly to create the Alarm
- C) Create a CloudTrail Event Rule for status 500
- D) Use X-Ray with an error threshold

<details><summary>Answer</summary>

**B** — API Gateway automatically publishes the `5XXError` metric to CloudWatch (namespace `AWS/ApiGateway`). Create the Alarm directly on this metric: `aws cloudwatch put-metric-alarm --alarm-name api-5xx --namespace AWS/ApiGateway --metric-name 5XXError --dimensions Name=ApiName,Value=my-api --period 300 --evaluation-periods 1 --threshold 10 --comparison-operator GreaterThanThreshold --statistic Sum`.

</details>

---

**13.** You ran `terraform apply`, but the `aws_iam_role_policy_attachment` resource is not in your code. Terraform shows this resource in the state (`terraform state list`). What happens if you run `terraform apply` again?

- A) Terraform will delete the role policy attachment because it is not in the code
- B) Terraform does nothing with resources that are not in the code (it only manages what is in the code)
- C) You would need `terraform state rm` to remove the resource from the state
- D) Terraform will show an error about an orphaned resource

<details><summary>Answer</summary>

**A** — Terraform manages resources declaratively: desired state = what is in the code. If a resource is in the state but not in the code, the next `terraform apply` will delete that resource to align the state with the code. To remove it from the state without deleting the real resource: `terraform state rm resource.name`.

</details>

---

**14.** An S3 bucket event notification is configured to invoke a Lambda when objects are created. But when testing, the Lambda is not invoked. Which two items should you check?

- A) The Lambda's resource policy allowing S3 to invoke it (Principal: s3.amazonaws.com) + the notification configured on the correct bucket
- B) The Lambda's IAM Role with s3:GetObject + the Lambda's SG
- C) Bucket versioning enabled + Lambda in the same region as the bucket
- D) CloudWatch Logs enabled on the Lambda + S3 Access Logging

<details><summary>Answer</summary>

**A** — For S3 to invoke Lambda: (1) a resource-based policy on the Lambda (`aws lambda add-permission`) allowing `lambda:InvokeFunction` with Principal `s3.amazonaws.com` and an `ArnEquals` condition on the bucket ARN; (2) a notification configuration on the bucket pointing to the correct Lambda ARN. Without the resource policy, S3 gets access denied when trying to invoke.

</details>

---

**15.** You are using `aws s3 sync` to back up files from a server to S3. The command includes unnecessary files (node_modules, .git). How do you exclude these directories?

- A) `aws s3 sync . s3://my-bucket --exclude "node_modules/*" --exclude ".git/*"`
- B) `aws s3 sync . s3://my-bucket --ignore node_modules .git`
- C) Create a .s3ignore file
- D) `aws s3 sync . s3://my-bucket --filter "node_modules"`

<details><summary>Answer</summary>

**A** — `--exclude` with a glob pattern: `aws s3 sync . s3://my-bucket --exclude "node_modules/*" --exclude ".git/*"`. Each exclusion is a separate `--exclude`. The pattern uses glob (* and **). Note: if you want to include only certain types, use `--exclude "*" --include "*.js"`. S3 has no .s3ignore file.

</details>
