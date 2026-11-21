# Flashcards — High Availability and Scalability

> Review one card at a time. Try to answer before revealing the answer.

---

**Q:** What is the fundamental difference between ALB and NLB?  
**A:** ALB operates at layer 7 (HTTP/HTTPS/HTTP2/gRPC), with content-based routing (path, host, header, query string). NLB operates at layer 4 (TCP/UDP/TLS), ultra-low latency, static IP per AZ, and preserves the client's source IP by default.

---

**Q:** What is GWLB (Gateway Load Balancer) and when should you use it?  
**A:** Layer 3, GENEVE protocol, designed for transparent insertion of security appliances (next-gen firewalls, IDS/IPS, deep packet inspection). It encapsulates traffic and sends it to the appliances (EC2 with NVA) in the target group. Use with VPC Routing and GWLB Endpoints.

---

**Q:** What is cross-zone load balancing and what is the default for each ELB type?  
**A:** Distributes traffic evenly across all registered instances, regardless of AZ. ALB: enabled by default, no extra charge. NLB/GWLB: disabled by default, enabling it charges for inter-AZ traffic. CLB: enabled by default via console, no charge.

---

**Q:** What is Connection Draining (Deregistration Delay)?  
**A:** Period (default 300s, 0–3600s) during which the ELB waits for in-flight connections to complete before deregistering/stopping an instance. During this period, the ALB stops sending new requests to the instance but keeps the existing ones. Reduce it for applications with short requests; increase it for long uploads.

---

**Q:** Which target types does the ALB support?  
**A:** EC2 instances (by instance ID), IP addresses (on-premises resources, other services with an IP), Lambda functions (for serverless backends), and other ALBs (for chained microservices architectures).

---

**Q:** Does the ALB preserve the client's source IP?  
**A:** The ALB does NOT preserve the source IP directly — it terminates the TCP connection and makes a new request to the target. The original client IP is in the `X-Forwarded-For` header. The NLB DOES preserve the source IP (layer 4 pass-through).

---

**Q:** What is the difference between Launch Template and Launch Configuration?  
**A:** Launch Configuration: legacy, immutable (cannot be edited, only a new one created), does not support multiple instance types, does not support a Spot and On-Demand mix. Launch Template: recommended, has versions ($Latest, $Default), supports mixed instances policy, Spot Fleet, placement groups, T2/T3 unlimited.

---

**Q:** What are the 4 Auto Scaling policies?  
**A:** 1) Target Tracking: keeps a metric at a target value (e.g., CPU 50%), creates alarms automatically. 2) Step Scaling: scales in steps based on metric ranges, requires a CloudWatch alarm. 3) Scheduled Scaling: based on date/time, for predictable patterns. 4) Predictive Scaling: uses ML to forecast load and pre-provisions.

---

**Q:** What is the cooldown period in Auto Scaling?  
**A:** Period (default 300s) after a scaling activity during which the ASG does not start another scaling action. Purpose: let the new instance start absorbing load before deciding to scale further. Target Tracking has a separate warmup period that replaces the cooldown for scale-out.

---

**Q:** What is a Lifecycle Hook in the ASG?  
**A:** Allows pausing the ASG during instance lifecycle transitions: `Pending:Wait` (before entering service) and `Terminating:Wait` (before being terminated). The hook can trigger Lambda, SNS, SQS or EventBridge. Default timeout: 3600s. To continue: `CompleteLifecycleAction(CONTINUE)` or `ABANDON`.

---

**Q:** What is the ASG's default termination policy?  
**A:** 1) Selects the AZ with the most instances (rebalance); 2) Instance with the oldest Launch Configuration/Template; 3) Instance closest to the next billing hour; 4) Random if tied. Other policies also exist: `OldestInstance`, `NewestInstance`, `ClosestToNextInstanceHour`, `AllocationStrategy`.

---

**Q:** What is Predictive Scaling and what is its requirement?  
**A:** Uses ML to analyze metric history and pre-provision capacity before the expected peak. Requires at least 14 days of metric history to generate forecasts. Can be used together with Target Tracking for proactive and reactive coverage.

---

**Q:** What is PrivateLink and how does the NLB relate to it?  
**A:** AWS PrivateLink allows exposing a service from one VPC to consumers in other VPCs/accounts without peering, via Interface Endpoints. The NLB is the prerequisite: the service must sit behind an NLB to be registered as a VPC Endpoint Service. Consumers create Interface VPC Endpoints that point to the service.

---

**Q:** How does the ALB health check differ from the ASG's?  
**A:** The ALB checks the health of instances in the target group (HTTP path, response code, interval). The ASG can use two types: EC2 (system status) or ELB (delegated to the ELB). In production, ALWAYS configure the ASG to use ELB health check, since it detects application failures that EC2 health check does not.

---

**Q:** What is the warm-up period in ASG Target Tracking?  
**A:** Period (in seconds) during which a new instance is not counted in the scaling metrics, giving it time to initialize. Prevents the ASG from interpreting a "warming up" instance as low performance and needlessly scaling out more.

---

**Q:** How do you implement blue/green deployment with ALB and ASG?  
**A:** Create two target groups: `tg-blue` (current version) and `tg-green` (new version). Configure a listener rule with weighted target groups (e.g., 95% blue, 5% green). Gradually increase the green weight. If successful, redirect 100% to green. If it fails, go back to 100% blue immediately.

---

**Q:** What are Sticky Sessions on the ALB?  
**A:** Ensures that requests from the same client go to the same instance for a period of time. Uses a cookie: `AWSALB` (generated by the ALB, duration 1s–7 days) or an application-defined cookie. Useful for stateful sessions. Can cause imbalance if one instance receives many long sessions.

---

**Q:** What are NLB Zonal DNS and static IP?  
**A:** Every NLB gets a DNS host name (e.g., `my-nlb-xxxx.elb.amazonaws.com`). Each enabled AZ has an NLB node with a static IP that can be assigned an Elastic IP. For client-side IP allowlisting, use the NLB's Elastic IP. DNS resolves to all nodes; for a fixed IP per AZ, use the IP directly.

---

**Q:** What is the difference between Step Scaling and Simple Scaling?  
**A:** Simple Scaling (legacy): triggers one action and waits for the cooldown before any other action. All or nothing. Step Scaling: allows multiple steps based on metric ranges without cooldown between different steps — reacts faster to gradual or sudden load changes.

---

**Q:** How does the Mixed Instance Policy work in the ASG with Spot?  
**A:** Defines a pool of instance types and the percentage of On-Demand vs Spot. The ASG tries to launch Spot of the cheapest type available. If interrupted, it tries another type from the pool. The `capacityOptimized` strategy minimizes interruptions by choosing pools with the most available capacity.
