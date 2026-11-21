# Exam Questions — Module 05: Amazon S3 and Storage

<!-- SAA-C03 Domain: Design Resilient Architectures / Cost-Optimized Architectures -->

---

**1.** A company stores access logs with an unpredictable access pattern for an indeterminate period. They need the lowest possible cost without managing tiers manually. Which S3 class is most suitable?

- A) S3 Standard-IA
- B) S3 Glacier Flexible Retrieval
- C) S3 Intelligent-Tiering
- D) S3 One Zone-IA

<details>
<summary>Answer</summary>
**C — S3 Intelligent-Tiering**
It automatically moves objects between tiers (Frequent, Infrequent, Archive Instant, Archive) based on access pattern. Ideal when the pattern is unpredictable. One Zone-IA would be cheaper but only if the data can tolerate loss; Standard-IA incurs retrieval costs under frequent access.
</details>

---

**2.** An S3 bucket needs to be publicly accessible to host a static website. The administrator enabled "Block Public Access" on the account but wants to open up only this bucket. What should they do?

- A) It is not possible — account-level Block Public Access is irrevocable
- B) Disable Block Public Access on the account, then apply a public bucket policy
- C) Enable "requester pays" on the bucket to allow public access
- D) Create an S3 endpoint and access it via CloudFront without making the bucket public

<details>
<summary>Answer</summary>
**D — Use CloudFront with OAC (recommended)**
The best practice is NOT to make the bucket public. Use CloudFront with Origin Access Control (OAC). But if the question requires a public bucket: the correct answer is B — disable Block Public Access on the account and apply a bucket policy. Option D is architecturally superior and is what AWS recommends.
</details>

---

**3.** A company wants to replicate S3 objects from us-east-1 to ap-southeast-1 to reduce latency for users in Asia. Which feature should be enabled?

- A) S3 Transfer Acceleration
- B) Cross-Region Replication (CRR)
- C) Same-Region Replication (SRR)
- D) S3 Multi-Region Access Points

<details>
<summary>Answer</summary>
**B — Cross-Region Replication (CRR)**
CRR asynchronously replicates objects between buckets in different Regions. It requires versioning enabled on both. SRR is for the same Region. Transfer Acceleration speeds up client uploads to S3; it does not replicate objects. Multi-Region Access Points route to the closest bucket but do not replicate on their own.
</details>

---

**4.** Which type of S3 encryption lets the customer manage the keys while AWS performs the encryption/decryption?

- A) SSE-S3 (AES-256)
- B) SSE-KMS
- C) SSE-C
- D) Client-Side Encryption (CSE)

<details>
<summary>Answer</summary>
**C — SSE-C**
SSE-C: the customer provides the key in each HTTP request; AWS encrypts/decrypts but never stores the key. SSE-S3: AWS manages everything. SSE-KMS: AWS manages via KMS (keys can be customer-managed CMKs). CSE: the customer encrypts before uploading, and AWS only sees encrypted bytes.
</details>

---

**5.** An S3 object needs to be accessed by an external user without AWS credentials for 1 hour. Which mechanism should be used?

- A) Public ACL on the object
- B) Pre-signed URL
- C) Bucket policy with condition aws:SourceIp
- D) VPC Endpoint with an open policy

<details>
<summary>Answer</summary>
**B — Pre-signed URL**
Pre-signed URLs grant temporary access (configurable duration) to a specific object, using the credentials of whoever generated the URL. They expire after the TTL. A public ACL would expose the object indefinitely. SourceIp restricts by IP, not by time.
</details>

---

**6.** A company has S3 objects that cannot be deleted for 7 years due to financial regulation. Which feature will guarantee that not even administrators can delete them?

- A) MFA Delete
- B) Versioning with a lifecycle policy
- C) S3 Object Lock in Compliance mode
- D) S3 Object Lock in Governance mode

<details>
<summary>Answer</summary>
**C — S3 Object Lock in Compliance mode**
In Compliance mode, NO user (including root) can delete or alter the object before the retention period expires. In Governance mode, users with the `s3:BypassGovernanceRetention` permission can bypass it. MFA Delete protects against deletion of the bucket's versions but does not guarantee a minimum retention.
</details>

---

**7.** An application uploads 1 TB files to S3. What is needed to ensure upload reliability and efficiency?

- A) Use S3 Transfer Acceleration exclusively
- B) Use Multipart Upload (required above 5 GB)
- C) Use S3 Batch Operations
- D) Upload to a regional bucket and then replicate with CRR

<details>
<summary>Answer</summary>
**B — Multipart Upload**
Multipart Upload is **required** for objects above **5 GB** and recommended for objects above 100 MB. It splits the file into parallel parts, increasing speed and resilience (failed parts are re-sent individually). Transfer Acceleration reduces geographic latency, not upload reliability.
</details>

---

**8.** A web client needs to upload files directly from the browser to an S3 bucket on a different domain. What should be enabled on the bucket?

- A) Transfer Acceleration
- B) CORS (Cross-Origin Resource Sharing)
- C) Bucket Policy with public s3:GetObject
- D) ACL with AllUsers permission

<details>
<summary>Answer</summary>
**B — CORS**
CORS controls which origins (external domains) can make requests to the S3 bucket directly from the browser. Without CORS configured, the browser will block cross-origin requests for security reasons. This is required for direct browser-to-S3 uploads.
</details>

---

**9.** A company wants to minimize costs by storing audit logs that will rarely be accessed but, when needed, must be available in less than 5 minutes. Which S3 class?

- A) S3 Glacier Deep Archive
- B) S3 Glacier Flexible Retrieval
- C) S3 Glacier Instant Retrieval
- D) S3 Standard-IA

<details>
<summary>Answer</summary>
**C — S3 Glacier Instant Retrieval**
Glacier Instant Retrieval offers retrieval in **milliseconds**, at a lower cost than Standard-IA. Ideal for rarely accessed data that needs immediate access when requested. Glacier Flexible Retrieval takes minutes to hours. Deep Archive takes hours.
</details>

---

**10.** A file system must be shared simultaneously among 100 Linux EC2 instances across multiple AZs. Which AWS service should be used?

- A) Amazon S3
- B) Amazon EBS (gp3)
- C) Amazon EFS
- D) Amazon FSx for Windows File Server

<details>
<summary>Answer</summary>
**C — Amazon EFS**
EFS is managed NFS for Linux, multi-AZ, and allows concurrent access from multiple instances/AZs. EBS is block storage attached to a single instance (except io2 Multi-Attach, limited to 16 instances in the same AZ). FSx for Windows is for Windows environments. S3 has different semantics (object storage).
</details>

---

**11.** A company's data center needs to migrate 10 TB of weekly backup data to AWS S3. The internet connection is limited. Which service should be used without the AWS Snow Family?

- A) AWS DataSync over the internet
- B) AWS Storage Gateway — Tape Gateway
- C) AWS DataSync over Direct Connect
- D) S3 Transfer Acceleration

<details>
<summary>Answer</summary>
**C — AWS DataSync over Direct Connect**
DataSync is the data migration/synchronization service. With Direct Connect, data travels over a private dedicated link, avoiding the internet. Storage Gateway Tape replaces physical tapes but does not optimize bulk transfer. Transfer Acceleration uses the internet (CloudFront edge).
</details>

---

**12.** A company needs an on-premises server to access files stored in S3 using the NFS/SMB protocol with a local cache for low latency. Which service?

- A) AWS DataSync
- B) Amazon EFS with VPN
- C) AWS Storage Gateway — File Gateway
- D) AWS Storage Gateway — Volume Gateway (Stored mode)

<details>
<summary>Answer</summary>
**C — File Gateway**
File Gateway exposes S3 buckets as NFS/SMB file shares to on-premises applications. A local cache keeps recent files for low latency. Data is stored in S3 as objects. DataSync is for batch migration, not continuous access. Volume Gateway works with block storage (iSCSI).
</details>

---

**13.** Which statement about S3 Lifecycle Policies is TRUE?

- A) Objects can transition from S3 Standard directly to S3 Glacier Deep Archive without going through other tiers
- B) Objects can only transition to cheaper tiers (it is not possible to go back to Standard)
- C) To transition to Standard-IA or One Zone-IA, objects must be at least 180 days old
- D) S3 Intelligent-Tiering cannot be the destination of a Lifecycle Policy

<details>
<summary>Answer</summary>
**A — Correct**
Lifecycle Policies allow direct transitions between any classes (e.g., Standard → Deep Archive directly). B is also true (lifecycle only transitions to cheaper tiers; restoring is manual). C is wrong — the minimum for IA tiers is **30 days**. D is wrong — Intelligent-Tiering can be a destination.
</details>

---

**14.** An S3 bucket receives 100,000 PUTs per second distributed across 10 different prefixes. What is the expected throughput per prefix?

- A) 100 PUT/s per prefix
- B) 3,500 PUT/s per prefix
- C) 5,500 GET/s per prefix
- D) 10,000 PUT/s per prefix (auto-scaling)

<details>
<summary>Answer</summary>
**B — 3,500 PUT/s per prefix**
S3 supports **3,500 PUT/COPY/POST/DELETE** and **5,500 GET/HEAD** requests per second **per prefix**. By distributing the load across 10 different prefixes, you get 35,000 PUTs/s in total. This is linear — more prefixes = more throughput.
</details>

---

**15.** A company wants to receive an immediate notification when a specific file is uploaded to S3. Which configuration should be used?

- A) S3 Inventory with a daily report
- B) CloudTrail with an S3 event filter
- C) S3 Event Notifications → SNS or Lambda
- D) EventBridge Scheduler polling the bucket

<details>
<summary>Answer</summary>
**C — S3 Event Notifications**
S3 Event Notifications trigger real-time events to SNS, SQS, or Lambda when objects are created, deleted, restored, etc. It is the most direct, lowest-latency approach. CloudTrail records API calls but is not meant for "immediate notification". EventBridge can also be used (S3 integrates with EventBridge), but the most direct approach is S3 Event Notifications → Lambda.
</details>
