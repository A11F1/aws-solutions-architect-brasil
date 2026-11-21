# 25 Encryption, KMS, and Secrets Management

## Module objectives

- master encryption at rest and in transit in the SAA-C03 context
- differentiate AWS KMS, CloudHSM, Secrets Manager, Parameter Store, and ACM
- decide between AWS-managed and customer-managed keys based on compliance
- apply security patterns with the lowest operational overhead

## Core concepts

The SAA-C03 exam tests applied security: knowing the service name is not enough, you need to choose the best combination of encryption, access control, and secret rotation.

## KMS in practice

- KMS manages keys and auditable cryptographic operations.
- Customer managed keys allow control of policy, rotation, and grants.
- Key policy is required for correct access.
- Envelope encryption is the standard for data at scale.

Summary of the envelope encryption flow:
1. the application requests a data key from KMS (GenerateDataKey)
2. KMS returns the data key in plaintext and encrypted form
3. the application encrypts the data with the plaintext key
4. it stores the encrypted data + the encrypted data key

## Secrets and certificates

- Secrets Manager: secrets with automatic rotation (strong for database credentials).
- Parameter Store SecureString: simple configuration and secrets, lower cost.
- ACM: TLS certificates for ALB, CloudFront, and API Gateway.
- ACM Private CA: issuing internal private certificates.

## Exam tips

- if there is a requirement for automatic credential rotation, the answer tends to be Secrets Manager.
- if it asks for "full control over encryption keys", use a customer managed KMS key.
- if you need a dedicated HSM and specialized cryptographic control, consider CloudHSM.
- for CloudFront with ACM, the public certificate must be in us-east-1.
- encryption without permission control does not solve the risk; IAM + key policy matter.

## Related links

- [Cheatsheet](./cheatsheet.md)
- [Use cases](./casos-de-uso.md)
- [Questions](./questoes.md)
- [Flashcards](./flashcards.md)
- [Lab](./lab.md)
- [Official links](./links.md)
