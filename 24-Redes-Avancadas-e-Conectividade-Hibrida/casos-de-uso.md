# Use Cases - Advanced Networking and Hybrid Connectivity

## Case 1: Multi-account corporate group

Scenario:
- 50 VPCs across several accounts
- need for controlled communication

Architecture:
- Central Transit Gateway
- separate route tables per domain
- traffic inspection in a security VPC

## Case 2: Private SaaS for enterprise customers

Scenario:
- customers in different AWS accounts
- no public exposure

Architecture:
- NLB + Endpoint Service
- consumption via Interface VPC Endpoint (PrivateLink)

## Case 3: Datacenter to AWS with low latency variation

Scenario:
- workloads sensitive to network variation
- high data volume

Architecture:
- AWS Direct Connect as primary
- Site-to-Site VPN as backup
- BGP for failover

## Case 4: Hybrid DNS integration

Scenario:
- on-premises applications need to resolve private zones in AWS

Architecture:
- Route 53 Resolver inbound endpoint
- conditional forwarding on the on-premises DNS
