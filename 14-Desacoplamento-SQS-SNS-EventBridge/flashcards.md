# Flashcards — Module 09: Decoupling (SQS, SNS, EventBridge)

---

**Q:** What is the default SQS VisibilityTimeout and why is it important?
**A:** Default: **30 seconds** (range: 0s to 12h). After a message is read, it stays "invisible" for that time. If it is not deleted within the timeout, it reappears in the queue. It must be ≥ the maximum processing time to avoid duplicate reprocessing

---

**Q:** What is the maximum throughput of an SQS FIFO queue?
**A:** 300 TPS (transactions per second) without batching. With batching of up to 10 messages: **3,000 TPS**. If you need more at high scale → use SQS Standard (nearly unlimited) but you lose ordering guarantees

---

**Q:** What is the maximum message retention period in SQS?
**A:** **14 days** (default: 4 days). Minimum: 1 minute. After the period, messages are automatically deleted even if they were not consumed

---

**Q:** What is SQS Long Polling and why use it?
**A:** The consumer waits up to `waitTimeSeconds` (maximum **20 seconds**) for available messages before returning. Reduces empty requests (cost) and reduces latency. Short Polling returns immediately (even without messages) — more expensive and inefficient

---

**Q:** What is a Dead Letter Queue (DLQ) and when does a message go to it?
**A:** A separate queue for messages that failed processing. A message goes to the DLQ when its `ReceiveCount` exceeds the `maxReceiveCount` configured in the redrive policy. It serves to isolate "poison pills" (unprocessable messages) for analysis

---

**Q:** What is the maximum message size in SQS?
**A:** **256 KB** per message. For larger messages, use the "S3 + SQS" pattern: store the payload in S3 and put only the S3 object reference in the SQS message (via the SQS Extended Client Library)

---

**Q:** What is a Delay Queue in SQS?
**A:** A queue with a configured delivery delay (0 to **15 minutes**). Published messages stay invisible for that period before appearing to consumers. A Message Timer allows a per-message delay (overrides the queue delay)

---

**Q:** What is SNS Message Filtering and how does it work?
**A:** Each SNS subscription can have a FilterPolicy (JSON with conditions on message attributes). Only messages matching the filter are delivered to that subscription. Example: `{"eventType": ["order.completed", "order.refunded"]}` — the subscription only receives those types

---

**Q:** What is an SNS FIFO Topic? What are its limitations?
**A:** SNS FIFO guarantees ordering and deduplication. Only SQS FIFO Queues can subscribe to SNS FIFO Topics. Throughput limited to 300 TPS (3,000 with batching) — the same limit as SQS FIFO. It does not integrate directly with Lambda, email, or HTTP (only SQS FIFO)

---

**Q:** What is the difference between the EventBridge Default Bus, Custom Bus, and Partner Bus?
**A:** Default Bus: events from AWS services (CloudTrail, EC2, RDS, etc.). Custom Bus: events from your own application (PutEvents API). Partner Bus: events from SaaS partners (Datadog, Salesforce, Zendesk, etc.) directly into EventBridge with no need for API polling

---

**Q:** What is the EventBridge Schema Registry?
**A:** Automatically discovers and stores event schemas. Lets you generate binding code (Java, Python, TypeScript) to serialize/deserialize events in a type-safe way. Facilitates event-driven development with type safety

---

**Q:** What is the per-shard capacity in Kinesis Data Streams?
**A:** Input: **1 MB/s** or 1,000 records/s per shard. Output: **2 MB/s** per shard (shared among consumers) or 2 MB/s per consumer with Enhanced Fan-Out (dedicated HTTP/2 push)

---

**Q:** How long does Kinesis Data Streams retain data?
**A:** Default: **24 hours**. Can be extended up to **365 days** (at additional cost). Lets multiple consumers read the same stream at different times (replay)

---

**Q:** What is the latency difference between Kinesis Data Streams and Kinesis Firehose?
**A:** Kinesis Data Streams: ~200ms latency (real-time). Kinesis Firehose: **near real-time** — 60-second (minimum) buffer or 1 MB before delivering to the destination (S3, Redshift, OpenSearch, Splunk). Firehose does not allow direct access to the stream or replay

---

**Q:** What is the Kinesis Producer Library (KPL) and what is its benefit?
**A:** A high-level library for Kinesis producers. Implements batching (PutRecords), retry with backoff, compression, and built-in monitoring. Increases throughput and efficiency compared to the raw SDK. Requires decompression on the consumer via KCL

---

**Q:** For which use case is Amazon MQ the right service?
**A:** Lift-and-shift migration of existing applications that use message brokers with standard protocols: **AMQP, MQTT, STOMP, OpenWire**. Supports managed ActiveMQ and RabbitMQ. If you are building a new application, prefer SQS/SNS (more scalable and managed)

---

**Q:** What is the "fan-out" architecture pattern with SNS and SQS?
**A:** Publish 1 message to an SNS Topic → multiple SQS queues (or Lambda, HTTP) subscribe. Each subscriber processes independently, can have its own DLQ and its own retry. Decouples the publisher from multiple consumers

---

**Q:** What is "at-least-once delivery" in SQS Standard?
**A:** SQS Standard guarantees the message is delivered **at least once** — it may be delivered more than once (duplication is rare). Applications must be **idempotent** (process the same message multiple times without side effects). For "exactly once": use SQS FIFO with a deduplication ID

---

**Q:** How do you integrate SQS with Auto Scaling to scale consumers based on the queue?
**A:** CloudWatch Alarm based on `ApproximateNumberOfMessagesVisible` (or `NumberOfMessagesSent` / instances) → Step Scaling or Target Tracking on the Auto Scaling Group. Target: e.g., keep ≤ 100 messages per instance. ECS can also scale tasks using this custom metric

---

**Q:** What is an EventBridge Pipe?
**A:** A point-to-point connection between a source (SQS, Kinesis, DynamoDB Streams, Kafka) and a target (Lambda, Step Functions, EventBridge Bus), with optional filtering and enrichment in between. Simplifies event pipelines without an intermediate Lambda used only for routing
