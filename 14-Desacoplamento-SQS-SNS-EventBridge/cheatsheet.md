# Cheatsheet — Module 09: Decoupling (SQS, SNS, EventBridge)

## SQS Standard vs FIFO

| | Standard Queue | FIFO Queue |
|---|---|---|
| Ordering | Best-effort (may reorder) | Exact (First-In-First-Out) |
| Delivery | At-least-once (may duplicate) | Exactly-once (with deduplication ID) |
| Throughput | Nearly unlimited | 300 TPS (3,000 with batching) |
| Deduplication | No | Yes (5-min window) |
| Message Groups | No | Yes (parallelism within FIFO) |
| Cost | Lower | Higher |

## SQS — Critical Parameters

| Parameter | Default | Maximum | Description |
|---|---|---|---|
| VisibilityTimeout | 30s | 12h | Time invisible after being read |
| Message Retention | 4 days | 14 days | Time in the queue before expiring |
| Max Message Size | 256 KB | 256 KB | Maximum payload (S3 for larger) |
| Long Polling waitTime | 0 (short) | 20s | Wait time for messages |
| Delay Queue | 0 | 15 min | Delay before becoming visible |
| Dead Letter Queue maxReceiveCount | - | Configurable | Attempts before going to the DLQ |

## SNS — Destinations by Type

| Destination | Standard Topic | FIFO Topic |
|---|---|---|
| SQS | Yes | SQS FIFO only |
| Lambda | Yes | No |
| HTTP/HTTPS | Yes | No |
| Email | Yes | No |
| SMS | Yes | No |
| Firehose | Yes | No |
| SQS FIFO | No | Yes |

## Kinesis Data Streams vs Firehose

| | Kinesis Data Streams | Kinesis Firehose |
|---|---|---|
| Management | Manual (shards, consumers) | Fully managed |
| Latency | ~200ms (real-time) | Near real-time (60s or 1 MB min. buffer) |
| Consumers | Multiple (KCL, Lambda, etc.) | Fixed destinations |
| Replay | Yes (1-365 days retention) | No |
| Destinations | Custom consumers | S3, Redshift, OpenSearch, Splunk, HTTP |
| Shards | Manual or Auto Scaling | Managed auto-scaling |
| Transformation | No (processed by the consumer) | Optional Lambda before the destination |

## SQS vs SNS vs EventBridge vs Kinesis

| | SQS | SNS | EventBridge | Kinesis |
|---|---|---|---|---|
| Model | Pull (consumer polls) | Push (delivers to subscribers) | Push (rules → targets) | Pull + Push (EFO) |
| Pattern | Task queue | Pub/Sub fan-out | Event routing | Data streaming |
| Retention | 1 min - 14 days | No retention | Archive (optional) | 1-365 days |
| Replay | No (DLQ for failures) | No | Yes (Archive + Replay) | Yes |
| Ordering | Optional FIFO | Optional FIFO | Not guaranteed | Per shard |
| Best for | Asynchronous workers | 1:N fan-out | Triggering actions on events | Stream analytics |

## Fan-Out Pattern with SNS → SQS
```
Producer
  └── SNS Topic (order.events)
        ├── SQS Queue → Consumer A (inventory)   [FilterPolicy: {eventType: [order.created]}]
        ├── SQS Queue → Consumer B (billing)     [FilterPolicy: {eventType: [order.created, order.completed]}]
        └── Lambda → Consumer C (audit)          [No filter - receives all]
```

## EventBridge Event Pattern (example)
```json
{
  "source": ["aws.rds"],
  "detail-type": ["RDS DB Instance Event"],
  "detail": {
    "EventID": ["RDS-EVENT-0006"],
    "SourceType": ["DB_INSTANCE"]
  }
}
```

## Quick Exam Tips
- SQS FIFO: the queue NAME must end in `.fifo`
- SQS DLQ: must be the same type (Standard DLQ for Standard; FIFO DLQ for FIFO)
- **Classic fan-out**: SNS → multiple SQS Queues (not multiple SNS → one SQS)
- Kinesis Shard = 1 MB/s in, 2 MB/s out (shared) or per consumer with Enhanced Fan-Out
- **Amazon MQ = lift-and-shift** of brokers using AMQP/MQTT/STOMP; does not scale infinitely like SQS
- EventBridge Partner Bus: SaaS (Salesforce, Datadog) → events straight into EventBridge with no polling code
- SQS Max Receive Count + DLQ = poison pill isolation (standard pattern for problematic messages)
- Kinesis Firehose: transforms with Lambda; loads to S3/Redshift/OpenSearch; near-real-time (buffer)
- For real-time events with multiple independent consumers: Kinesis Data Streams > SQS
- EventBridge Pipes = source → (optional filter + enrich) → target without a routing Lambda
