# Lab - Basic Governance and Cost Control

## Objective
Set up an initial governance structure and financial monitoring.

## Steps

1. Create an Organization (if one does not exist yet).
2. Create OUs: Sandbox, Dev, and Prod.
3. Move accounts to the correct OUs.
4. Create a regional restriction SCP (example: only us-east-1 and sa-east-1).
5. Enable Cost Allocation Tags (Environment, CostCenter, Owner).
6. Create a monthly AWS Budget per account with an email/SNS alert.
7. Open Cost Explorer and validate data by service and tag.

## Validation

- Accounts in the correct OUs
- SCP applied and validated
- Budget alerts working
- Costs visible by tag in the explorer

## Cleanup

- remove test budgets
- detach the lab SCP
