# Flashcards — EC2 Compute

> Review one card at a time. Try to answer before revealing the answer.

---

**Q:** What are the main EC2 instance families and their use cases?  
**A:** M (general purpose, balanced CPU/RAM), C (compute-optimized: HPC, batch, ML inference), R (memory-optimized: in-memory DB, cache), I (storage-optimized: local NVMe, high IOPS DB), P/G (GPU: ML training, rendering), T (burstable: apps that rarely need peaks, but have a low baseline).

---

**Q:** What is the CPU credits mechanism in T instances?  
**A:** T instances accumulate credits when CPU is below the baseline and spend them when above. T3/T3a in Unlimited mode can use CPU beyond the accumulated credits by paying for extra credits. In Standard mode with zero credits, CPU is limited to the baseline.

---

**Q:** What is the difference between On-Demand and Reserved Instance?  
**A:** On-Demand: pay per hour/second, no commitment, most expensive. Reserved Instance: 1- or 3-year commitment, discount of up to 72%. Standard RI cannot change type, Region or OS. Convertible RI can change attributes, lower discount (~54%). Both can be paid all upfront, partial upfront or monthly.

---

**Q:** What is the difference between Compute Savings Plans and EC2 Instance Savings Plans?  
**A:** Compute Savings Plans: up to 66% discount, applies to any family, Region, OS and size, also covers Lambda and Fargate. EC2 Instance Savings Plans: up to 72%, locked to a specific family + Region (e.g., m5 in us-east-1), higher discount, less flexibility.

---

**Q:** What is a Spot Instance and when should you use it?  
**A:** Spot Instances use AWS spare capacity at up to 90% discount. They can be interrupted with a 2-minute warning. Ideal for: fault-tolerant batch jobs, distributed ML training, rendering, testing. Do NOT use for: primary database, stateful web sessions.

---

**Q:** What is Spot Fleet and how does it work?  
**A:** Spot Fleet manages a set of Spot Instances (and optionally On-Demand). You define a pool of types/Regions and the strategy: `lowestPrice` (picks the cheapest), `capacityOptimized` (picks the pool with the most available capacity — recommended to reduce interruptions), `diversified` (distributes across all pools).

---

**Q:** What is the difference between Dedicated Host and Dedicated Instance?  
**A:** Dedicated Host: you rent a specific physical server, have socket/core visibility, and can use BYOL licenses (per socket/core such as SQL Server, Windows Server). Dedicated Instance: instance on hardware dedicated to your account but without control of the physical host. Host is more expensive but necessary for licensing compliance.

---

**Q:** What is IMDSv2 and why is it preferable to IMDSv1?  
**A:** IMDSv2 (Instance Metadata Service v2) uses a token-based model: first you make a PUT request to obtain a token with a TTL, then you use the token in the `X-aws-ec2-metadata-token` header. It protects against SSRF attacks where a process on the instance could capture metadata without the token. To disable IMDSv1, set `HttpTokens: required`.

---

**Q:** What is the difference between gp2 and gp3 in EBS?  
**A:** gp2: IOPS tied to size (3 IOPS/GB, max 16,000), burst. gp3: baseline of 3,000 IOPS and 125 MB/s regardless of size, can provision up to 16,000 IOPS and 1,000 MB/s separately. gp3 is cheaper (~20%) and more flexible — use it by default.

---

**Q:** When should you use io1/io2 vs gp3?  
**A:** io2: up to 64,000 IOPS per volume, io2 Block Express up to 256,000 IOPS, 99.999% durability (vs 99.8-99.9% for gp). Use for high-performance databases (Oracle, SQL Server, workloads that need >16,000 guaranteed IOPS). io1: previous version, 99.8% durability.

---

**Q:** What is Instance Store and what are its characteristics?  
**A:** Ephemeral NVMe storage directly on the instance's physical host. Very high IOPS (e.g., i3.large has ~100K IOPS). Data is LOST on: stop, terminate or hardware failure. Persists on: reboot. Ideal for high-performance cache, temporary buffer, scratch disk.

---

**Q:** What are the three types of Placement Group?  
**A:** Cluster: all instances on the same rack and AZ, <1ms latency between instances, ideal for HPC and MPI. Spread: each instance on a separate rack, maximum 7 per AZ, for critical high availability. Partition: groups of instances per partition (each partition = separate rack), up to 7 partitions per AZ, for HDFS/Cassandra/Kafka.

---

**Q:** What happens to an EC2 instance when we Stop it (vs Hibernate)?  
**A:** Stop: instance stops, RAM is lost, root EBS preserved, you are billed for EBS but not the instance. Hibernate: RAM is written to the root EBS volume (must have space and be encrypted), and on start the OS resumes from where it stopped. Hibernate limit: 60 days.

---

**Q:** What are AMIs and what are the virtualization categories?  
**A:** Amazon Machine Image is an instance template (OS, data, configurations). HVM (Hardware Virtual Machine): full hardware emulation — the only one supported by modern types. PV (Paravirtual): legacy. AMIs can be shared across accounts and copied across Regions. To copy an unencrypted AMI to an encrypted one, specify a CMK on the copy.

---

**Q:** What is EC2 User Data?  
**A:** Script (shell or cloud-init) executed only once at the instance's first boot with root privileges. Accessible at `http://169.254.169.254/latest/user-data`. Useful for installing software and configuring the application. It is not re-run on reboots (unless explicitly configured).

---

**Q:** How do you query an EC2 instance's metadata from inside it with IMDSv2?  
**A:** 
```bash
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id
```
Without the token (IMDSv1), just: `curl http://169.254.169.254/latest/meta-data/instance-id`

---

**Q:** What is a Capacity Reservation and when should you use it?  
**A:** Reserves capacity in a specific AZ with no price discount (you pay On-Demand). Guarantees capacity will be available when you need it. Can be combined with Savings Plans or Regional RI to get a discount. Useful for DR, compliance or critical workloads that cannot be left without capacity.

---

**Q:** What is the maximum IOPS of io2 Block Express, and how do st1 and sc1 compare?  
**A:** io2 Block Express: up to 256,000 IOPS, 4,000 MB/s. st1 (throughput): 40 MB/s per TB baseline, 250 MB/s per TB burst, max 500 MB/s — sequential access (logs, Kafka, data warehouse). sc1 (cold HDD): 12 MB/s per TB baseline, cheapest — infrequently accessed data.

---

**Q:** What is an Elastic IP and when should it be avoided?  
**A:** An Elastic IP is a static IPv4 address that can be associated/reassociated with instances or ENIs. Charged when it is NOT in use (0.005 USD/hour). Avoid in scalable architectures — use DNS (Route 53) instead of relying on a fixed IP. Use only when a client needs the IP on an allowlist.

---

**Q:** What is AWS Systems Manager Session Manager?  
**A:** Allows interactive access to EC2 instances without opening port 22 (SSH) or 3389 (RDP). The instance must have the SSM Agent installed and an IAM role with `AmazonSSMManagedInstanceCore`. Access via Console or CLI. Audited by CloudTrail and sessions can be logged to S3/CloudWatch Logs.
