# Questions — IAM and Security

## Question 1
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A security team wants to ensure that developers can create IAM roles for their applications, but that these roles can never have permission to access the AWS billing service or export KMS keys. Which IAM mechanism should be configured?

- A) AWS Organizations SCP applied to the developers' OU
- B) IAM Permission Boundary applied to the roles created by the developers
- C) IAM Group with Explicit Deny for Billing and KMS
- D) S3 Bucket Policy with a PrincipalOrgID condition

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

Permission Boundaries define the ceiling of effective permissions for a role or user. Even if a developer creates a role with full access permission, if the boundary does not include Billing or KMS:ExportKey, those accesses are blocked. Option A would be correct to restrict an entire account, but does not solve the problem of roles created by the developers themselves within the account. Option C blocks the dev group, but not the roles they create. Option D is an S3 resource-based policy and does not solve the case.

**Key concept:** Permission Boundary as a ceiling for controlled delegation
</details>

## Question 2
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Hard

A company has multiple AWS accounts in an organization. The security team wants to ensure that no account, regardless of any local admin, can disable CloudTrail. Which solution accomplishes this?

- A) Create an IAM Permission Boundary with Deny for cloudtrail:StopLogging in each account
- B) Create an SCP in the organization with Deny for cloudtrail:StopLogging and apply it to the root
- C) Create an IAM Policy with Deny for CloudTrail at the account level
- D) Configure a bucket policy on the CloudTrail bucket

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

SCPs applied to the organization root affect all accounts (except the management account) and no local administrator can override an SCP explicit deny. Option A requires configuration in each account individually and can be removed by a local admin. Option C is an IAM policy that can also be modified by an account admin. Option D protects the bucket, but does not prevent StopLogging or DeleteTrail before the logs reach the bucket.

**Key concept:** SCP as an organizational guardrail that local admins cannot override
</details>

## Question 3
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A Flask application running on EC2 needs to read database secrets with automatic rotation every 30 days. The compliance requirement states that credentials must never be stored in code or configuration files. What is the most appropriate solution?

- A) Store the credentials in environment variables on the instance
- B) Use AWS Secrets Manager with an IAM role on the EC2 instance and automatic rotation configured
- C) Store the credentials in Parameter Store Standard and read them in cloud-init
- D) Embed the credentials in the instance User Data

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

Secrets Manager is the preferred solution for rotatable credentials. The EC2 instance uses an IAM role (Instance Profile) to call the Secrets Manager API without exposing credentials. Automatic rotation is native for RDS, Redshift, and other databases. Option A stores them in plaintext in environment variables. Option C has no native rotation. Option D exposes credentials in User Data, visible via metadata.

**Key concept:** Secrets Manager + Instance Profile for rotatable credentials
</details>

## Question 4
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Hard

A team wants calls to Amazon S3 from a specific VPC to be allowed only for objects with the prefix `/reports/`. Via bucket policy, which combination of elements implements this correctly?

- A) Condition: aws:SourceIp pointing to the VPC CIDR + resource with prefix /reports/*
- B) Condition: aws:SourceVpce pointing to the VPC Endpoint ID + resource with prefix /reports/*
- C) Permission Boundary with s3:prefix = /reports/
- D) SCP with Condition: aws:SourceVpc and the bucket resource

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

`aws:SourceVpce` identifies traffic passing through a specific VPC Endpoint, and is more precise than the VPC IP (which can change). Combined with the resource `arn:aws:s3:::bucket/reports/*`, it restricts access to the correct prefix. Option A uses an IP, which is not the recommended way to restrict via VPC. Option C applies a Permission Boundary, which does not solve the prefix restriction in a bucket policy. For option D, SCPs do not operate on service-specific resource policies.

**Key concept:** aws:SourceVpce in a bucket policy for access via VPC Endpoint
</details>

## Question 5
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A company encrypts data in Amazon RDS using KMS. An auditor wants to verify which type of CMK was used to ensure that the company controls the entire key lifecycle. Which CMK type meets this requirement?

- A) AWS-owned CMK
- B) AWS-managed CMK (aws/rds)
- C) Customer-managed CMK created by the customer
- D) Data key generated by RDS

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: C**

Only the customer-managed CMK gives the customer full control: key policy definition, enabling/disabling, controlled rotation, and detailed auditing via CloudTrail. An AWS-owned CMK is not visible in the account. An AWS-managed CMK is visible, but the customer does not manage it. A data key is derived from a CMK and is not what the auditor is asking about.

**Key concept:** customer-managed CMK for control of the cryptographic lifecycle
</details>

## Question 6
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A new feature needs to request S3 objects from account B using a Lambda in account A. The team prefers to avoid creating a role in account B for simplicity. Which approach works?

- A) Add an SCP in account B that allows the Lambda ARN
- B) Add account A as a principal in the bucket policy of the bucket in account B
- C) Create an IAM user in account B and use the credentials in the Lambda in account A
- D) Configure VPC Peering between the accounts

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

A resource-based policy (bucket policy) can specify a principal from another account without needing to create a role. The Lambda in account A must have permission for s3:GetObject in its execution role, and the bucket in account B must have the bucket policy that allows that ARN. This is the simplest form of cross-account access without role assumption. In option A, an SCP does not grant permission, it only restricts. Option C shares long-term credentials, which is bad practice. Option D does not solve IAM; VPC Peering is about networking.

**Key concept:** cross-account via resource-based policy without role assumption
</details>

## Question 7
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Hard

An architect needs to implement envelope encryption for sensitive data stored in S3. What is the correct sequence of operations?

- A) Encrypt the data directly with the CMK, store it in S3
- B) Generate a data key via KMS, encrypt the data with it in plaintext, encrypt the data key with the CMK, store the encrypted data + encrypted data key
- C) Store the data key in plaintext together with the encrypted data
- D) Use only the data key without any CMK

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

Envelope encryption works as follows: (1) a GenerateDataKey call to KMS returns the data key in plaintext and ciphertext; (2) encrypt the data with the plaintext data key; (3) discard the plaintext data key from memory; (4) store the encrypted data together with the ciphertext data key. To decrypt, send the ciphertext data key to KMS (Decrypt) to get the plaintext back. Option A encrypts directly with the CMK, which is inefficient for large volumes and does not scale. Option C exposes the data key. Option D does not protect the data key.

**Key concept:** correct sequence of envelope encryption with KMS
</details>

## Question 8
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A multinational company wants all employees from different subsidiaries to access multiple AWS accounts with single sign-on (SSO) via the existing corporate Active Directory. Which service is most suitable?

- A) Create IAM Users in each AWS account
- B) IAM Identity Center integrated with Active Directory via SAML or SCIM
- C) AWS Cognito User Pools
- D) IAM Roles with ADFS as the IdP per individual account

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

IAM Identity Center offers centralized access management for multiple accounts, native integration with AD via AWS Managed AD or AD Connector (it also supports SAML/SCIM with other IdPs such as Okta), and permission sets that map the access level per account. It eliminates the need to manage IAM users in each account. Option A is the highest-effort and least secure approach. Option C is for authenticating customers/application users, not internal employees. Option D is operationally complex to maintain at scale.

**Key concept:** IAM Identity Center for centralized multi-account SSO
</details>

## Question 9
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A developer inadvertently deleted a customer-managed CMK from KMS that was being used to encrypt data on EBS. What is the state of the CMK after the delete action?

- A) The CMK is deleted immediately and the data is unrecoverable
- B) The CMK enters a waiting period of 7 to 30 days before effective deletion, and the deletion can be canceled
- C) The CMK is disabled but remains available indefinitely
- D) The CMK is automatically archived to S3

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

KMS implements a waiting period of 7 to 30 days (configurable, default 30 days) before the effective deletion of a CMK. During this period, the deletion can be canceled. No cryptographic operation can be performed with the key during the waiting period. This protects against accidental deletion. If the deletion occurs, data encrypted exclusively by that key becomes unrecoverable.

**Key concept:** CMK deletion waiting period in KMS
</details>

## Question 10
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A multi-microservice application uses a VPC with private subnets. Each microservice needs access to S3 without exposing traffic to the public internet. Which solution is the most efficient and secure?

- A) Configure a NAT Gateway for S3 traffic
- B) Create a VPC Gateway Endpoint for S3
- C) Create a VPC Interface Endpoint for S3
- D) Use Direct Connect to access S3

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

The Gateway Endpoint for S3 (and DynamoDB) is free and is reached via the route table. Traffic stays within the AWS network without needing a NAT Gateway. It is the most cost-effective solution. Option A involves NAT Gateway processing and transfer costs in addition to routing to the internet. In option C, an Interface Endpoint for S3 (PrivateLink) also works, but has an hourly and per-GB cost — preferable when you need to access S3 from on-premises via Direct Connect. Option D is for on-premises connectivity.

**Key concept:** Gateway Endpoint for S3 with no data cost vs Interface Endpoint with cost
</details>

## Question 11
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Easy

An IAM policy has the following statements: an Allow for s3:GetObject on any bucket, and an explicit Deny for s3:GetObject on the bucket `logs-confidenciais`. What happens when the user tries to access an object in `logs-confidenciais`?

- A) The Allow in the first statement prevails
- B) The explicit Deny prevails and access is denied
- C) It depends on the order of the statements in the JSON
- D) Another Allow policy is needed to resolve the conflict

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

The fundamental rule of IAM evaluation is: an explicit Deny always overrides any Allow, regardless of the order of statements or how many policies allow. There is no way for an Allow to override an explicit Deny in the same account.

**Key concept:** explicit Deny has absolute priority over Allow
</details>

## Question 12
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Hard

A company wants TLS certificates for APIs exposed via ALB to be managed centrally, with automatic renewal and no manual intervention. Which combination of services accomplishes this?

- A) CloudFront with a self-signed certificate created by the team
- B) ACM with a public certificate for the domain, DNS validation, integrated with the ALB
- C) Generate a certificate with OpenSSL, import it into IAM, and attach it to the ALB
- D) EC2 with Nginx managing certificates manually

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

ACM provisions and renews public certificates automatically. DNS validation is the preferred method because it is automated via a CNAME record. The certificate is integrated directly with the ALB without needing to export the private key. Option A is not centralized management and self-signed is not for production. Option C requires manual deletion and recreation at renewal. Option D unnecessarily increases operational effort.

**Key concept:** ACM with DNS validation for zero-touch renewal
</details>

## Question 13
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

A team needs to differentiate Parameter Store from Secrets Manager when choosing where to store a boolean feature flag with no sensitive value and a rotatable database password. Which allocation is correct?

- A) Both in Secrets Manager to standardize
- B) Feature flag in Parameter Store Standard; database password in Secrets Manager
- C) Both in Parameter Store Advanced
- D) Feature flag in Secrets Manager; database password in the EC2 Systems Manager agent

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

Parameter Store Standard is free and ideal for non-sensitive configuration or simple secrets without rotation. Secrets Manager is the most suitable for credentials that require automatic rotation. Using Secrets Manager for feature flags pays for something that was not needed. Option C uses Advanced unnecessarily. Option D mixes concepts incorrectly.

**Key concept:** Parameter Store for config/simple secrets; Secrets Manager for rotation
</details>

## Question 14
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Hard

An organization needs a KMS key to be used in both us-east-1 and eu-west-1 to encrypt and decrypt data without needing to re-encrypt when moving data between regions. Which KMS feature meets this requirement?

- A) Cross-region replication of a CMK via S3
- B) KMS Multi-Region Keys with a replica in eu-west-1
- C) Using the same exported data key in both regions
- D) Create independent CMKs in each region and use AWS DataSync to synchronize

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

Multi-Region Keys share the same key material across distinct regions, using the same key ID (with the mrk- prefix). Data encrypted in us-east-1 can be decrypted in eu-west-1 without re-encrypting, because the keys are cryptographically equivalent. Option A does not exist. Option C, an exported data key, exposes the material outside KMS — it breaks the security model. Option D creates keys with different material, preventing direct cross-region decrypt.

**Key concept:** KMS Multi-Region Keys for cross-region encrypt/decrypt
</details>

## Question 15
**Domain:** Design Secure Applications and Architectures  
**Difficulty:** Medium

An EC2 instance in a production environment needs only permissions to put metrics into CloudWatch and nothing else. What is the best least-privilege approach?

- A) Create an IAM User with the AmazonCloudWatchFullAccess policy
- B) Create an IAM Role with a customer-managed policy granting only cloudwatch:PutMetricData and associate it with the instance via Instance Profile
- C) Create an IAM Group with CloudWatch permission and add the instance to the group
- D) Use the admin access key in the AWS_ACCESS_KEY_ID environment variable

<details>
<summary>✅ Answer and Explanation</summary>

**Correct answer: B**

IAM Roles with Instance Profile provide automatically rotated temporary credentials. The policy should be minimal: only `cloudwatch:PutMetricData` on resource `*`. It avoids the use of long-term credentials. Option A uses a user (long-term credentials) and Full Access permissions — it violates least privilege. For option C, instances cannot be members of IAM groups. Option D is the worst practice: hardcoded credentials in the environment.

**Key concept:** Instance Profile + least privilege policy for EC2
</details>
