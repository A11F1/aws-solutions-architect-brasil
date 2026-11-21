# Use Cases - AWS Organizations, Governance, and Costs

## Case 1: Fast-growing company

Scenario:
- 40 AWS accounts in 6 months
- independent teams per product
- need for common guardrails

Architecture:
- AWS Control Tower for the landing zone
- OUs per environment (Sandbox, Dev, Prod)
- SCP blocking CloudTrail deactivation and use outside allowed regions
- Identity Center for federated access

## Case 2: FinOps with chargeback

Scenario:
- leadership wants cost per product and per squad

Architecture:
- Mandatory tags (Owner, CostCenter, Product, Environment)
- CUR in S3
- Athena + QuickSight for chargeback dashboards
- Budgets per account and per tag

## Case 3: Compute cost reduction

Scenario:
- high spend on EC2 and Fargate

Architecture:
- Savings Plans for the stable baseline
- Spot for batch and stateless workers
- autoscaling with limits
- rightsizing with Compute Optimizer

## Case 4: Security governance

Scenario:
- audit requires immutable trails

Architecture:
- central log account
- CloudTrail org trail
- log bucket with Object Lock
- SCP blocks to prevent trail modification
