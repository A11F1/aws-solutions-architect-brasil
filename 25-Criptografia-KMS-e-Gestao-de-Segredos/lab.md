# Lab - KMS + Secrets Manager + IAM Role

## Objective
Protect application credentials using a rotatable secret and role-based access.

## Steps

1. Create a KMS customer-managed key for secrets.
2. Create a secret in Secrets Manager (test username/password) using the key.
3. Create an IAM role for the workload (EC2/Lambda) with the `secretsmanager:GetSecretValue` and `kms:Decrypt` permissions.
4. Associate the role with the compute resource.
5. Read the secret via SDK/CLI without static credentials.
6. Enable automatic rotation (when applicable).

## Validation

- workload reads the secret with the role
- access denied for a principal without permission
- access events appear in CloudTrail

## Cleanup

- remove the lab secret
- remove the role/policies and the created key
