# Flashcards - Disaster Recovery and Continuity

1. What does RTO stand for?
- Maximum time to restore the service.

2. What does RPO stand for?
- Maximum window of data loss.

3. Does Multi-AZ cover a regional disaster?
- No.

4. Lowest-cost DR?
- Backup and Restore.

5. Strategy with a minimal environment active in a second region?
- Pilot Light.

6. Strategy with a scaled-down environment always active?
- Warm Standby.

7. Generally the lowest RTO/RPO?
- Active/Active multi-region.

8. AWS server DR service?
- AWS Elastic Disaster Recovery.

9. Protection of objects against accidental deletion?
- S3 Versioning.

10. Replicating objects between regions?
- S3 CRR.

11. Managed DNS failover?
- Route 53 with health checks.

12. RDS Multi-AZ is for?
- Regional high availability, not multi-region DR.

13. DynamoDB for global DR?
- Global Tables.

14. What to test periodically in DR?
- Failover and failback runbook.

15. Exam phrase: "regional disaster"?
- Requires a multi-region design.