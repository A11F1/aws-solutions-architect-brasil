# 22 Disaster Recovery and Continuity

## Module objectives

- master RTO and RPO and map each DR strategy to the business requirement
- differentiate Backup and Restore, Pilot Light, Warm Standby, and Multi-Site Active/Active
- select AWS services for continuity: AWS Backup, Elastic Disaster Recovery, Route 53, Aurora Global Database, DynamoDB Global Tables, and S3 CRR
- answer exam questions balancing resilience, cost, and operational complexity

## Core concepts

Disaster recovery in the SAA-C03 is not just backup. The exam tests architectural design that keeps operating when there is an AZ, region, managed service, or human error failure.

- RTO (Recovery Time Objective): maximum acceptable time to restore the service.
- RPO (Recovery Point Objective): maximum acceptable data loss measured in time.

The lower the RTO and RPO, the higher the cost and complexity tend to be.

## DR strategies

1. Backup and Restore
- lowest cost
- slowest restoration
- high RTO, moderate/high RPO
- ideal for non-critical workloads

2. Pilot Light
- minimal critical components stay active in the secondary region
- application and capacity scale only during the disaster
- medium RTO, low/medium RPO

3. Warm Standby
- scaled-down environment always active in another region
- fast scale-up during the disaster
- low RTO, low RPO

4. Multi-Site Active/Active
- two active regions serving traffic
- near-immediate failover
- very low RTO and RPO
- higher cost and more complex operation

## Most frequently tested services and patterns

- Route 53 Failover + Health Checks for redirection between regions.
- AWS Elastic Disaster Recovery for continuous server replication and fast cutover.
- AWS Backup for central backup policy and cross-account vaults.
- S3 Versioning + CRR for objects and regional resilience.
- RDS/Aurora: snapshots, cross-region read replicas, and Aurora Global Database.
- DynamoDB: PITR and Global Tables for low latency and continuity.
- Cross-region EBS snapshots for EC2 restoration.

## Exam tips

- Multi-AZ does not solve a regional disaster; for that, the answer needs multi-region.
- If the question highlights "lowest cost", Backup and Restore or Pilot Light tends to win.
- If it asks for "minimum downtime", Warm Standby or Active/Active is usually better.
- If it says "least operational effort", prefer managed services and native automation.
- If it asks to preserve DNS during failover, think Route 53 with health checks.

## Related links

- [Cheatsheet](./cheatsheet.md)
- [Use cases](./casos-de-uso.md)
- [Questions](./questoes.md)
- [Flashcards](./flashcards.md)
- [Lab](./lab.md)
- [Official links](./links.md)