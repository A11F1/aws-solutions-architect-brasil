# Hands-On Lab — AWS Glossary: Advanced IAM and Policy Investigation (Module 20)

> **Region:** us-east-1 | **Estimated cost:** ~$0 (IAM is free)  
> **Prerequisites:** AWS CLI configured, IAM permissions in the account

---

## Objective

Explore the IAM policy types in practice, simulate permission evaluations, create cross-account roles, and understand the AWS policy evaluation model.

---

## Part 1 — IAM Policy Types

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# 1. Identity-based Policy (inline on a user)
aws iam create-user --user-name "lab-glossario-dev"

# Attach managed policy
aws iam attach-user-policy \
  --user-name "lab-glossario-dev" \
  --policy-arn "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"

# Create customer managed policy
POLICY_ARN=$(aws iam create-policy \
  --policy-name "lab-s3-bucket-especifico" \
  --policy-document "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [{
      \"Sid\": \"LeituraEscritaBucketLab\",
      \"Effect\": \"Allow\",
      \"Action\": [\"s3:GetObject\", \"s3:PutObject\", \"s3:ListBucket\"],
      \"Resource\": [
        \"arn:aws:s3:::lab-glossario-${ACCOUNT_ID}\",
        \"arn:aws:s3:::lab-glossario-${ACCOUNT_ID}/*\"
      ]
    }]
  }" \
  --query 'Policy.Arn' --output text)

echo "Policy ARN: $POLICY_ARN"

# Attach to the user
aws iam attach-user-policy \
  --user-name "lab-glossario-dev" \
  --policy-arn "$POLICY_ARN"

# List the user's policies
echo "User's managed policies:"
aws iam list-attached-user-policies \
  --user-name "lab-glossario-dev" \
  --query 'AttachedPolicies[*].[PolicyName, PolicyArn]' \
  --output table
```

---

## Part 2 — Permissions Boundary

```bash
# Create a Permissions Boundary — defines the MAXIMUM the role can have
BOUNDARY_ARN=$(aws iam create-policy \
  --policy-name "lab-dev-boundary" \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "PermitirS3eDynamoDB",
        "Effect": "Allow",
        "Action": ["s3:*", "dynamodb:*", "cloudwatch:*", "logs:*"],
        "Resource": "*"
      },
      {
        "Sid": "DenyIAM",
        "Effect": "Deny",
        "Action": "iam:*",
        "Resource": "*"
      },
      {
        "Sid": "DenyEC2Caros",
        "Effect": "Deny",
        "Action": ["ec2:RunInstances"],
        "Resource": "arn:aws:ec2:*:*:instance/*",
        "Condition": {
          "StringNotLike": {
            "ec2:InstanceType": ["t3.*", "t4g.*"]
          }
        }
      }
    ]
  }' \
  --query 'Policy.Arn' --output text)

# Create a role WITH a permissions boundary
ROLE_ARN=$(aws iam create-role \
  --role-name "lab-dev-role-com-boundary" \
  --assume-role-policy-document "{
    \"Version\":\"2012-10-17\",
    \"Statement\":[{
      \"Action\":\"sts:AssumeRole\",
      \"Effect\":\"Allow\",
      \"Principal\":{\"AWS\":\"arn:aws:iam::${ACCOUNT_ID}:root\"}
    }]
  }" \
  --permissions-boundary "$BOUNDARY_ARN" \
  --query 'Role.Arn' --output text)

# Give the role AdministratorAccess (but the boundary limits it)
aws iam attach-role-policy \
  --role-name "lab-dev-role-com-boundary" \
  --policy-arn "arn:aws:iam::aws:policy/AdministratorAccess"

echo "Role with boundary created: $ROLE_ARN"
echo ""
echo "Even with AdministratorAccess, the role can NOT:"
echo "  - Perform IAM actions"
echo "  - Launch EC2 instance types other than t3.* and t4g.*"
```

---

## Part 3 — simulate-principal-policy

```bash
# Simulate whether the user can do PutObject on S3
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::${ACCOUNT_ID}:user/lab-glossario-dev" \
  --action-names "s3:PutObject" \
  --resource-arns "arn:aws:s3:::lab-glossario-${ACCOUNT_ID}/teste.txt" \
  --query 'EvaluationResults[0].[EvalActionName, EvalDecision]' \
  --output table

# Simulate whether it can create EC2
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::${ACCOUNT_ID}:user/lab-glossario-dev" \
  --action-names "ec2:RunInstances" \
  --resource-arns "arn:aws:ec2:us-east-1::instance/*" \
  --query 'EvaluationResults[0].[EvalActionName, EvalDecision]' \
  --output table

# Simulate multiple actions at once
aws iam simulate-principal-policy \
  --policy-source-arn "arn:aws:iam::${ACCOUNT_ID}:user/lab-glossario-dev" \
  --action-names \
    "s3:GetObject" \
    "s3:DeleteObject" \
    "dynamodb:PutItem" \
    "iam:CreateRole" \
    "lambda:InvokeFunction" \
  --resource-arns "arn:aws:s3:::lab-glossario-${ACCOUNT_ID}/*" \
  --query 'EvaluationResults[*].[EvalActionName, EvalDecision]' \
  --output table
```

---

## Part 4 — Cross-Account Role

```bash
# Role to be assumed by ANOTHER account (or by the same account for the demo)
CROSS_ROLE_ARN=$(aws iam create-role \
  --role-name "lab-cross-account-readonly" \
  --assume-role-policy-document "{
    \"Version\": \"2012-10-17\",
    \"Statement\": [{
      \"Effect\": \"Allow\",
      \"Principal\": {\"AWS\": \"arn:aws:iam::${ACCOUNT_ID}:root\"},
      \"Action\": \"sts:AssumeRole\",
      \"Condition\": {
        \"StringEquals\": {\"sts:ExternalId\": \"segredo-compartilhado-123\"}
      }
    }]
  }" \
  --query 'Role.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-cross-account-readonly" \
  --policy-arn "arn:aws:iam::aws:policy/ReadOnlyAccess"

echo "Cross-Account Role: $CROSS_ROLE_ARN"

# Assume the role (AssumeRole with ExternalId)
CREDS=$(aws sts assume-role \
  --role-arn "$CROSS_ROLE_ARN" \
  --role-session-name "lab-cross-session" \
  --external-id "segredo-compartilhado-123" \
  --duration-seconds 3600 \
  --query 'Credentials' --output json)

echo "Temporary credentials received:"
echo "$CREDS" | python3 -c "
import json, sys
c = json.load(sys.stdin)
print(f'  AccessKeyId: {c[\"AccessKeyId\"][:10]}...')
print(f'  Expiration: {c[\"Expiration\"]}')
"

# Use the assumed credentials
AWS_ACCESS_KEY_ID=$(echo "$CREDS" | python3 -c "import json,sys; print(json.load(sys.stdin)['AccessKeyId'])")
AWS_SECRET_ACCESS_KEY=$(echo "$CREDS" | python3 -c "import json,sys; print(json.load(sys.stdin)['SecretAccessKey'])")
AWS_SESSION_TOKEN=$(echo "$CREDS" | python3 -c "import json,sys; print(json.load(sys.stdin)['SessionToken'])")

# Verify identity with the assumed credentials
AWS_ACCESS_KEY_ID="$AWS_ACCESS_KEY_ID" \
AWS_SECRET_ACCESS_KEY="$AWS_SECRET_ACCESS_KEY" \
AWS_SESSION_TOKEN="$AWS_SESSION_TOKEN" \
aws sts get-caller-identity
```

---

## Part 5 — Review AWS Managed Policies

```bash
# List the most commonly used AWS managed policies
echo "=== Common AWS Managed Policies ==="
for POLICY in \
  "AmazonS3FullAccess" \
  "AmazonEC2ReadOnlyAccess" \
  "ReadOnlyAccess" \
  "AdministratorAccess" \
  "AWSLambdaBasicExecutionRole" \
  "AmazonDynamoDBFullAccess"; do

  PARN="arn:aws:iam::aws:policy/${POLICY}"
  VERSION=$(aws iam get-policy \
    --policy-arn "$PARN" \
    --query 'Policy.DefaultVersionId' --output text 2>/dev/null)
  
  if [ -n "$VERSION" ]; then
    echo ""
    echo "--- $POLICY ---"
    aws iam get-policy-version \
      --policy-arn "$PARN" \
      --version-id "$VERSION" \
      --query 'PolicyVersion.Document.Statement[*].[Effect, Action[0:3], Resource[0]]' \
      --output table 2>/dev/null | head -10
  fi
done
```

---

## Cleanup

```bash
# Detach and delete policies/roles/user
aws iam detach-user-policy \
  --user-name "lab-glossario-dev" \
  --policy-arn "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"

aws iam detach-user-policy \
  --user-name "lab-glossario-dev" \
  --policy-arn "$POLICY_ARN"

aws iam delete-policy --policy-arn "$POLICY_ARN"
aws iam delete-user --user-name "lab-glossario-dev"

aws iam detach-role-policy \
  --role-name "lab-dev-role-com-boundary" \
  --policy-arn "arn:aws:iam::aws:policy/AdministratorAccess"
aws iam delete-role --role-name "lab-dev-role-com-boundary"
aws iam delete-policy --policy-arn "$BOUNDARY_ARN"

aws iam detach-role-policy \
  --role-name "lab-cross-account-readonly" \
  --policy-arn "arn:aws:iam::aws:policy/ReadOnlyAccess"
aws iam delete-role --role-name "lab-cross-account-readonly"
```

---

## What You Learned

- **Identity-based policy:** attached to a user/role/group — defines what the identity CAN do
- **Resource-based policy:** attached to the resource (S3, SQS) — defines who CAN access the resource
- **Permissions Boundary:** does not grant access — it only lowers the ceiling; evaluation: `identity AND boundary`
- **`simulate-principal-policy`:** essential tool for debugging permissions without making real changes
- **AssumeRole:** swaps long-lived credentials for temporary ones; ExternalId prevents the confused deputy problem
- **Evaluation rule:** Explicit Deny > everything. No explicit Allow = implicit Deny = access denied
