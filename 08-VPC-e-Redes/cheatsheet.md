# Cheatsheet — Module 07: VPC and Networking

## Security Groups vs NACLs

| | Security Group | NACL |
|---|---|---|
| Level | Instance/ENI | Subnet |
| State | **Stateful** (automatic response) | **Stateless** (separate inbound AND outbound rules) |
| Rules | Allow only | Allow and **Deny** |
| Processing | All rules evaluated | Numeric order; stops at the 1st match |
| Default | Blocks all inbound; allows all outbound | Allow all (inbound and outbound) |
| Association | N SGs per ENI, 1 SG across multiple ENIs | 1 NACL per subnet; 1 subnet per NACL |

## VPC Networking Components

| Component | Function |
|---|---|
| Internet Gateway (IGW) | Outbound + inbound internet for a public subnet (IPv4/IPv6) |
| NAT Gateway | Internet outbound for a private subnet (IPv4 only); zonal |
| Egress-Only IGW | Internet outbound for a private subnet (IPv6 only) |
| Virtual Private Gateway (VGW) | AWS endpoint for Site-to-Site VPN and DX Private VIF |
| Customer Gateway (CGW) | Represents the on-prem device in the VPN configuration |
| Transit Gateway (TGW) | Central hub for multiple VPCs and on-prem connections (transitive) |

## VPC Endpoints

| | Gateway Endpoint | Interface Endpoint (PrivateLink) |
|---|---|---|
| Services | S3, DynamoDB (only) | ~200+ AWS services + custom |
| How it works | Route table entry | ENI with a private IP from the VPC |
| Cost | Free | Billed ($/hour + $/GB) |
| On-premises | Not accessible | Accessible via DX/VPN |
| DNS | Does not change DNS | Private DNS option → resolves to ENI IP |

## Direct Connect (DX)

| Type | Capacity | Provisioning | Via |
|---|---|---|---|
| Dedicated Connection | 1/10/100 Gbps | Weeks | Directly with AWS |
| Hosted Connection | 50 Mbps to 10 Gbps | Faster | Via APN partner |

**Virtual Interfaces (VIF):**
- **Private VIF** → VPC via VGW or DX Gateway
- **Public VIF** → AWS public endpoints (S3, EC2 IPs)
- **Transit VIF** → Transit Gateway (multiple VPCs)

**DX + VPN:** add an IPSec VPN over DX to encrypt data in transit (DX by itself does not encrypt)

## Site-to-Site VPN

| | Site-to-Site VPN | Accelerated VPN |
|---|---|---|
| Path | Public internet (HTTPS/IPSec) | AWS edge → private backbone |
| Latency | Variable | Reduced |
| Cost | Lower | Accelerator fee + hourly or additional transfer charges |
| Throughput | ~1.25 Gbps per tunnel | ~1.25 Gbps per tunnel |

- Each VPN connection has **2 tunnels** (redundancy; HA via 2 customer gateways is recommended)
- VGW = AWS side; CGW = on-premises side

## Routing: Peering vs Transit Gateway

| | VPC Peering | Transit Gateway |
|---|---|---|
| Transit | Not transitive | **Transitive** |
| Scale | N*(N-1)/2 connections for N VPCs | N attachments per TGW |
| Cross-region | Yes (inter-region peering) | Yes (TGW Peering) |
| Cost | No connection cost | Cost per attachment + per GB |
| CIDR overlap | Prevents peering | Does not allow communication (but attachment is possible) |

## Reserved IPs per AWS Subnet
```
10.0.0.0/24 (256 IPs) → 251 available:
  10.0.0.0  → Network address
  10.0.0.1  → VPC router (default gateway)
  10.0.0.2  → DNS (Amazon DNS = VPC_CIDR_base + 2)
  10.0.0.3  → Reserved by AWS (future use)
  10.0.0.255 → Broadcast
```

## VPC Flow Logs — Main Fields
```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
```
- Destinations: CloudWatch Logs or S3
- Does NOT capture: DNS (Route 53), DHCP, IMDS (169.254.x.x), Windows license activation

## Quick Exam Tips
- NAT Gateway: create **1 per AZ** for HA; each AZ's private subnet routes to its own AZ's NAT GW
- VPC Peering: CIDRs cannot overlap; not transitive; works cross-region
- Transit Gateway: replaces a mesh of peerings for large topologies; supports multicast
- Gateway Endpoint: adds a route to the route table; no public IP needed to access S3/DynamoDB
- Interface Endpoint with Private DNS: resolves the service's public name to a private IP in the VPC
- PrivateLink: ideal for exposing SaaS or internal services to other VPCs without peering
- Direct Connect: does NOT encrypt by default; add a VPN over DX for encryption
- Egress-Only IGW: for IPv6 outbound only (NAT does not exist for IPv6)
- Route Tables: one per subnet; subnets without an explicit association use the main route table
