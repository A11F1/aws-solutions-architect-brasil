# Flashcards — Module 15: Migration and Transfer

> **Format:** Q = Question | A = Answer  
> **Total:** 25 flashcards

---

**Q:** What are the 7 Rs of cloud migration strategy?  
**A:** **Retire** (decommission), **Retain** (keep on-premises), **Rehost** (lift-and-shift, minimal change), **Relocate** (move VMware to VMware Cloud on AWS), **Replatform** (targeted optimization without refactoring — e.g., RDS instead of self-managed MySQL), **Repurchase** (switch to SaaS), **Refactor/Rearchitect** (redesign for cloud-native).

---

**Q:** What is AWS MGN (Application Migration Service) and how does it work?  
**A:** A **rehosting** (lift-and-shift) service to EC2. An agent installed on the source server performs **continuous replication** (block by block) to AWS. At cutover: the source server stops, and AWS launches an EC2 instance from the most recent snapshot. Minimizes downtime (~minutes). Replaces SMS (Server Migration Service).

---

**Q:** What is the capacity of Snowmobile?  
**A:** **100 PB** per trip (physical truck with a storage container). For migrating entire datacenters (100 PB+). Faster than internet transfer when the volume is > 10 PB (general rule: Snowball when > 10 TB, Snowmobile when > 10 PB). 

---

**Q:** What are the differences between the Snow Family variants?  
**A:** **Snowcone Small (8 TB SSD) / Large (14 TB SSD):** portable, remote locations, edge computing, 2 vCPUs. **Snowball Edge Storage Optimized (80 TB HDD):** large-scale migration, 40 vCPUs, transfer to S3. **Snowball Edge Compute Optimized (42 TB HDD):** heavy edge computing, 52 vCPUs, optional GPU. **Snowmobile:** 100 PB, entire datacenters.

---

**Q:** What is AWS DataSync and which sources does it support?  
**A:** An **online** data transfer service (over the network). Sources: NFS, SMB (on-premises or other clouds), HDFS, on-premises S3, self-managed object storage. Destinations: S3 (any class), EFS, FSx for Windows/Lustre/ONTAP. Features: **preserves metadata** (permissions, timestamps), integrity verification, scheduled or on-demand transfer.

---

**Q:** What is the difference between AWS DataSync and AWS Storage Gateway?  
**A:** **DataSync:** batch data migration/transfer (one-time or scheduled). Moves files from A to B. **Storage Gateway:** **continuous hybrid** integration — an extension of on-premises storage to AWS. File GW: NFS/SMB → S3. Volume GW: iSCSI → EBS/S3. Tape GW: emulates a tape library → S3/Glacier. DataSync = move data; Storage Gateway = permanent extension.

---

**Q:** What is AWS DMS and when is it also necessary to use SCT?  
**A:** **DMS (Database Migration Service):** migrates data with CDC (Change Data Capture) for minimal downtime. **SCT (Schema Conversion Tool):** converts schema and PL/SQL code between incompatible engines. **Rule:** **homogeneous** migration (Oracle → Oracle RDS): DMS only. **Heterogeneous** migration (Oracle → Aurora PostgreSQL): SCT first (converts schema) + DMS (migrates data).

---

**Q:** What is continuous replication in AWS DMS (CDC)?  
**A:** **CDC (Change Data Capture):** after the initial full load, DMS captures and applies changes from the source database in real time (inserts, updates, deletes). Keeps the databases in sync during the migration. Allows cutover with a minimal maintenance window — the source can keep receiving transactions until cutover.

---

**Q:** What is AWS Migration Hub?  
**A:** A centralized dashboard to track the migration progress of multiple applications and servers. Integrates with MGN, DMS, CloudEndure. Categorizes discovered servers and tracks status (Discovered → Not Started → In Progress → Migrated). Does not perform migrations — it is only tracking and multi-tool visibility.

---

**Q:** What is AWS Application Discovery Service?  
**A:** Discovers and assesses on-premises workloads for migration planning. **Agentless Discovery (VMware only):** via VMware vCenter, collects VM specs (CPU, memory, storage) without installing agents. **Agent-based Discovery:** installs an agent on each server, collects: CPU, memory, processes, network connections (dependency mapping). Data is sent to Migration Hub.

---

**Q:** What is AWS Transfer Family?  
**A:** A managed service for secure file transfer using legacy protocols: **SFTP, FTPS, FTP, and AS2**. Backend storage: S3 or EFS. Use cases: B2B file exchange, partners that need SFTP, migrating legacy FTP servers to AWS. Keeps existing clients without changing protocol.

---

**Q:** How do I calculate whether to use Snowball or network transfer?  
**A:** Rule of thumb: **Snowball** when the internet transfer would take > 1 week. Formula: `time = data_size / available_bandwidth`. E.g.: 100 TB / 100 Mbps = ~100 days. Use Snowball. 1 TB / 1 Gbps = ~2.2 hours. Use the internet. Also consider: bandwidth cost, security of data in transit.

---

**Q:** What is the benefit of Direct Connect for migration instead of VPN?  
**A:** **Direct Connect:** dedicated private connection between the data center and AWS via a partner (10 Gbps or 100 Gbps). Consistent latency, dedicated bandwidth, more expensive, takes weeks to provision. **VPN:** encrypted over the public internet, latency variation, cheaper, provisioned in hours. For migrating large volumes: Direct Connect is more efficient (constant bandwidth).

---

**Q:** What is AWS Snowball Edge with compute and when should I use it?  
**A:** **Snowball Edge Compute Optimized:** 52 vCPUs, optional GPU (NVIDIA Tesla), 42 TB HDD. Use when: you need to process data **at the edge** (factories, ships, locations without stable connectivity) before sending — pre-processing, compression, ML inference in the field. Runs EC2 instances (sbe-c, sbe-m family) and Lambda functions.

---

**Q:** What is an AWS DataSync Location?  
**A:** An endpoint configured in DataSync representing either the source or the destination of a transfer. Types: NFS Server, SMB Share, Hadoop HDFS, Amazon S3, Amazon EFS, Amazon FSx, object storage. A **Task** connects two Locations (source → destination) and defines the transfer settings (schedule, filter, verification).

---

**Q:** What is the "Discovery" phase in the AWS migration methodology?  
**A:** Initial phase: inventory and assess on-premises workloads. Tools: Application Discovery Service, Migration Evaluator (TCO analysis — calculates how much it costs to keep on-premises vs migrate to AWS). Output: list of servers/apps with specs, mapped dependencies, prioritization by complexity and impact.

---

**Q:** How long does it take to provision a Snowball after requesting it?  
**A:** Typically **5-7 business days** for delivery after the order. Process: request in the console → AWS ships the Snowball → the company loads data → returns it to AWS → AWS imports into S3 (1-3 days after receipt). Total: typically 2-3 weeks. For urgent cases: multiple Snowballs in parallel.

---

**Q:** What is VMware Cloud on AWS and when should I use Relocate?  
**A:** **VMware Cloud on AWS:** runs the VMware SDDC (Software-Defined Data Center) on dedicated AWS hardware. For migrating VMware VMs without refactoring. You manage it via vCenter as on-premises. **Relocate:** migrate VMware VMs to VMware Cloud on AWS without conversion — near zero-touch migration, keeping the same VMware stack.

---

**Q:** How does DMS support Oracle to Amazon Aurora PostgreSQL?  
**A:** 2-step process: **(1) SCT:** converts the Oracle schema (tables, constraints, views, PL/SQL procedures → PL/pgSQL). The conversion report indicates manual complexity. **(2) DMS:** initial full-load of the data + ongoing CDC. DMS supports Oracle as a source with LogMiner (for CDC), capturing redo logs.

---

**Q:** What is AWS DataSync metadata preservation and why does it matter?  
**A:** DataSync preserves: **POSIX permissions** (owner, group, permissions), **timestamps** (creation, modification, access), **ACLs** (for SMB/NFS). Why it matters: migrated files keep their original permissions — applications that depend on file permissions keep working without manual reconfiguration. S3 Object Metadata is also preserved.

---

**Q:** What are the Snowball Edge "job types"?  
**A:** **Import to S3:** transfer data from on-premises to S3 (most common use). **Export from S3:** extract data from S3 to on-premises (data leaves AWS). **Local Compute and Storage:** use Snowball at the edge for processing without sending data — runs EC2/Lambda locally. Each job type has a different shipping process.

---

**Q:** What is AWS Migration Evaluator?  
**A:** A free TCO (Total Cost of Ownership) analysis tool for on-premises vs AWS. Installs agents or uses imported utilization data. Generates a report: a "business case" showing estimated cost on AWS vs current on-premises cost. The basis for executive approval of a migration.

---

**Q:** What is Server Migration Service (SMS) and what is its replacement?  
**A:** SMS: a legacy service for replicating on-premises VMs to AMIs on AWS. **Replaced by AWS MGN (Application Migration Service)** — more modern, continuous incremental replication instead of periodic snapshots, lower downtime. SMS was discontinued for new users. MGN is the recommended service.

---

**Q:** How does Snowball protect the physical data?  
**A:** Physical and logical protection: automatic AES-256 encryption, **Trusted Platform Module (TPM)** for hardware integrity verification, tamper-evident case, data cannot be accessed without the KMS keys associated with the job. After importing to S3, the data is erased from the Snowball per NIST 800-88.

---

**Q:** What is the AWS DataSync Task Schedule vs Task Execution?  
**A:** **Task:** persistent transfer configuration (source, destination, options). **Task Execution:** a single run of the task. **Schedule:** automatic periodic execution (cron/rate). A Task can have multiple Executions. Each execution can be: LAUNCHING → PREPARING → TRANSFERRING → VERIFYING → SUCCESS/ERROR.
