# Hands-On Lab — DynamoDB: Modeling, GSI, DAX, and Streams (Module 07)

> **Region:** us-east-1 | **Estimated cost:** ~$0 (on-demand mode + DAX ~$0.08/h — destroy after the lab)  
> **Prerequisites:** AWS CLI configured, Python 3.8+, pip

---

## Objective

Practice NoSQL data modeling in DynamoDB, create a GSI, enable Streams, explore TTL, and understand the consistency model.

---

## Part 1 — Create a Table with GSI and TTL

```bash
# Table: orders (pedidos) with a GSI to query by customer
aws dynamodb create-table \
  --table-name "pedidos" \
  --attribute-definitions \
    "AttributeName=pedido_id,AttributeType=S" \
    "AttributeName=cliente_id,AttributeType=S" \
    "AttributeName=data_pedido,AttributeType=S" \
  --key-schema \
    "AttributeName=pedido_id,KeyType=HASH" \
  --billing-mode PAY_PER_REQUEST \
  --global-secondary-indexes '[
    {
      "IndexName": "cliente-data-index",
      "KeySchema": [
        {"AttributeName": "cliente_id", "KeyType": "HASH"},
        {"AttributeName": "data_pedido", "KeyType": "RANGE"}
      ],
      "Projection": {"ProjectionType": "ALL"}
    }
  ]' \
  --stream-specification "StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES"

# Enable TTL (expires_at field = Unix timestamp)
aws dynamodb update-time-to-live \
  --table-name "pedidos" \
  --time-to-live-specification "Enabled=true,AttributeName=expires_at"

aws dynamodb wait table-exists --table-name "pedidos"
echo "Table pedidos created!"

# Second table: cart (carrinho) with TTL (1h session)
aws dynamodb create-table \
  --table-name "carrinho" \
  --attribute-definitions \
    "AttributeName=sessao_id,AttributeType=S" \
    "AttributeName=item_id,AttributeType=S" \
  --key-schema \
    "AttributeName=sessao_id,KeyType=HASH" \
    "AttributeName=item_id,KeyType=RANGE" \
  --billing-mode PAY_PER_REQUEST

aws dynamodb update-time-to-live \
  --table-name "carrinho" \
  --time-to-live-specification "Enabled=true,AttributeName=expires_at"

aws dynamodb wait table-exists --table-name "carrinho"
echo "Table carrinho created!"
```

---

## Part 2 — CRUD Operations and Queries

```python
# dynamo_lab.py
import boto3
from boto3.dynamodb.conditions import Key, Attr
from datetime import datetime, timedelta, timezone
import uuid

dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
tabela_pedidos  = dynamodb.Table('pedidos')
tabela_carrinho = dynamodb.Table('carrinho')

# ──────────────────────────────────────────
# 1. PutItem — create orders
# ──────────────────────────────────────────
agora = datetime.utcnow()
pedidos_ids = []

for i, (cliente, produto, valor) in enumerate([
    ("CLI-001", "Notebook Pro",   3500.00),
    ("CLI-002", "Mouse Wireless",   89.90),
    ("CLI-001", "Teclado",         350.00),
    ("CLI-003", "Monitor",        1200.00),
    ("CLI-001", "Headset",         180.00),
]):
    pid = str(uuid.uuid4())[:8]
    pedidos_ids.append(pid)
    tabela_pedidos.put_item(Item={
        'pedido_id':   pid,
        'cliente_id':  cliente,
        'produto':     produto,
        'valor':       str(valor),
        'status':      'pendente' if i % 2 == 0 else 'aprovado',
        'data_pedido': (agora - timedelta(days=i)).strftime('%Y-%m-%dT%H:%M:%S'),
        'expires_at':  int((agora + timedelta(days=30)).timestamp()),
    })

print(f"Inserted {len(pedidos_ids)} orders")

# ──────────────────────────────────────────
# 2. GetItem — lookup by HASH key
# ──────────────────────────────────────────
resp = tabela_pedidos.get_item(
    Key={'pedido_id': pedidos_ids[0]},
    ConsistentRead=True  # Strongly consistent read
)
print(f"\nGetItem (consistent): {resp['Item']}")

# Without ConsistentRead = Eventually Consistent (default, cheaper)
resp2 = tabela_pedidos.get_item(Key={'pedido_id': pedidos_ids[0]})
print(f"GetItem (eventual): {resp2['Item']['produto']}")

# ──────────────────────────────────────────
# 3. Query on the GSI: all orders for CLI-001
# ──────────────────────────────────────────
resp = tabela_pedidos.query(
    IndexName='cliente-data-index',
    KeyConditionExpression=Key('cliente_id').eq('CLI-001'),
    ScanIndexForward=False  # descending order by date
)
print(f"\nQuery GSI (CLI-001): {len(resp['Items'])} orders")
for p in resp['Items']:
    print(f"  {p['data_pedido']} — {p['produto']} — {p['status']}")

# ──────────────────────────────────────────
# 4. Query with range: recent orders for CLI-001
# ──────────────────────────────────────────
data_corte = (agora - timedelta(days=3)).strftime('%Y-%m-%dT%H:%M:%S')
resp = tabela_pedidos.query(
    IndexName='cliente-data-index',
    KeyConditionExpression=(
        Key('cliente_id').eq('CLI-001') &
        Key('data_pedido').gt(data_corte)
    )
)
print(f"\nRecent orders CLI-001 (last 3 days): {len(resp['Items'])}")

# ──────────────────────────────────────────
# 5. UpdateItem — update status
# ──────────────────────────────────────────
tabela_pedidos.update_item(
    Key={'pedido_id': pedidos_ids[0]},
    UpdateExpression='SET #s = :ns, atualizado_em = :ts',
    ExpressionAttributeNames={'#s': 'status'},
    ExpressionAttributeValues={':ns': 'enviado', ':ts': agora.isoformat()},
    ConditionExpression=Attr('status').eq('pendente')  # Conditional write
)
print(f"\nStatus updated conditionally: order {pedidos_ids[0]} → shipped")

# ──────────────────────────────────────────
# 6. Scan with FilterExpression (less efficient than Query)
# ──────────────────────────────────────────
resp = tabela_pedidos.scan(
    FilterExpression=Attr('status').eq('aprovado'),
    Select='COUNT'
)
print(f"\nScan (status=aprovado): {resp['Count']} orders")

# ──────────────────────────────────────────
# 7. Cart with TTL (expires in 1 hour)
# ──────────────────────────────────────────
sessao = str(uuid.uuid4())[:8]
expira = int((agora + timedelta(hours=1)).timestamp())

for item_id, produto, qtd in [("PROD-1", "Notebook", 1), ("PROD-2", "Mouse", 2)]:
    tabela_carrinho.put_item(Item={
        'sessao_id':  sessao,
        'item_id':    item_id,
        'produto':    produto,
        'quantidade': qtd,
        'expires_at': expira,
    })

print(f"\nCart {sessao}: 2 items (TTL in 1 hour)")

# ──────────────────────────────────────────
# 8. BatchWrite
# ──────────────────────────────────────────
with tabela_pedidos.batch_writer() as batch:
    for j in range(10):
        batch.put_item(Item={
            'pedido_id': f'batch-{j:03}',
            'cliente_id': f'CLI-{j % 3:03}',
            'produto': f'Produto {j}',
            'valor': str(j * 10.0),
            'status': 'pendente',
            'data_pedido': agora.isoformat(),
            'expires_at': int((agora + timedelta(days=7)).timestamp()),
        })
print("BatchWrite: 10 orders inserted")
```

```bash
python3 dynamo_lab.py
```

---

## Part 3 — DynamoDB Streams + Lambda Trigger

```bash
# Get the Stream ARN
STREAM_ARN=$(aws dynamodb describe-table \
  --table-name "pedidos" \
  --query 'Table.LatestStreamArn' --output text)

echo "Stream ARN: $STREAM_ARN"

# Create a Lambda that processes the stream
LAMBDA_ROLE=$(aws iam create-role \
  --role-name "lab-dynamo-stream-role" \
  --assume-role-policy-document '{"Version":"2012-10-17","Statement":[{"Action":"sts:AssumeRole","Effect":"Allow","Principal":{"Service":"lambda.amazonaws.com"}}]}' \
  --query 'Role.Arn' --output text)

aws iam attach-role-policy \
  --role-name "lab-dynamo-stream-role" \
  --policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaDynamoDBExecutionRole"

sleep 10

zip -j /tmp/stream_handler.zip - << 'PYTHON'
import json

def handler(event, context):
    for record in event['Records']:
        if record['eventName'] == 'INSERT':
            novo = record['dynamodb']['NewImage']
            print(f"NEW ORDER: id={novo['pedido_id']['S']} cliente={novo.get('cliente_id',{}).get('S')}")
        elif record['eventName'] == 'MODIFY':
            antigo = record['dynamodb'].get('OldImage', {})
            novo = record['dynamodb']['NewImage']
            old_status = antigo.get('status', {}).get('S', '?')
            new_status = novo.get('status', {}).get('S', '?')
            if old_status != new_status:
                print(f"STATUS CHANGED: {old_status} → {new_status}")
        elif record['eventName'] == 'REMOVE':
            print(f"ITEM REMOVED (TTL expired)")
    return {'batchItemFailures': []}
PYTHON

FUNC_ARN=$(aws lambda create-function \
  --function-name "lab-dynamo-stream-processor" \
  --runtime python3.12 \
  --role "$LAMBDA_ROLE" \
  --handler stream_handler.handler \
  --zip-file fileb:///tmp/stream_handler.zip \
  --query 'FunctionArn' --output text)

# Connect the stream to the Lambda
aws lambda create-event-source-mapping \
  --event-source-arn "$STREAM_ARN" \
  --function-name "lab-dynamo-stream-processor" \
  --starting-position TRIM_HORIZON \
  --batch-size 10

echo "Stream trigger created!"
sleep 15

# Check the Lambda logs (batch write events)
aws logs filter-log-events \
  --log-group-name "/aws/lambda/lab-dynamo-stream-processor" \
  --filter-pattern "NEW ORDER" \
  --query 'events[*].message' \
  --output text | head -10
```

---

## Cleanup

```bash
# Delete Lambda and trigger
aws lambda delete-function --function-name "lab-dynamo-stream-processor"
aws iam detach-role-policy \
  --role-name "lab-dynamo-stream-role" \
  --policy-arn "arn:aws:iam::aws:policy/service-role/AWSLambdaDynamoDBExecutionRole"
aws iam delete-role --role-name "lab-dynamo-stream-role"

# Delete tables
aws dynamodb delete-table --table-name "pedidos"
aws dynamodb delete-table --table-name "carrinho"

rm -f dynamo_lab.py /tmp/stream_handler.zip
```

---

## What You Learned

- **Partition Key** distributes data; **Sort Key** enables range queries within a partition
- **GSI:** a new view of the table with different keys; eventually consistent by default
- **Scan vs Query:** Scan reads EVERYTHING and then filters (expensive); Query uses the index (efficient)
- **TTL:** automatic, free expiration via a Unix timestamp attribute — not immediate (up to 48h)
- **Streams:** immutable change log; integrates with Lambda for real-time reaction (event sourcing)
- **ConditionExpression:** atomic, idempotent write — prevents lost updates under concurrency
