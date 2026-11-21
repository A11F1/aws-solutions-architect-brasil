# Hands-On Lab — Real-World Use Cases: SNS Fan-out + Step Functions + EventBridge (Module 17)

> **Region:** us-east-1 | **Estimated cost:** ~$0.01 (Step Functions Express, SNS, SQS — lab volumes)  
> **Prerequisites:** AWS CLI configured, Terraform >= 1.5

---

## Objective

Implement common architectural patterns used in industry: asynchronous fan-out with SNS+SQS, process orchestration with Step Functions Express Workflows, and event-driven design with EventBridge.

---

## Part 1 — Terraform: SNS + SQS + Lambda Fan-out

```hcl
# main.tf
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" { region = "us-east-1" }

data "aws_caller_identity" "current" {}

locals {
  prefix = "lab-fanout"
  region = "us-east-1"
  account_id = data.aws_caller_identity.current.account_id
}

# IAM Role for Lambdas
resource "aws_iam_role" "lambda_exec" {
  name = "${local.prefix}-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "lambda.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "lambda_basic" {
  role       = aws_iam_role.lambda_exec.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

resource "aws_iam_role_policy" "lambda_sqs" {
  name = "${local.prefix}-sqs-policy"
  role = aws_iam_role.lambda_exec.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["sqs:ReceiveMessage", "sqs:DeleteMessage", "sqs:GetQueueAttributes"]
      Resource = "*"
    }]
  })
}

# Main SNS Topic
resource "aws_sns_topic" "pedidos" {
  name = "${local.prefix}-pedidos"
}

# SQS queue for inventory processing
resource "aws_sqs_queue" "estoque_dlq" {
  name                       = "${local.prefix}-estoque-dlq"
  message_retention_seconds  = 1209600
}

resource "aws_sqs_queue" "estoque" {
  name                       = "${local.prefix}-estoque"
  visibility_timeout_seconds = 60
  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.estoque_dlq.arn
    maxReceiveCount     = 3
  })
}

# SQS queue for sending email
resource "aws_sqs_queue" "email_dlq" {
  name = "${local.prefix}-email-dlq"
}

resource "aws_sqs_queue" "email" {
  name                       = "${local.prefix}-email"
  visibility_timeout_seconds = 60
  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.email_dlq.arn
    maxReceiveCount     = 3
  })
}

# SQS policies to receive from SNS
data "aws_iam_policy_document" "sqs_estoque" {
  statement {
    principals {
      type        = "Service"
      identifiers = ["sns.amazonaws.com"]
    }
    actions   = ["sqs:SendMessage"]
    resources = [aws_sqs_queue.estoque.arn]
    condition {
      test     = "ArnEquals"
      variable = "aws:SourceArn"
      values   = [aws_sns_topic.pedidos.arn]
    }
  }
}

resource "aws_sqs_queue_policy" "estoque" {
  queue_url = aws_sqs_queue.estoque.id
  policy    = data.aws_iam_policy_document.sqs_estoque.json
}

data "aws_iam_policy_document" "sqs_email" {
  statement {
    principals {
      type        = "Service"
      identifiers = ["sns.amazonaws.com"]
    }
    actions   = ["sqs:SendMessage"]
    resources = [aws_sqs_queue.email.arn]
    condition {
      test     = "ArnEquals"
      variable = "aws:SourceArn"
      values   = [aws_sns_topic.pedidos.arn]
    }
  }
}

resource "aws_sqs_queue_policy" "email" {
  queue_url = aws_sqs_queue.email.id
  policy    = data.aws_iam_policy_document.sqs_email.json
}

# SNS → SQS subscriptions
resource "aws_sns_topic_subscription" "para_estoque" {
  topic_arn = aws_sns_topic.pedidos.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.estoque.arn
  filter_policy = jsonencode({
    tipo = ["pedido_criado"]
  })
}

resource "aws_sns_topic_subscription" "para_email" {
  topic_arn = aws_sns_topic.pedidos.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.email.arn
}

# Inventory processing Lambda
data "archive_file" "lambda_estoque" {
  type        = "zip"
  output_path = "/tmp/lambda_estoque.zip"
  source {
    content  = <<PYTHON
import json

def handler(event, context):
    for record in event['Records']:
        body = json.loads(record['body'])
        msg = json.loads(body.get('Message', '{}'))
        print(f"[ESTOQUE] Decrementing inventory for product: {msg.get('produto_id', 'UNKNOWN')}")
        print(f"[ESTOQUE] Quantity: {msg.get('quantidade', 0)}")
    return {'statusCode': 200}
PYTHON
    filename = "handler.py"
  }
}

resource "aws_lambda_function" "estoque" {
  filename         = data.archive_file.lambda_estoque.output_path
  source_code_hash = data.archive_file.lambda_estoque.output_base64sha256
  function_name    = "${local.prefix}-estoque"
  role             = aws_iam_role.lambda_exec.arn
  handler          = "handler.handler"
  runtime          = "python3.12"
  timeout          = 30
}

resource "aws_lambda_event_source_mapping" "estoque" {
  event_source_arn = aws_sqs_queue.estoque.arn
  function_name    = aws_lambda_function.estoque.arn
  batch_size       = 5
}

# Email processing Lambda
data "archive_file" "lambda_email" {
  type        = "zip"
  output_path = "/tmp/lambda_email.zip"
  source {
    content  = <<PYTHON
import json

def handler(event, context):
    for record in event['Records']:
        body = json.loads(record['body'])
        msg = json.loads(body.get('Message', '{}'))
        print(f"[EMAIL] Sending confirmation to: {msg.get('email_cliente', 'no-email')}")
        print(f"[EMAIL] Order ID: {msg.get('pedido_id', 'NO-ID')}")
    return {'statusCode': 200}
PYTHON
    filename = "handler.py"
  }
}

resource "aws_lambda_function" "email" {
  filename         = data.archive_file.lambda_email.output_path
  source_code_hash = data.archive_file.lambda_email.output_base64sha256
  function_name    = "${local.prefix}-email"
  role             = aws_iam_role.lambda_exec.arn
  handler          = "handler.handler"
  runtime          = "python3.12"
  timeout          = 30
}

resource "aws_lambda_event_source_mapping" "email" {
  event_source_arn = aws_sqs_queue.email.arn
  function_name    = aws_lambda_function.email.arn
  batch_size       = 5
}

output "sns_topic_arn" { value = aws_sns_topic.pedidos.arn }
output "fila_estoque_url" { value = aws_sqs_queue.estoque.id }
output "fila_email_url" { value = aws_sqs_queue.email.id }
```

---

## Part 2 — Publish Messages and Observe the Fan-out

```bash
terraform init && terraform apply -auto-approve

SNS_ARN=$(terraform output -raw sns_topic_arn)

# Publish a message — goes to BOTH queues
aws sns publish \
  --topic-arn "$SNS_ARN" \
  --subject "New Order" \
  --message '{
    "pedido_id": "PED-001",
    "produto_id": "PROD-42",
    "quantidade": 2,
    "email_cliente": "joao@exemplo.com",
    "tipo": "pedido_criado"
  }' \
  --message-attributes '{
    "tipo": {
      "DataType": "String",
      "StringValue": "pedido_criado"
    }
  }'

# Publish without the attribute — goes only to the email queue (no filter)
aws sns publish \
  --topic-arn "$SNS_ARN" \
  --message '{
    "pedido_id": "PED-002",
    "produto_id": "PROD-10",
    "quantidade": 1,
    "email_cliente": "maria@exemplo.com"
  }'

# Check the Lambda logs
sleep 15  # wait for processing

LOG_GROUP_ESTOQUE="/aws/lambda/lab-fanout-estoque"
LOG_GROUP_EMAIL="/aws/lambda/lab-fanout-email"

aws logs filter-log-events \
  --log-group-name "$LOG_GROUP_ESTOQUE" \
  --filter-pattern "[ESTOQUE]" \
  --query 'events[*].message' \
  --output text

aws logs filter-log-events \
  --log-group-name "$LOG_GROUP_EMAIL" \
  --filter-pattern "[EMAIL]" \
  --query 'events[*].message' \
  --output text
```

---

## Part 3 — Step Functions Express Workflow

```bash
# IAM Role for Step Functions
SF_ROLE_ARN=$(aws iam create-role \
  --role-name "lab-sf-express-role" \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Action": "sts:AssumeRole",
      "Effect": "Allow",
      "Principal": {"Service": "states.amazonaws.com"}
    }]
  }' \
  --query 'Role.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-sf-express-role" \
  --policy-arn "arn:aws:iam::aws:policy/CloudWatchLogsFullAccess"

# Create State Machine (Express — low latency, cheaper)
SF_ARN=$(aws stepfunctions create-state-machine \
  --name "lab-workflow-pedido" \
  --type EXPRESS \
  --role-arn "$SF_ROLE_ARN" \
  --definition '{
    "Comment": "Order processing workflow",
    "StartAt": "ValidarPedido",
    "States": {
      "ValidarPedido": {
        "Type": "Task",
        "Resource": "arn:aws:states:::lambda:invoke",
        "Parameters": {
          "FunctionName": "lab-fanout-estoque",
          "Payload.$": "$"
        },
        "Retry": [{
          "ErrorEquals": ["Lambda.ServiceException", "Lambda.AWSLambdaException"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }],
        "Catch": [{
          "ErrorEquals": ["States.ALL"],
          "Next": "PedidoRejeitado",
          "ResultPath": "$.erro"
        }],
        "Next": "VerificarEstoque"
      },
      "VerificarEstoque": {
        "Type": "Choice",
        "Choices": [{
          "Variable": "$.quantidade",
          "NumericLessThanEquals": 0,
          "Next": "EstoqueInsuficiente"
        }],
        "Default": "ProcessarPagamento"
      },
      "ProcessarPagamento": {
        "Type": "Pass",
        "Parameters": {
          "pedido_id.$": "$.pedido_id",
          "status": "PAGAMENTO_APROVADO",
          "timestamp.$": "$$.Execution.StartTime"
        },
        "Next": "NotificarCliente"
      },
      "NotificarCliente": {
        "Type": "Task",
        "Resource": "arn:aws:states:::lambda:invoke",
        "Parameters": {
          "FunctionName": "lab-fanout-email",
          "Payload.$": "$"
        },
        "End": true
      },
      "EstoqueInsuficiente": {
        "Type": "Fail",
        "Error": "EstoqueInsuficiente",
        "Cause": "Product out of stock"
      },
      "PedidoRejeitado": {
        "Type": "Fail",
        "Error": "ValidacaoFalhou",
        "Cause": "Order failed validation"
      }
    }
  }' \
  --query 'stateMachineArn' --output text)

echo "State Machine: $SF_ARN"

# Run the workflow
EXEC_ARN=$(aws stepfunctions start-execution \
  --state-machine-arn "$SF_ARN" \
  --input '{
    "pedido_id": "PED-100",
    "produto_id": "PROD-42",
    "quantidade": 2,
    "email_cliente": "joao@exemplo.com"
  }' \
  --query 'executionArn' --output text)

# Wait and check the result
sleep 10
aws stepfunctions describe-execution \
  --execution-arn "$EXEC_ARN" \
  --query '[status, stopDate, output]' \
  --output text
```

---

## Cleanup

```bash
# Delete Step Functions
aws stepfunctions delete-state-machine --state-machine-arn "$SF_ARN"
aws iam detach-role-policy \
  --role-name "lab-sf-express-role" \
  --policy-arn "arn:aws:iam::aws:policy/CloudWatchLogsFullAccess"
aws iam delete-role --role-name "lab-sf-express-role"

# Destroy Terraform (SNS, SQS, Lambdas)
terraform destroy -auto-approve

rm -f /tmp/lambda_estoque.zip /tmp/lambda_email.zip
```

---

## What You Learned

- **SNS Fan-out:** one message → multiple consumers; filter policies filter by attribute
- **SQS as a buffer:** decouples SNS from Lambda, guarantees delivery even if Lambda fails
- **DLQ:** after `maxReceiveCount` attempts, the message goes to the DLQ — alert via CloudWatch
- **Step Functions Express:** ideal for high frequency, < 5 min; Standard: long-running, exactly-once
- **Retry + Catch:** Step Functions manages resilience without retry code in the Lambda
- **Choice State:** equivalent to `if/else` in the workflow — branches based on execution data
