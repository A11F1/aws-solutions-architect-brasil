# 23 AWS Organizations, Governance, and Costs

## Module objectives

- design multi-account environments with security and central governance
- apply SCPs, tagging, and access control per Organizational Unit
- master cost services in an architecture context: Budgets, Cost Explorer, CUR, Savings Plans, and RI
- answer SAA-C03 scenarios that call for cost reduction with the lowest operational overhead

## Core concepts

In the SAA-C03, governance and cost appear alongside security and operations. The central idea is to structure accounts to reduce risk, improve financial visibility, and apply guardrails without blocking the team.

## Recommended multi-account architecture

- Management account only for governance and billing
- OUs separated by environment and criticality (Sandbox, Dev, Prod, Security)
- Dedicated accounts for logs and security
- Central human access via IAM Identity Center

## Key services and practices

- AWS Organizations: account hierarchy and consolidated billing
- SCP: defines the permission ceiling per OU/account
- AWS Control Tower: multi-account baseline with guardrails
- AWS Budgets: proactive alerts by spend or usage threshold
- Cost Explorer: trend analysis and recommendations
- Cost and Usage Report (CUR): detailed basis for chargeback/showback
- Savings Plans and Reserved Instances: commitment to reduce compute cost
- Cost Allocation Tags: track cost by team/product/environment

## Exam tips

- SCPs do not grant permissions; they only restrict.
- "Least operational overhead" in multi-account points to Control Tower and Identity Center.
- For granular cost visibility at scale, CUR + Athena is a recurring alternative.
- For stable workloads, RI/Savings Plans is usually the optimization answer.
- Spot is best for interruption-tolerant workloads.

## Anti-patterns

- using a single account for everything
- using the root user day to day
- no standard of mandatory tags
- no cost limits/alerts per environment
- no segregation of log and security accounts

## Related links

- [Cheatsheet](./cheatsheet.md)
- [Use cases](./casos-de-uso.md)
- [Questions](./questoes.md)
- [Flashcards](./flashcards.md)
- [Lab](./lab.md)
- [Official links](./links.md)
