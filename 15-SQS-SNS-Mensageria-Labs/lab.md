# Hands-On Lab — SQS and SNS: Queues, DLQ, and Fan-out (Module 10)

> **Region:** us-east-1 | **Estimated cost:** ~$0 (first million messages free/month)  
> **Prerequisites:** AWS CLI configured, Python 3.8+

---

## Objective

Practice SQS Standard and FIFO, Dead Letter Queue, visibility timeout, SNS fan-out, and SQS → Lambda integration.

---

## Part 1 — SQS Standard + DLQ

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Create the DLQ first
DLQ_URL=$(aws sqs create-queue \
  --queue-name "lab-pedidos-dlq" \
  --attributes "MessageRetentionPeriod=1209600" \
  --query 'QueueUrl' --output text)

DLQ_ARN=$(aws sqs get-queue-attributes \
  --queue-url "$DLQ_URL" \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' --output text)

echo "DLQ created: $DLQ_ARN"

# Create the main queue with a redrive policy
QUEUE_URL=$(aws sqs create-queue \
  --queue-name "lab-pedidos" \
  --attributes "{
    \"VisibilityTimeout\": \"30\",
    \"ReceiveMessageWaitTimeSeconds\": \"20\",
    \"MessageRetentionPeriod\": \"345600\",
    \"RedrivePolicy\": \"{\\\"deadLetterTargetArn\\\":\\\"${DLQ_ARN}\\\",\\\"maxReceiveCount\\\":\\\"3\\\"}\"
  }" \
  --query 'QueueUrl' --output text)

echo "Main queue created: $QUEUE_URL"

# Check attributes
aws sqs get-queue-attributes \
  --queue-url "$QUEUE_URL" \
  --attribute-names All \
  --query 'Attributes.[VisibilityTimeout, ReceiveMessageWaitTimeSeconds, RedrivePolicy]' \
  --output json | python3 -m json.tool
```

---

## Part 2 — Produce and Consume Messages

```python
# sqs_lab.py
import boto3
import json
import time

sqs = boto3.client('sqs', region_name='us-east-1')
QUEUE_URL = "REPLACE_WITH_QUEUE_URL"  # terraform output or CLI

# ──────────────────────────────────────────
# 1. Send messages in batch
# ──────────────────────────────────────────
pedidos = [
    {"pedido_id": f"PED-{i:03}", "cliente": f"CLI-{i % 5:03}",
     "produto": f"Produto {i}", "valor": i * 50.0}
    for i in range(10)
]

entries = [
    {
        'Id': str(i),
        'MessageBody': json.dumps(p),
        'MessageAttributes': {
            'prioridade': {
                'DataType': 'String',
                'StringValue': 'alta' if p['valor'] > 200 else 'normal'
            }
        }
    }
    for i, p in enumerate(pedidos)
]

# Batch of 10 (maximum per call)
resp = sqs.send_message_batch(QueueUrl=QUEUE_URL, Entries=entries)
print(f"Sent: {len(resp['Successful'])} messages")

# ──────────────────────────────────────────
# 2. Consume messages (long polling)
# ──────────────────────────────────────────
processadas = 0
MAX = 10

print("\nConsuming messages...")
while processadas < MAX:
    resp = sqs.receive_message(
        QueueUrl=QUEUE_URL,
        MaxNumberOfMessages=5,
        WaitTimeSeconds=5,
        MessageAttributeNames=['All']
    )
    
    mensagens = resp.get('Messages', [])
    if not mensagens:
        print("Queue empty.")
        break
    
    for msg in mensagens:
        corpo = json.loads(msg['Body'])
        prioridade = msg.get('MessageAttributes', {}).get('prioridade', {}).get('StringValue', 'normal')
        
        print(f"  Processing: {corpo['pedido_id']} | prioridade={prioridade}")
        time.sleep(0.1)  # simulate processing
        
        # Delete after processing
        sqs.delete_message(QueueUrl=QUEUE_URL, ReceiptHandle=msg['ReceiptHandle'])
        processadas += 1

print(f"\nTotal processed: {processadas}")

# ──────────────────────────────────────────
# 3. Simulate a failing message (goes to the DLQ)
# ──────────────────────────────────────────
sqs.send_message(
    QueueUrl=QUEUE_URL,
    MessageBody=json.dumps({"pedido_id": "PED-FALHA", "tipo": "invalido"})
)
print("\nFailure message sent (maxReceive=3 → DLQ)")
print("Receive without deleting 3x to send it to the DLQ...")

DLQ_URL = QUEUE_URL.replace('lab-pedidos', 'lab-pedidos-dlq')
for tentativa in range(3):
    msgs = sqs.receive_message(
        QueueUrl=QUEUE_URL,
        MaxNumberOfMessages=1,
        WaitTimeSeconds=1
    ).get('Messages', [])
    
    for msg in msgs:
        corpo = json.loads(msg['Body'])
        if corpo.get('tipo') == 'invalido':
            print(f"  Attempt {tentativa+1}: simulating failure (NOT deleting)")
            # Wait for the visibility timeout to expire (30s = wait or use ChangeMessageVisibility)
            sqs.change_message_visibility(
                QueueUrl=QUEUE_URL,
                ReceiptHandle=msg['ReceiptHandle'],
                VisibilityTimeout=0  # makes it visible immediately for the next attempt
            )
```

```bash
# Replace the URL and run
QUEUE_URL=$(aws sqs get-queue-url --queue-name "lab-pedidos" --query 'QueueUrl' --output text)
sed -i "s|REPLACE_WITH_QUEUE_URL|$QUEUE_URL|" sqs_lab.py
python3 sqs_lab.py
```

---

## Part 3 — SQS FIFO

```bash
# Create a FIFO queue (name must end in .fifo)
FIFO_URL=$(aws sqs create-queue \
  --queue-name "lab-pagamentos.fifo" \
  --attributes "{
    \"FifoQueue\": \"true\",
    \"ContentBasedDeduplication\": \"false\",
    \"VisibilityTimeout\": \"30\",
    \"ReceiveMessageWaitTimeSeconds\": \"10\",
    \"DeduplicationScope\": \"messageGroup\",
    \"FifoThroughputLimit\": \"perMessageGroupId\"
  }" \
  --query 'QueueUrl' --output text)

echo "FIFO created: $FIFO_URL"

# Send messages with MessageGroupId and MessageDeduplicationId
for i in 1 2 3 4 5; do
  aws sqs send-message \
    --queue-url "$FIFO_URL" \
    --message-body "{\"pagamento_id\": \"PAG-$i\", \"valor\": $((i*100))}" \
    --message-group-id "cliente-001" \
    --message-deduplication-id "pay-cli001-seq-$i"
done

# Messages arrive IN SEND ORDER for the same MessageGroupId
echo "5 payments sent in order for cliente-001"

# Consume from FIFO
aws sqs receive-message \
  --queue-url "$FIFO_URL" \
  --max-number-of-messages 5 \
  --query 'Messages[*].Body' \
  --output text
```

---

## Part 4 — SNS Fan-out to SQS

```bash
# Create the SNS Topic
SNS_ARN=$(aws sns create-topic \
  --name "lab-eventos-compra" \
  --query 'TopicArn' --output text)

# Create queues to subscribe
FILA_ESTOQUE_URL=$(aws sqs create-queue \
  --queue-name "lab-fila-estoque" \
  --query 'QueueUrl' --output text)
FILA_ESTOQUE_ARN=$(aws sqs get-queue-attributes \
  --queue-url "$FILA_ESTOQUE_URL" \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' --output text)

FILA_EMAIL_URL=$(aws sqs create-queue \
  --queue-name "lab-fila-email" \
  --query 'QueueUrl' --output text)
FILA_EMAIL_ARN=$(aws sqs get-queue-attributes \
  --queue-url "$FILA_EMAIL_URL" \
  --attribute-names QueueArn \
  --query 'Attributes.QueueArn' --output text)

# Policy for SQS to accept messages from SNS
for ARN in "$FILA_ESTOQUE_ARN" "$FILA_EMAIL_ARN"; do
  URL=$(aws sqs get-queue-url --queue-name "$(basename "${ARN%:*}" )" 2>/dev/null \
        || echo "$FILA_ESTOQUE_URL")
done

aws sqs set-queue-attributes \
  --queue-url "$FILA_ESTOQUE_URL" \
  --attributes "{
    \"Policy\": \"{\\\"Statement\\\":[{\\\"Effect\\\":\\\"Allow\\\",\\\"Principal\\\":{\\\"Service\\\":\\\"sns.amazonaws.com\\\"},\\\"Action\\\":\\\"sqs:SendMessage\\\",\\\"Resource\\\":\\\"${FILA_ESTOQUE_ARN}\\\",\\\"Condition\\\":{\\\"ArnEquals\\\":{\\\"aws:SourceArn\\\":\\\"${SNS_ARN}\\\"}}}]}\"
  }"

aws sqs set-queue-attributes \
  --queue-url "$FILA_EMAIL_URL" \
  --attributes "{
    \"Policy\": \"{\\\"Statement\\\":[{\\\"Effect\\\":\\\"Allow\\\",\\\"Principal\\\":{\\\"Service\\\":\\\"sns.amazonaws.com\\\"},\\\"Action\\\":\\\"sqs:SendMessage\\\",\\\"Resource\\\":\\\"${FILA_EMAIL_ARN}\\\",\\\"Condition\\\":{\\\"ArnEquals\\\":{\\\"aws:SourceArn\\\":\\\"${SNS_ARN}\\\"}}}]}\"
  }"

# Subscribe queues to the topic
aws sns subscribe \
  --topic-arn "$SNS_ARN" \
  --protocol sqs \
  --notification-endpoint "$FILA_ESTOQUE_ARN"

aws sns subscribe \
  --topic-arn "$SNS_ARN" \
  --protocol sqs \
  --notification-endpoint "$FILA_EMAIL_ARN"

# Publish a message — goes to BOTH queues
aws sns publish \
  --topic-arn "$SNS_ARN" \
  --message '{"compra_id":"C001","cliente":"joao@ex.com","total":350.00}' \
  --subject "New Purchase Confirmed"

sleep 5

# Check messages in the queues
echo "Inventory Queue:"
aws sqs receive-message --queue-url "$FILA_ESTOQUE_URL" \
  --query 'Messages[0].Body' --output text | python3 -m json.tool

echo "Email Queue:"
aws sqs receive-message --queue-url "$FILA_EMAIL_URL" \
  --query 'Messages[0].Body' --output text | python3 -m json.tool
```

---

## Cleanup

```bash
# Delete subscriptions and SNS
aws sns delete-topic --topic-arn "$SNS_ARN"
aws sqs delete-queue --queue-url "$FILA_ESTOQUE_URL"
aws sqs delete-queue --queue-url "$FILA_EMAIL_URL"
aws sqs delete-queue --queue-url "$QUEUE_URL"
aws sqs delete-queue --queue-url "$DLQ_URL"
aws sqs delete-queue --queue-url "$FIFO_URL"

rm -f sqs_lab.py
```

---

## What You Learned

- **Visibility Timeout:** during processing the message is "invisible"; if you don't delete it, it returns to the queue
- **Long Polling (WaitTimeSeconds=20):** reduces empty calls by 99% vs short polling — ALWAYS prefer it
- **DLQ:** after `maxReceiveCount` failures, the message is diverted to the DLQ; alarm on messages in the DLQ
- **FIFO:** guarantees ordering (strict FIFO), exactly-once via MessageDeduplicationId; max 300 TPS without batching
- **Standard:** at-least-once, out-of-order, practically unlimited throughput — for most cases
- **SNS fan-out:** publish once → multiple independent consumers; a DLQ on SQS protects against lost messages
