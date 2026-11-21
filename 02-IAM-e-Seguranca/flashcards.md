# Flashcards — IAM and Security

> Review one card at a time. Try to answer before revealing the answer.

---

**Q:** What is the difference between an IAM User and an IAM Role?  
**A:** A User has permanent credentials (password + access key). A Role provides temporary credentials via STS, assumed by AWS services, other accounts, or federated identities. It never has its own password or access key.

---

**Q:** What is a Permission Boundary and what is it for?  
**A:** It is a managed IAM policy that defines the maximum permissions a user or role can have — even if the identity policy grants more. Used to delegate role creation without privilege escalation.

---

**Q:** What is the difference between an AWS-managed policy, a Customer-managed policy, and an Inline policy?  
**A:** AWS-managed: maintained by AWS, reusable, not editable. Customer-managed: you create and control it, reusable across multiple identities. Inline: embedded in the identity, 1:1, deleted along with it — avoid except for exclusive permissions.

---

**Q:** In IAM policy evaluation, what comes first: SCP, Permission Boundary, or identity policy?  
**A:** Explicit Deny (any source) → does the SCP block? → does the Permission Boundary block? → does the identity policy or resource policy allow? For cross-account access: both identity AND resource policy must allow.

---

**Q:** What does the `aws:PrincipalOrgID` condition do?  
**A:** Restricts access to a resource to only principals that belong to a specific AWS Organization. Useful in bucket policies to ensure only the company's accounts access the bucket, without listing each account ID.

---

**Q:** What is an SCP and who does it affect?  
**A:** Service Control Policy — a maximum-permissions guardrail in AWS Organizations. It applies to all member accounts and OUs, but NEVER to the management account. An explicit allow in the SCP does not grant permission on its own — the identity policy must still allow it.

---

**Q:** What is the difference between `aws:SourceVpc` and `aws:SourceVpce`?  
**A:** `aws:SourceVpc` restricts to the VPC ID. `aws:SourceVpce` restricts to the specific VPC Endpoint ID. Use `aws:SourceVpce` when you want to ensure that only access via a specific endpoint is allowed.

---

**Q:** What is IAM Identity Center (formerly AWS SSO) used for?  
**A:** Centralized single sign-on for multiple AWS accounts and SaaS applications. It lets you create permission sets (a set of policies) and assign them to users/groups per account. It integrates with AD, Okta, and other identity providers via SAML 2.0/SCIM.

---

**Q:** What are the three KMS CMK types?  
**A:** 1) AWS-owned: managed by AWS, not visible to the customer, no additional cost (e.g., S3 SSE with aws/s3). 2) AWS-managed CMK: created by the service in your account (aws/ebs), visible, automatic annual rotation, key policy cannot be edited. 3) Customer-managed CMK: you create it, pay monthly, edit the key policy, define rotation, can disable it.

---

**Q:** What is envelope encryption?  
**A:** A pattern in which: (1) KMS generates a Data Key (plaintext + encrypted with the CMK); (2) the client uses the plaintext data key to encrypt the data; (3) it discards the plaintext key and stores the encrypted data key alongside the data; (4) to decrypt, it sends the encrypted data key to KMS and uses the returned plaintext key.

---

**Q:** What is the purpose of the KMS key policy?  
**A:** It defines who can manage and use the CMK. It is mandatory — every CMK needs a key policy. The policy includes the root account as an administrator (ensures recovery), key administrators, and key users. It is a resource-based policy, so it allows cross-account access without role assumption.

---

**Q:** What are KMS Grants and when should they be used?  
**A:** Grants give temporary, programmatic permission to use a CMK without changing the key policy. They are created via `CreateGrant` and the grant token is passed in API calls (`kms:decrypt --grant-tokens`). Used internally by AWS services (e.g., EBS when it creates a snapshot of an encrypted volume).

---

**Q:** What are KMS Multi-Region Keys?  
**A:** Keys that have the same key material replicated across multiple regions. They allow encrypting in us-east-1 and decrypting in eu-west-1 without re-encrypting. Key ARNs are different but the material is the same. Useful for cross-region DR with encrypted data.

---

**Q:** What is the main difference between Secrets Manager and Parameter Store?  
**A:** Secrets Manager: native automatic rotation (integrates with RDS, Redshift, DocumentDB), charges per secret per month. Parameter Store: no native rotation, Standard is free (max 4KB), SecureString uses KMS. Secrets Manager is preferable for credentials with rotation; Parameter Store for application configs.

---

**Q:** What is ACM and what are its limitations?  
**A:** AWS Certificate Manager manages SSL/TLS certificates. Public certificates are free and automatically renewed. They can only be used with integrated AWS services (ALB, CloudFront, API Gateway) — they cannot be exported. For CloudFront, the certificate must be in us-east-1.

---

**Q:** What is ACM Private CA?  
**A:** A private Certificate Authority managed by AWS. It issues private certificates for internal services (mTLS, microservices). It has a monthly cost per CA + per certificate. Issued certificates ARE exportable, unlike ACM public ones.

---

**Q:** How does cross-account role assumption work?  
**A:** Account A creates a role with a trust policy allowing account B to assume it (sts:AssumeRole). Account B creates a policy allowing sts:AssumeRole on the ARN of account A's role. The user in account B calls AssumeRole and receives temporary credentials from account A.

---

**Q:** Which IAM condition requires MFA on an API call?  
**A:** `"Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}}` — ensures the operation only occurs when the principal authenticated with MFA. Useful to protect sensitive actions such as deleting S3 objects or disabling a CMK.

---

**Q:** What is the difference between an identity-based policy and a resource-based policy for cross-account access?  
**A:** With a resource-based policy (e.g., S3 bucket policy, KMS key policy) that explicitly allows the principal from another account: direct access without role assumption. With an identity-based policy only: you need to assume a role. Resource-based policies allow cross-account access without a role hop — considered more direct for services that support them.

---

**Q:** What is Parameter Store SecureString and what are its KMS key options?  
**A:** SecureString encrypts the value using KMS. It can use the aws/ssm CMK (no CMK cost, but no key policy control) or a customer-managed CMK (full control, auditing, can revoke access). For cross-account use, a customer-managed CMK is required.

---

**Q:** What does `"Effect": "Deny"` mean in an IAM policy vs an SCP?  
**A:** In both cases, an explicit Deny overrides any Allow in any other policy. In IAM: if any applicable policy has a Deny for the action, the call is denied. In an SCP: the Deny blocks the entire account/OU, even if identity policies allow.
