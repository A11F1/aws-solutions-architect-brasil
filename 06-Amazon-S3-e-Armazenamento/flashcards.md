# Flashcards — Module 05: Amazon S3 and Storage

---

**Q:** What are the 7 S3 storage classes in descending order of storage cost?
**A:** Standard → Intelligent-Tiering → Standard-IA → One Zone-IA → Glacier Instant Retrieval → Glacier Flexible Retrieval → Glacier Deep Archive

---

**Q:** What is the minimum storage duration charged for S3 Standard-IA?
**A:** 30 days (below that, you are billed as if it were 30 days)

---

**Q:** What is the minimum storage duration for S3 Glacier Deep Archive?
**A:** 180 days

---

**Q:** How many copies does S3 Standard keep, and across how many AZs?
**A:** At least 3 copies in at least 3 AZs of the same Region (11 9s durability = 99.999999999%)

---

**Q:** Which S3 class stores data in only 1 AZ? What is the risk?
**A:** S3 One Zone-IA. Risk: if the AZ is destroyed, the data is lost. Availability is 99.5% vs 99.9% for the other classes

---

**Q:** What is S3 Intelligent-Tiering and when should you use it?
**A:** It automatically moves objects between tiers (Frequent Access, Infrequent Access, Archive tiers) based on access. Ideal for unpredictable patterns. It charges a small monitoring fee but no retrieval fee

---

**Q:** What are the requirements to enable CRR or SRR?
**A:** Versioning must be enabled on BOTH buckets (source and destination). Existing objects are not replicated automatically (use S3 Batch Replication for that)

---

**Q:** What is the difference between SSE-S3, SSE-KMS, and SSE-C?
**A:** SSE-S3: AWS manages the keys (AES-256). SSE-KMS: keys managed via AWS KMS (audit trail, fine-grained permissions). SSE-C: the customer provides the key in each request; AWS encrypts/decrypts but does not store the key

---

**Q:** What is an S3 Bucket Key and what is its benefit?
**A:** It reduces the number of KMS calls by creating an envelope key per bucket (not per object). It reduces SSE-KMS cost by up to 99%

---

**Q:** What is the S3 throughput per prefix for PUT and GET?
**A:** 3,500 PUTs/COPY/POST/DELETE and 5,500 GET/HEAD per second **per prefix**. Spreading across multiple prefixes increases total throughput linearly

---

**Q:** When is Multipart Upload mandatory in S3?
**A:** Required for objects above **5 GB**. Recommended for objects above **100 MB**

---

**Q:** What is the difference between Object Lock Governance and Compliance?
**A:** Governance: users with the `s3:BypassGovernanceRetention` permission can delete/modify. Compliance: NO ONE can delete (not even root/AWS) during the retention period. Compliance is irrevocable

---

**Q:** What is Legal Hold in S3 Object Lock?
**A:** It protects the object from deletion regardless of the retention period. It can be applied/removed by any user with the `s3:PutObjectLegalHold` permission. It does not expire automatically

---

**Q:** What is S3 Transfer Acceleration for?
**A:** It uses CloudFront edge locations to accelerate client uploads to S3 (especially for clients geographically distant from the S3 Region). Traffic goes to the nearest CloudFront edge, then over the AWS backbone network to S3

---

**Q:** What is the difference between a Gateway Endpoint and an Interface Endpoint for S3?
**A:** Gateway Endpoint: free, added as a route in the route table, only within the VPC. Interface Endpoint (PrivateLink): has a cost ($/hour + $/GB), creates an ENI with a private IP, accessible from on-prem via DX/VPN

---

**Q:** What is Amazon FSx for Lustre and what is its main use case?
**A:** A high-performance file system based on Lustre (open-source HPC). It integrates natively with S3 (reads/writes directly). Use cases: ML training, HPC, video rendering, parallel data processing

---

**Q:** What are the 4 types of AWS Storage Gateway?
**A:** File Gateway (NFS/SMB → S3), Volume Gateway Cached (iSCSI, primary data in S3, local cache), Volume Gateway Stored (iSCSI, primary data local, S3 backup), Tape Gateway (VTL → S3/Glacier)

---

**Q:** What is S3 Select and what is its benefit?
**A:** It lets you run simple SQL queries (SELECT) on S3 objects (CSV, JSON, Parquet) without downloading the whole file. It reduces data transferred and cost. A simple alternative to Athena for one-off queries

---

**Q:** How do S3 Event Notifications work?
**A:** They trigger events to SNS, SQS, or Lambda when actions occur on the bucket (s3:ObjectCreated, s3:ObjectRemoved, s3:ObjectRestore, etc.). You can filter by key prefix and suffix. Alternative: use EventBridge (more flexible)

---

**Q:** What is the difference between EFS and EBS?
**A:** EFS: managed NFS4, multi-AZ, concurrent access from multiple instances/AZs, scales automatically, Linux only. EBS: block storage, typically attached to 1 instance (except io2 Multi-Attach), single-AZ, Windows/Linux
