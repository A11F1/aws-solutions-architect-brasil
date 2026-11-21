# 02 IAM and Security

## 📋 Table of Contents
- [Module Objectives](#-module-objectives)
- [Core Concepts](#-core-concepts)
- [Architecture and Components](#-architecture-and-components)
- [Important Settings for the Exam](#-important-settings-for-the-exam)
- [Service Comparison](#-service-comparison)
- [Exam Tips and Pitfalls](#-exam-tips-and-pitfalls)
- [For the Exam](#-for-the-exam)
- [Related Links](#-related-links)

## 🎯 Module Objectives

By the end of this module, you should be able to:

- precisely differentiate IAM users, groups, roles, and policies, knowing when to use each one
- understand the AWS permission evaluation logic and apply it in cross-account scenarios
- use permission boundaries and SCPs as governance tools without blocking legitimate operations
- distinguish the three types of KMS CMKs and describe how envelope encryption works
- decide between Secrets Manager and Parameter Store based on the functional requirement
- configure IAM Conditions to restrict access by region, MFA, VPC, or organization

## 📚 Core Concepts

### IAM Users, Groups, and Roles

An IAM User is a permanent identity that represents a person or an application. It can have a console password and up to two access keys for programmatic access. Access keys are long-term credentials and must never be shared or embedded in code — AWS recommends replacing them with roles whenever possible.

IAM Groups are logical containers for users. They make permission management easier at scale, since a policy is attached to the group and all members inherit it automatically. A user can belong to multiple groups. Groups cannot be nested and cannot assume roles, which is a crucial difference.

IAM Roles are identities that any eligible principal can temporarily assume via AWS Security Token Service. When assuming a role, the principal receives temporary credentials with an expiration time. A role has two components: the trust policy, which defines who can assume it, and the permissions policy, which defines what the role can do. Roles are the preferred mechanism for compute (EC2, Lambda, ECS tasks) and cross-account access.

### Policy Types

Policies are JSON documents with four core fields: Effect (Allow or Deny), Action (the API operation), Resource (which resources it applies to), and Condition (optional, additional criteria).

**Identity-based policies** are attached to users, groups, or roles. They can be AWS-managed (managed by AWS, read-only for the customer), customer-managed (created by the customer, reusable across multiple identities, versioned), or inline (embedded directly in an identity, 1:1 relationship, cannot be reused, deleted along with the identity).

**Resource-based policies** are attached to the resource, such as an S3 bucket policy, KMS key policy, SQS queue policy, or Lambda or SNS resource-based policy. The differentiator is that they explicitly specify the principal (who can access), which in several cases eliminates the need to assume a role for cross-account access.

**Permission Boundaries** are a permissions ceiling. Even if an identity policy grants broad access, the effective permission is the intersection between what the identity policy allows and what the permission boundary allows. This is essential for environments where administrators delegate the ability to create roles, but need to ensure the created roles do not exceed a maximum level.

**Service Control Policies (SCPs)** are applied at the AWS Organizations level to accounts or OUs. They work as a guardrail, limiting what any principal (including the account root) can do within an account — except for the management account, which is never limited by SCPs. SCPs do not grant permissions; they only define the ceiling of what is permissible.

### Permission Evaluation Logic

AWS evaluates whether an action is allowed following a defined order. First it checks whether there is an explicit deny in any policy — if so, the action is denied regardless of any allow. Then it goes through SCPs (if Organizations is enabled), permission boundaries, identity policies, and resource-based policies. For same-account access, a single allow in an identity or resource policy is enough. For cross-account, an allow is required in both layers: the identity OF the calling account (or trust + permission of the assumed role) and the resource policy in the destination account.

### IAM Conditions

Conditions let you restrict when and how a permission applies. The most commonly tested on the exam:

- `aws:RequestedRegion` — restricts calls to specific regions. Useful for implementing data residency.
- `aws:MultiFactorAuthPresent` — requires the caller to have authenticated with MFA.
- `aws:PrincipalOrgID` — restricts the principal to members of an organization. Very useful in resource policies.
- `aws:SourceVpc` and `aws:SourceVpce` — restricts access to traffic originating from a specific VPC or VPC endpoint.
- `aws:CurrentTime` — control based on date and time.
- `s3:prefix` — access control to specific prefixes in S3 buckets.

### AWS Organizations and IAM Identity Center

AWS Organizations groups multiple accounts under a managed hierarchy. The root contains the management account and OUs. SCPs are applied to OUs or directly to accounts. Consolidated billing unifies invoices.

IAM Identity Center (formerly AWS SSO) centralizes access to multiple accounts and applications. Instead of creating IAM users in each account, you define permission sets (a set of policies that represent an access level) and assign them to users and groups per account. It can integrate with Active Directory, Okta, and other external providers. For the exam, IAM Identity Center is the preferred answer when the scenario talks about managing employee access across multiple accounts.

### KMS — Key Management Service

KMS protects cryptographic keys, and the operations that use them never expose the key material. There are three types of Customer Master Keys (CMKs):

**AWS-owned CMK**: managed and maintained by AWS, completely transparent to the customer. You cannot see or manage them. Examples: S3 default encryption (SSE-S3), DynamoDB by default. Zero cost to the customer.

**AWS-managed CMK**: created automatically by AWS services (such as aws/s3, aws/ebs, aws/rds) in your account when the service needs them. You can view them and see audit usage in CloudTrail, but you do not manage them directly. Automatic annual rotation. Zero cost.

**Customer-managed CMK**: you create it, define the key policy, control rotation (optional annual), can enable/disable it, can import external material or use CloudHSM as a key store. Charged per month and per API usage.

**Key Policy**: every CMK must have a key policy. If no identity policy is configured, the key policy alone controls access. The default document gives access to the account root, which allows managing access via IAM policies. Without this clause, only the principals explicitly listed in the key policy have access.

**Envelope Encryption**: the encryption pattern for scale. The application generates a data key via KMS (GenerateDataKey). The data key is returned in two formats: plaintext (to encrypt the data) and encrypted (protected by the CMK). The application encrypts the data, discards the plaintext data key, and stores the encrypted data together with the encrypted data key. To decrypt, it sends the encrypted data key to KMS (Decrypt) and gets the plaintext data key back.

**Grants**: delegate access to a specific KMS operation programmatically, without changing the key policy. Useful when an application needs temporary access. Grants can be revoked.

**Multi-Region Keys**: a pair of primary and replica keys in different regions that share the same key material and key ID. Allows encrypting in one region and decrypting in another without re-encrypting.

### Secrets Manager vs Parameter Store

Secrets Manager is designed for secrets that need rotation. It integrates directly with RDS, Redshift, DocumentDB, and other services for automatic rotation without downtime. Each secret is billed monthly. It supports a resource-based policy for cross-account access.

Parameter Store is a hierarchical configuration store. The standard tier is free and supports up to 4KB. The advanced tier supports up to 8KB, policies (TTL, notification), and higher throughput. SecureString uses KMS for encryption. It has no native rotation, but can be integrated with Lambda to simulate rotation. The hierarchy (such as /app/prod/database/password) makes control via IAM easier, with a parameter equivalent of s3:prefix.

### ACM — AWS Certificate Manager

ACM provisions and renews TLS/SSL certificates. Public certificates are free and automatically renewed via DNS or email validation. DNS validation is preferable because it is automatic. ACM certificates can only be used with services that integrate natively: ALB, API Gateway, CloudFront, Elastic Beanstalk. It is not possible to export the private key of a public ACM certificate.

For CloudFront, the certificate must be in us-east-1 regardless of the distribution. For ALB and other regional services, the certificate must be in the same region.

ACM Private CA provisions a private certificate authority to issue internal certificates. It is charged per provisioned CA and per issued certificate.

## 🏗️ Architecture and Components

```text
AWS Organizations
    ├── Management Account
    └── OU: Production
         ├── Account A (workload account)
         │    └── IAM Identity Center → Permission Set → Assumed Role
         │
         └── Account B (shared account)
              └── S3 Bucket with resource policy:
                   Principal: arn:aws:iam::ACCOUNT_A:role/AppRole
                   Condition: aws:PrincipalOrgID = o-xxxx

Permission evaluation flow:
  API call
      │
      ▼ SCP allows?  ──NO──► DENY
      │YES
      ▼ Permission Boundary allows? ──NO──► DENY
      │YES
      ▼ Explicit Deny in any policy? ──YES──► DENY
      │NO
      ▼ Identity policy OR resource policy allows? ──NO──► DENY
      └──YES──► ALLOW
```

## ⚙️ Important Settings for the Exam

| Concept | Important detail |
|---|---|
| IAM Users limit per account | 5,000 |
| Access Keys limit per user | 2 |
| Default temporary credential duration | 1 hour (adjustable up to 12h with sts:DurationSeconds) |
| SCPs do not affect | the organization's management account |
| Permission Boundary affects | users and roles (not groups) |
| Customer-managed CMK rotation | optional, annual |
| AWS-managed CMK rotation | mandatory, annual |
| Secrets Manager pricing | US$ 0.40/secret/month + US$ 0.05/10,000 calls |
| Parameter Store Standard | free, up to 10,000 parameters, 4KB |
| Parameter Store Advanced | charged, up to 8KB, parameter policies |
| ACM CloudFront | certificate must be in us-east-1 |
| KMS GenerateDataKey | returns the key in plaintext and ciphertext |

## 🔄 Service Comparison

| Feature | Secrets Manager | Parameter Store SecureString |
|---|---|---|
| Native automatic rotation | ✅ Yes | ❌ Requires external Lambda |
| RDS integration | ✅ Native | ❌ Manual |
| Cost | Paid per secret | KMS API + Advanced parameter |
| Maximum size | 65,536 bytes | 8KB (Advanced) |
| Cross-account | Resource-based policy | Not natively |
| Main use case | Rotatable database credentials | Application configuration and simple secrets |

| Feature | AWS-owned CMK | AWS-managed CMK | Customer-managed CMK |
|---|---|---|---|
| Visibility in the account | No | Yes | Yes |
| Customer management | No | No | Yes |
| Rotation | AWS manages | Automatic annual | Annual (optional) |
| Cost | Free | Free | US$ 1/month + API |
| Customizable Key Policy | No | No | Yes |

| Policy | Who is affected | Where it is attached | Purpose |
|---|---|---|---|
| Identity policy | User / Role / Group | Identity | Grants permissions |
| Resource policy | Any external principal | Resource (S3, KMS, SQS...) | Direct cross-account access |
| Permission Boundary | User / Role | Identity | Defines the permissions ceiling |
| SCP | Accounts and OUs | Organizations | Organizational guardrail |
| Session Policy | Session principal | Passed to AssumeRole | Limits the temporary session |

## 💡 Exam Tips and Pitfalls

- SCPs never affect the management account. If the question mentions that "not even the admin can do X", think permission boundary, not SCP.
- For cross-account access without assuming a role, use a resource-based policy with `aws:PrincipalOrgID` — it is simpler and more scalable.
- A KMS key policy is not the same as an IAM policy. The key policy is mandatory and separate. Without the statement that gives control to the account root, no one other than those explicitly in the key policy can get access — not even an IAM admin.
- An AWS-managed CMK does not appear as selectable for custom encryption. If the question requires control of the key lifecycle, the answer is customer-managed CMK.
- Envelope encryption does not store the data key in plaintext. It always stores the encrypted version alongside the data.
- IAM Groups do not assume roles. If the scenario requires a group of EC2 instances to access S3, the answer is an IAM Role on the EC2 Instance Profile, not an IAM group.
- Rotating credentials for applications: the answer is Secrets Manager with automatic rotation, especially when the question mentions RDS, Redshift, or DocumentDB.
- IAM Identity Center is the answer for centralized workforce access management across multiple accounts. IAM Users in each account is an anti-pattern for this scenario.

## 💡 For the Exam

- Memorize the evaluation logic: explicit deny wins, then SCP, then boundary, then identity/resource policy.
- Cross-account = role assumption (trust policy + permission policy) OR resource-based policy with an explicit principal.
- Automatic rotation = Secrets Manager, not Parameter Store.
- Encryption key control = customer-managed CMK.
- Multi-account workforce access = IAM Identity Center.
- Restrict access by VPC = `aws:SourceVpc` condition, not NACL or Security Group alone.

## 📎 Related Links

- [Module questions](./questoes.md)
- [Module flashcards](./flashcards.md)
- [Module cheatsheet](./cheatsheet.md)
- [Module use cases](./casos-de-uso.md)
- [Module hands-on lab](./lab.md)
- [Official links](./links.md)
- [Module 01: Introduction to SAA-C03](../01-Introducao-SAA-C03/)
- [Module 07: VPC and Networking](../08-VPC-e-Redes/)
- [Module 14: Monitoring](../20-Monitoramento-CloudWatch-CloudTrail/)
