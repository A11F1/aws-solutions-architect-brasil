# Use Cases - Encryption, KMS, and Secrets Management

## Case 1: Relational database with password rotation

Scenario:
- ECS application accesses Aurora
- audit requires automatic rotation

Architecture:
- Secrets Manager with rotation
- ECS task role to read the secret
- KMS customer-managed key to encrypt the secret

## Case 2: Public API with managed TLS

Scenario:
- API Gateway + CloudFront
- no manual certificate management

Architecture:
- Public ACM with automatic renewal
- certificate in us-east-1 for CloudFront

## Case 3: Company requiring strictly controlled keys

Scenario:
- compliance requires segregation of access to keys

Architecture:
- customer-managed keys in KMS
- minimal key policy + grants for applications
- audit trail via CloudTrail

## Case 4: Legacy application with a secret in a file

Scenario:
- password hardcoded in the code

Architecture:
- migrate to Parameter Store/Secrets Manager
- remove the credential from the repository
- use an IAM role at runtime
