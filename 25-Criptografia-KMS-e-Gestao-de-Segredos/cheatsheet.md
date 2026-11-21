# Cheatsheet - Encryption, KMS, and Secrets Management

## Quick map

| Need | Best choice | Note |
|---|---|---|
| Key with policy control and auditing | KMS customer-managed key | More control for compliance |
| Automatic secret rotation | Secrets Manager | Strong integration with RDS |
| Low-cost app parameters | Parameter Store | SecureString uses KMS |
| TLS certificate for ALB/CloudFront/API Gateway | ACM | Free public cert |
| Cryptographic control on a dedicated HSM | CloudHSM | Higher operational complexity |

## Exam rules

- key policy is essential for permissions in KMS
- explicit deny always prevails
- use an IAM role for apps instead of static keys
- CloudFront + public ACM => certificate in us-east-1

## Pitfalls

- confusing Secrets Manager with Parameter Store in an automatic-rotation scenario
- forgetting the kms:Decrypt permission on the consuming principal
- assuming that encrypting in the service removes the need for access management
- ignoring the cost of KMS calls at high throughput
