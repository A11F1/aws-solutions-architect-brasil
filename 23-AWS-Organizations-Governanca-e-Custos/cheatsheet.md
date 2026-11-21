# Cheatsheet - AWS Organizations, Governance, and Costs

## Quick map

| Need | Service/Pattern | Exam tip |
|---|---|---|
| Permission guardrail across several accounts | SCP | Remember: it does not grant, it only restricts |
| Provision a standard landing zone | Control Tower | Lowest overhead for multi-account |
| Centralized user access | IAM Identity Center | Avoids IAM users in every account |
| Alert on budget overrun | AWS Budgets | Alerts before the bill closes |
| Detailed spend analysis | CUR + Athena | Granular view by service/tag |
| Reduce predictable compute cost | Savings Plans / RI | Usage commitment lowers the price |

## Cost optimization

- rightsizing of EC2 and RDS
- shutting down non-production environments
- appropriate storage classes (S3 lifecycle)
- use Spot for stateless workloads
- review transfers between AZs/regions

## Trigger phrases

- "across multiple AWS accounts" => Organizations/Control Tower
- "prevent specific actions" => SCP
- "cost visibility by team" => Cost Allocation Tags + CUR
- "predictable usage" => RI/Savings Plans
- "variable workload" => Savings Plans or on-demand + autoscaling
