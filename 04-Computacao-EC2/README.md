# 03 EC2 Compute

## 📋 Table of Contents
- [Module Objectives](#-module-objectives)
- [Core Concepts](#-core-concepts)
- [Architecture and Components](#-architecture-and-components)
- [Important Settings for the Exam](#-important-settings-for-the-exam)
- [Service Comparison](#-service-comparison)
- [Exam Tips and Pitfalls](#-exam-tips-and-pitfalls)
- [For the Exam](#-for-the-exam)
- [Related Links](#-related-links)

## 🎯 Module Objectives

By the end of this module, you should be able to:

- identify the most suitable EC2 instance family for a given workload
- compare purchasing models and select the most cost-effective one for each usage profile
- differentiate EBS gp3, io2, st1 and sc1 by performance and cost characteristics
- explain Instance Store vs EBS and the implications of each on failure
- choose the correct placement group type for each latency or isolation requirement
- describe IMDSv2 and User Data and apply them to instance automation questions

## 📚 Core Concepts

### Instance Families

AWS organizes EC2 instance types into families, and each family is optimized for a hardware characteristic. Recognizing the family by its first letter is essential for the exam.

**M — General Purpose (balanced)**: balanced ratio of CPU and memory. They are the most common choice for web applications, mid-sized databases and development environments. Example: m7g.xlarge.

**C — Compute Optimized**: high CPU performance. Suited for batch processing, video encoding, HPC servers, game servers and compute-intensive microservices. Example: c7g.large.

**R — Memory Optimized**: large amount of RAM relative to CPU. Used for in-memory databases such as Redis and Memcached, SAP HANA, and real-time analytics. Example: r7g.2xlarge.

**I — Storage Optimized**: high IOPS and low local I/O latency. Suited for local NoSQL databases, data warehousing, and distributed file systems such as HDFS. Use when temporary data needs ultra-fast access. Example: i4i.xlarge.

**P/G — Accelerated Computing**: GPUs for machine learning, 3D rendering and scientific simulations. P is geared toward ML training and G toward inference and graphics. Example: p4de.24xlarge.

**T — Burstable**: baseline performance with accumulable credits for spikes. Ideal for intermittent loads such as dev/test environments and low-traffic microservices. T3 and T4g have an unlimited mode for unlimited credits at an additional cost. Caution: continuous high-CPU workloads exhaust the credits and performance drops to the baseline.

### Purchasing Models

The purchasing model is one of the topics most frequently tested in cost optimization questions.

**On-Demand**: pay per second or hour with no commitment. Highest unit cost, no planning required. Suited for unpredictable workloads, development and testing.

**Reserved Instances (RI)**: 1- or 3-year commitment with a discount of up to 72% vs On-Demand. There are three variants: Standard RI (highest discount, fixed type and Region), Convertible RI (lower discount, can change type/OS within the term), and Zonal RI (reserves capacity in a specific AZ). Scheduled RI has been discontinued.

**Savings Plans**: more flexible than Reserved Instances. You commit to an hourly spend in dollars for 1 or 3 years. Compute Savings Plans (lower discount, cover EC2, Lambda and Fargate, any type, Region and OS) vs EC2 Savings Plans (higher discount, but tied to type and Region). There is no concept of a "specific instance"; the discount is applied automatically.

**Spot Instances**: purchase of AWS spare capacity at a discount of up to 90%. AWS can interrupt the instance with a 2-minute warning when it needs the capacity back. Ideal for fault-tolerant workloads: batch, data processing, rendering, CI/CD. Should not be used for stateful databases or applications without checkpointing.

**Spot Fleet**: a collection of Spot Instances and optionally On-Demand Instances with a target capacity. You define multiple launch pools (type, AZ) and the Fleet selects the cheapest. Strategies: lowestPrice, diversified, capacityOptimized, priceCapacityOptimized (recommended by AWS).

**Dedicated Hosts**: dedicated physical server. You get visibility into sockets/cores, useful for per-socket licensing (SQL Server, Oracle, Windows Server). Can be On-Demand or Reserved.

**Dedicated Instances**: instances that run on dedicated hardware, but without control over the specific host. Covers compliance that requires physical isolation without needing socket visibility.

**Capacity Reservations**: reserve capacity in a specific AZ with no discount commitment. Combine with Savings Plans or Regional RI to cover the cost.

### EC2 User Data and Instance Metadata

**User Data** is a script that runs only once, at the instance's first boot (via cloud-init). It is used to install packages, configure services and download artifacts. It runs as root. To run on every boot, you must explicitly configure it through cloud-init or use MIME multi-part.

**Instance Metadata Service (IMDS)** exposes information about the instance at the special endpoint `http://169.254.169.254`. The second version, IMDSv2, is session-oriented and requires a prior call to obtain a token with a TTL. The token is then used in the `X-aws-ec2-metadata-token` header in subsequent calls. IMDSv2 protects against SSRF, because the token cannot be captured by an attacker who manages to execute a request via SSRF. The exam tests when to use IMDSv2: whenever the scenario mentions enhanced metadata security or protection against SSRF on EC2 instances.

### AMIs

Amazon Machine Images are templates that define an instance's operating system, software configuration and device mapping. They can be created from running instances (snapshot of the EBS volumes) or directly from EBS volumes.

An AMI can be copied across Regions, which facilitates multi-Region deployment. It can be shared with other accounts or made public. Copying an AMI creates a new EBS snapshot in the destination Region. Encrypted AMIs can only be copied to accounts that have access to the CMK used.

For the exam, AMIs usually appear in the context of launch templates, golden images for consistent bootstrapping, and fast recovery strategies.

### Amazon EBS — Elastic Block Store

EBS provides persistent block volumes for EC2 instances. The volume exists independently of the instance — even if the instance is terminated, the volume can be preserved.

**gp3**: current-generation general purpose SSD. Baseline of 3,000 IOPS and 125 MBps regardless of size. IOPS and throughput can be scaled separately from size up to 16,000 IOPS and 1,000 MBps. Lower cost than gp2 for additional IOPS.

**gp2**: previous generation. IOPS tied to size: 3 IOPS/GB, minimum 100, maximum 16,000 (requires a 5,333 GB volume). Can burst to 3,000 IOPS using credits. Prefer gp3 for new volumes.

**io2 Block Express**: very high-performance provisioned IOPS. 99.999% durability, consistent latency, up to 256,000 IOPS and 4,000 MBps. Supports Multi-Attach (multiple instances on the same volume — cluster-aware FS required).

**io1**: previous-generation provisioned IOPS. Up to 64,000 IOPS. Prefer io2 for new use cases.

**st1**: throughput-optimized HDD. Good for large sequential data volumes: logs, data lake, Hadoop. Up to 500 MBps. Cannot be a boot volume.

**sc1**: cold HDD. Lowest EBS cost. For infrequently accessed data. Up to 250 MBps. Cannot be a boot volume.

EBS snapshots are incremental and stored in S3 (transparent to the user). The first snapshot copies everything; subsequent ones copy only the changed blocks. They can be copied across Regions. Encryption is per volume: if the CMK is specified when the volume is created, all snapshots inherit the encryption.

### Instance Store

Instance Store is temporary storage directly on the physical disks of the server hosting the instance. Very low latency and very high IOPS (NVMe). The drawback is that data is lost when the instance is stopped, hibernated or terminated. Use it for cache, buffers and temporary data that can be recreated — never for permanent data.

### Placement Groups

**Cluster Placement Group**: packs instances physically close together in a rack or group of racks in the same AZ. Minimizes network latency (10 Gbps between instances). Risk: a hardware failure can affect all instances. Suited for HPC, low-latency applications and heavy traffic between nodes.

**Spread Placement Group**: spreads instances across distinct hardware. Maximum of 7 instances per AZ per placement group. Maximizes availability: a rack failure affects at most one instance. Suited for critical applications where each instance must not share a point of failure.

**Partition Placement Group**: divides instances into partitions, each on a separate rack. Up to 7 partitions per AZ and hundreds of instances per group. The application can know which partition each instance is in (via metadata). Suited for HDFS, HBase, Cassandra, Kafka — distributed systems that are topology-aware.

### Instance Lifecycle

**Stop/Start**: for instances with EBS as root. Data on EBS persists. The instance may be moved to different hardware on restart. The public IP changes (unless you use an Elastic IP).

**Hibernate**: saves the RAM contents to the root EBS volume (which must have enough space and be encrypted). The instance can resume where it left off, with its memory state intact. Faster than a full reboot. It cannot stay hibernated for more than 60 days.

**Terminate**: deletes the instance and, by default, the root EBS volume. Additional volumes can be configured to persist with `DeleteOnTermination=false`.

## 🏗️ Architecture and Components

```text
┌─────────────────────────────────────────────────────┐
│                  EC2 Instance                        │
│                                                      │
│  AMI (boot)                                          │
│  ┌──────────────┐    ┌────────────────────────────┐  │
│  │  EBS Root    │    │       EBS Volumes           │  │
│  │  (gp3/io2)   │    │  data, gp3, io2, st1, sc1  │  │
│  └──────────────┘    └────────────────────────────┘  │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │   Instance Store (NVMe) — temporary          │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  User Data (cloud-init)                              │
│  IMDS v2 → 169.254.169.254                           │
│  IAM Instance Profile → Role → STS temp credentials │
│                                                      │
│  Security Group (stateful)                           │
└─────────────────────────────────────────────────────┘

Placement Groups:
  Cluster:    [ i1 ][ i2 ][ i3 ]  ← 1 rack, low latency
  Spread:     [ i1 ] ... [ i2 ] ... [ i3 ]  ← distinct racks
  Partition:  P1[ i1,i2 ] P2[ i3,i4 ] P3[ i5,i6 ]  ← racks per partition
```

## ⚙️ Important Settings for the Exam

| Item | Value | Note |
|---|---|---|
| Maximum Spot discount | ~90% vs On-Demand | Interruption-tolerant workload |
| Maximum Standard RI 3-year discount | ~72% | All Upfront, same type and Region |
| gp3 baseline IOPS | 3,000 | Independent of size (unlike gp2) |
| gp3 maximum IOPS | 16,000 | With additional cost per IOPS |
| io2 Block Express maximum IOPS | 256,000 | High-performance volumes |
| io2 durability | 99.999% | Highest of EBS |
| Spread placement max per AZ | 7 instances | Per placement group |
| Maximum hibernation duration | 60 days | RAM saved to encrypted EBS |
| IMDSv2 default token TTL | 6 hours | Configurable at creation |
| Spot interruption notice | 2 minutes | Via metadata and EventBridge |

## 🔄 Service Comparison

| Purchasing Model | Discount | Commitment | Ideal for |
|---|---|---|---|
| On-Demand | — | None | Unpredictable workload, dev/test |
| Reserved (Standard) | up to 72% | 1 or 3 years | Stable production, same type |
| Reserved (Convertible) | up to 66% | 1 or 3 years | Production that may change type |
| Savings Plans (Compute) | up to 66% | 1 or 3 years (hourly $) | EC2 + Lambda + Fargate, any type |
| Savings Plans (EC2) | up to 72% | 1 or 3 years (hourly $) | EC2 same type and Region |
| Spot | up to 90% | None | Batch, CI/CD, fault-tolerant processing |
| Dedicated Host | variable | Can be RI | Per-socket license, compliance |
| Dedicated Instance | variable + fee | None | Physical isolation without host control |

| EBS Type | Use Case | Max IOPS | Max Throughput | Boot? |
|---|---|---:|---|---|
| gp3 | General purpose | 16,000 | 1,000 MBps | ✅ |
| gp2 | General (legacy) | 16,000 | 250 MBps | ✅ |
| io2 Block Express | Critical OLTP, low latency | 256,000 | 4,000 MBps | ✅ |
| io1 | Provisioned IOPS (legacy) | 64,000 | 1,000 MBps | ✅ |
| st1 | Big data, logs, sequential | 500 | 500 MBps | ❌ |
| sc1 | Cold, cheap archiving | 250 | 250 MBps | ❌ |

## 💡 Exam Tips and Pitfalls

- A T3 burstable instance under a continuous high-CPU workload will exhaust its credits and perform below expectations — Watch out for questions that describe unexpected performance degradation.
- Instance Store does not persist after stop or terminate. If the question mentions "temporary high-performance cache data", Instance Store may be the answer; if it mentions persistence, it is not.
- gp2 ties IOPS to size. If you want high IOPS without a large volume, use gp3 (size and IOPS are independent).
- Spot Fleet with the `capacityOptimized` strategy reduces the chance of interruption; `lowestPrice` maximizes savings but increases interruption risk.
- Dedicated Host is for per-socket/core licensing. Dedicated Instance is for physical isolation compliance without host visibility.
- The public IP of an EC2 instance changes on every stop/start. For a fixed IP, use an Elastic IP.
- Cluster Placement Groups stay within a single AZ. If the question asks for multi-AZ distribution, Cluster does not fit.
- IMDSv2 is more secure than IMDSv1. AWS recommends IMDSv2 only; in infrastructure security questions, prefer IMDSv2 as the answer.

## 💡 For the Exam

- Family M = balanced, C = compute, R = memory, I = storage, T = burstable. If the workload is described, map it to the family.
- Stable long-term production = Reserved or Savings Plans. Unpredictable burst = On-Demand. Interruption-tolerant = Spot.
- High IOPS and consistent latency = io2. Adequate performance and lower cost = gp3. Sequential big data = st1. Minimum cost without performance = sc1.
- HPC between nodes = Cluster PG. Critical high availability per instance = Spread PG. HDFS/Cassandra/Kafka with rack awareness = Partition PG.

## 📎 Related Links

- [Module questions](./questoes.md)
- [Module flashcards](./flashcards.md)
- [Module cheatsheet](./cheatsheet.md)
- [Module use cases](./casos-de-uso.md)
- [Module hands-on lab](./lab.md)
- [Official links](./links.md)
- [Module 04: High Availability and Scalability](../05-Alta-Disponibilidade-e-Escalabilidade/)
- [Module 05: Amazon S3 and Storage](../06-Amazon-S3-e-Armazenamento/)
