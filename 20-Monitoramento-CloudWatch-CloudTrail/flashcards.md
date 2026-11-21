# Flashcards — Module 14: Monitoring, CloudWatch, and CloudTrail

> **Format:** Q = Question | A = Answer  
> **Total:** 25 flashcards

---

**Q:** How does the CloudWatch Agent let you monitor memory and disk on EC2?  
**A:** By default, CloudWatch does **not** collect RAM, disk usage, or process metrics. Installing the **CloudWatch Agent** on the instance lets you send custom metrics to CloudWatch (e.g., `mem_used_percent`, `disk_used_percent`). The Agent also collects logs and sends them to CloudWatch Logs.

---

**Q:** What is CloudWatch Logs Insights?  
**A:** An **interactive query engine** for CloudWatch Logs. Own syntax (similar to SQL). Lets you filter, aggregate, and visualize logs. E.g.: `fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 20`. Pay per data scanned. Visual results (tables, time series charts).

---

**Q:** What is CloudTrail and which events does it record?  
**A:** **API call auditing** service on AWS. Records: who called which API, when, from which IP, with what result. Types: **Management Events** (create/delete resources — CloudFormation, IAM, EC2 start/stop), **Data Events** (operations on data — S3 GetObject, Lambda Invoke; disabled by default), **Insights Events** (unusual API activity).

---

**Q:** What is the difference between CloudTrail and AWS Config?  
**A:** **CloudTrail:** "who did what and when?" — API call history. Action auditing. **AWS Config:** "how is it configured now and how did it change?" — resource configuration history. Compliance: evaluate resources against rules. CloudTrail records events; Config records configuration states.

---

**Q:** What are AWS Config Rules and how do they work?  
**A:** Rules that evaluate whether AWS resources comply with policies. **Managed Rules:** prebuilt by AWS (e.g., `s3-bucket-public-read-prohibited`). **Custom Rules:** Lambda code that evaluates compliance. Evaluation: triggered by changes or periodic. Status: COMPLIANT / NON_COMPLIANT. Automatic remediation via SSM.

---

**Q:** What is AWS X-Ray and which concepts are central?  
**A:** **Distributed tracing** service for applications. Concepts: **Trace** = end-to-end request; **Segment** = work done by a service; **Subsegment** = granular sub-work; **Annotations** = key-values for filtering traces; **Sampling** = % of requests to trace (default: 5%). Service Map: visualization of dependencies and latencies.

---

**Q:** What is CloudWatch Anomaly Detection?  
**A:** Uses ML to create a model of expected behavior for a metric (based on history). Creates a dynamic band of "normal". Alerts when the metric leaves the band. Adapts to seasonality (days of the week, hours). More sophisticated than a static threshold alarm.

---

**Q:** What is AWS Systems Manager Session Manager and why is it more secure than SSH?  
**A:** Access to EC2 instances (and managed on-premises servers) via browser or CLI **without opening port 22 (SSH) and without SSH keys**. Auditable: all sessions are logged to CloudWatch/S3. Works on instances in private subnets without a bastion host. Requires the SSM Agent on the instance + an IAM Role with `ssm:StartSession` permission.

---

**Q:** What are the two tiers of SSM Parameter Store?  
**A:** **Standard:** free, up to 10,000 parameters, value up to 4 KB, no Advanced Policies, no TTL. **Advanced:** paid ($0.05/month per parameter), up to 100,000 parameters, value up to 8 KB, supports Parameter Policies (automatic TTL, expiration notifications). For rotating secrets: use Secrets Manager.

---

**Q:** What is CloudWatch Contributor Insights?  
**A:** Analyzes logs and metrics to identify the **top contributors** to latency, errors, or cost. E.g.: "which 10 IPs generate the most 5XX errors?" or "which customers have the highest API latency?". Creates rules with log patterns. Useful for: security (detecting suspicious IPs), performance (identifying users/resources that degrade the system).

---

**Q:** How do you make CloudTrail logs tamper-proof?  
**A:** Enable **Log File Validation** on the trail: CloudTrail generates a Digest file (SHA-256 hash) every hour. Any modification of the log file is detectable by comparing with the digest. Store logs in an S3 bucket with **Object Lock (Compliance mode)** + MFA Delete + restrictive policies. CloudTrail Lake for immutable analysis.

---

**Q:** What is AWS Config Aggregator?  
**A:** Collects AWS Config compliance data from multiple accounts and regions into a **central account**. Two source types: individual accounts (list of account IDs) or Organization (all accounts in the AWS Org automatically). Enables centralized multi-account compliance visibility. The central account needs permission from the sources via a delegated administrator.

---

**Q:** What is SSM Patch Manager?  
**A:** Automates patching of EC2 instances (and on-premises). **Patch Baselines:** define which patches to install (severity, classification). **Maintenance Windows:** time windows to apply patches. **Compliance Reporting:** reports in Config which instance is up to date. Can use Run Command to apply patches on demand.

---

**Q:** What is the default log retention in CloudWatch Logs?  
**A:** By default: **indefinite** (logs never expire). Configurable per Log Group: 1 day to 10 years, or "Never Expire". To reduce cost: configure a retention policy. For compliance: configure appropriate retention. For long term: export to S3 (CloudWatch Logs Subscription Filters → Kinesis Firehose → S3).

---

**Q:** What is AWS Trusted Advisor?  
**A:** Advisor that analyzes the AWS account in 5 categories: **Cost Optimization** (idle instances, unused Reserved), **Performance** (EBS throughput, EC2 type), **Security** (open SGs, root MFA, public S3), **Fault Tolerance** (RDS backups, ELB health checks), **Service Limits** (alerts when approaching limits). Basic checks free; full set with Business/Enterprise Support.

---

**Q:** How does a CloudWatch Alarm monitor the state of an EC2 instance and trigger recovery?  
**A:** Alarm on the `StatusCheckFailed_System` metric (failure of the underlying hardware). Alarm action: **EC2 Instance Recovery** — moves the instance to healthy hardware while keeping the same private IP, Elastic IP, metadata, and EBS volumes. (vs. Reboot, which only restarts on the same hardware).

---

**Q:** What is a CloudWatch Synthetics Canary?  
**A:** Scripts (Node.js or Python) that emulate user behavior on web applications/APIs at configurable intervals. Monitors: availability, latency, user flows (login → checkout). Detects problems before real users are affected. Metrics sent to CloudWatch. Different from Real User Monitoring.

---

**Q:** What is EventBridge (formerly CloudWatch Events)?  
**A:** Serverless event bus. Connects AWS services and SaaS. Sources: EC2 state changes, CodePipeline stages, Scheduled events (cron), S3 events, custom events. Targets: Lambda, SQS, SNS, Step Functions, Kinesis, ECS, API Gateway. Rules with event pattern matching (JSON filters). EventBridge Pipes for point-to-point transformation.

---

**Q:** What is a CloudWatch Logs Subscription Filter?  
**A:** In real time, filters and delivers logs from a Log Group to: **Kinesis Data Streams**, **Kinesis Firehose**, **Lambda**. Use cases: real-time log analysis (KDS → Lambda), long-term storage (KDF → S3), multi-account aggregation (cross-account subscription destinations).

---

**Q:** How does AWS Config automatically detect non-compliant resources and remediate them?  
**A:** **Detection:** Config Rule evaluated continuously or on change. NON_COMPLIANT status is alerted in the console and via EventBridge. **Automatic remediation:** SSM Automation Document executed automatically when the rule detects non-compliance. E.g.: public S3 bucket → remediation runs SSM that applies Block Public Access.

---

**Q:** What is CloudTrail Insights?  
**A:** Detects anomalous API call activity. Analyzes the normal pattern of write management events and alerts when there is a significant deviation (e.g., an unusual spike of `TerminateInstances`, `DeleteSecurityGroup`). Use cases: detect compromised credentials, automation errors, attacks. Insights events are billed separately.

---

**Q:** What is the difference between CloudWatch Metrics and CloudWatch Custom Metrics?  
**A:** **Built-in Metrics:** collected automatically by AWS services (EC2 CPU, ELB latency, Lambda InvocationDuration). Default granularity: 5 minutes (1 min with "detailed monitoring"). **Custom Metrics:** sent via the PutMetricData API or CloudWatch Agent from custom applications. Granularity: 1 second (High Resolution). Examples: business metrics, application memory.

---

**Q:** What is AWS Security Hub?  
**A:** Centralizer of **security findings** from multiple services (GuardDuty, Inspector, Macie, IAM Analyzer, Config, Firewall Manager, 3rd party). Unified dashboard. Security posture scoring against standards (CIS AWS Benchmarks, AWS Foundational Security Best Practices). Automates remediations via EventBridge. Multi-account via Organization.

---

**Q:** What is SSM Run Command?  
**A:** Runs scripts (Shell, PowerShell, etc.) on EC2 instances and managed on-premises servers at scale. No SSH required. Targets: instances by tag, by instance ID, all managed instances. Auditable: output sent to S3/CloudWatch. Rate control to apply gradually. Foundation of Patch Manager and other SSM automation.

---

**Q:** What is the difference between CloudWatch Logs and CloudTrail for security auditing?  
**A:** **CloudTrail:** auditing of actions in AWS (API calls, console logins, IAM changes). Answers: "was it deleted/modified? By whom? When?". **CloudWatch Logs:** logs from applications, OS, AWS services (VPC Flow Logs, ALB access logs). Answers: "what happened inside the application?". For a complete audit: CloudTrail (who did what) + CloudWatch Logs (what happened in the system).
