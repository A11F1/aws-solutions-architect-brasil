# Module 07 — VPC and Networking

## Objective

In this module, you will learn to design secure, resilient, and scalable VPC networks, choosing the right connectivity, segmentation, routing, and private access for scenarios covered on the exam.

## Core AWS services

- Amazon VPC
- AWS Transit Gateway
- AWS Direct Connect
- AWS Site-to-Site VPN
- AWS PrivateLink (Interface Endpoints)

## Architecture and trade-offs

## VPC Fundamentals

A VPC (Virtual Private Cloud) is a logically isolated virtual network within AWS. Each account gets a **Default VPC** per Region.

### Main Components
```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24) — AZ-a
│     ├── Internet Gateway → route 0.0.0.0/0
│     └── Instances with public IP
├── Private Subnet (10.0.2.0/24) — AZ-a
│     ├── NAT Gateway (outbound to internet, no inbound)
│     └── Instances without public IP
├── Private Subnet (10.0.3.0/24) — AZ-b
│     └── (multi-AZ redundancy)
├── Route Tables (one per subnet or shared)
├── Network ACLs (stateless, subnet level)
├── Security Groups (stateful, instance/ENI level)
├── DHCP Options Set
├── Elastic IPs
└── ENIs (Elastic Network Interfaces)
```

### CIDRs
- **Primary**: assigned at creation (cannot be changed), e.g., `10.0.0.0/16`
- **Secondary**: up to 5 additional blocks can be added
- Subnet size: `/28` (minimum) to `/16` (maximum)
- AWS **reserves 5 IPs** per subnet (.0 network, .1 VPC router, .2 DNS, .3 reserved, .255 broadcast)

---

## Security Groups vs Network ACLs

| Characteristic | Security Group | Network ACL |
|---|---|---|
| Level | Instance / ENI | Subnet |
| State | **Stateful** (automatic response) | **Stateless** (independent inbound and outbound rules) |
| Rules | ALLOW only | ALLOW and DENY |
| Evaluation | All rules evaluated | Rules in numeric order (1, 100, 200...) — first match wins |
| New default | Denies everything, needs rules | Allow All inbound/outbound |
| Reference | Can reference another SG | CIDR only |

**Inbound traffic evaluation order:**
1. NACL (subnet level) → 2. Security Group (instance level)

---

## NAT Gateway vs NAT Instance

| Characteristic | NAT Gateway | NAT Instance (legacy) |
|---|---|---|
| Managed | AWS manages | You manage (EC2) |
| Scaling | Automatic (up to 100 Gbps) | Manual (instance type) |
| Availability | Highly available per AZ | Single point of failure |
| Cost | Per hour + per GB processed | EC2 per hour + EIP |
| Source/Dest Check | Automatic | Must be disabled manually |
| Security Groups | No (uses the subnet's NACL) | Yes (SG on the instance) |

**Important:** NAT Gateway is zonal — for real HA, use **one NAT GW per AZ** with a route table per AZ pointing to the local NAT GW.

---

## VPC Peering

Private network connection between two VPCs (same or different Regions/accounts).

- Traffic uses the AWS private network (not the internet)
- **Not transitive**: VPC A ↔ VPC B and VPC B ↔ VPC C does **not** imply VPC A ↔ VPC C
- CIDRs cannot overlap
- Requires updating route tables in both VPCs
- Supports cross-account and cross-region

```
VPC A ←──── peering ────→ VPC B
VPC B ←──── peering ────→ VPC C
VPC A ←────────────────── VPC C  ← NEEDS a separate peering
```

---

## Transit Gateway (TGW)

Central hub that connects multiple VPCs and on-premises networks:

```
VPC A ──┐
VPC B ──┤
VPC C ──┼──→ Transit Gateway ←──→ VPN / Direct Connect
VPC D ──┤
VPC E ──┘
```

- **Transitive** by default (A ↔ TGW ↔ C = A ↔ C, unlike peering)
- TGW **Route Tables** control which VPC can communicate with which
- Supports **multicast** (the only AWS service that supports it)
- TGW Resource Sharing via AWS RAM (share across accounts in the Org)
- **TGW Peering**: connects TGWs in different Regions
- Ideal for hub-and-spoke with dozens/hundreds of VPCs

---

## VPC Endpoints

Allow private access to AWS services **without leaving the VPC** (no IGW, NAT, VPN, DX):

### Gateway Endpoints
- Free
- Only for **S3 and DynamoDB**
- Added as an entry in the route table (service prefix → endpoint ID)
- Cannot be extended to on-premises or VPC peering

### Interface Endpoints (PrivateLink)
- ENI with a private IP in the VPC
- Billed per hour + per GB processed
- Supports **most AWS services** and third-party services
- Access via private DNS (Endpoint-specific DNS names or Private DNS enabled)
- Can be accessed from on-premises via VPN/DX ✅

---

## AWS Direct Connect (DX)

Dedicated network connection between on-premises and AWS:

| Type | Speed | SLA | Provisioning |
|---|---|---|---|
| Dedicated Connection | 1, 10, 100 Gbps | AWS SLA | Directly with AWS, weeks |
| Hosted Connection | 50 Mbps – 10 Gbps | Partner | Through a partner, more flexible |

### Virtual Interfaces (VIFs)
- **Public VIF**: accesses AWS public endpoints (S3, DynamoDB, KMS API)
- **Private VIF**: accesses private VPC IPs
- **Transit VIF**: accesses Transit Gateway (for multiple VPCs via DX)

### Direct Connect Gateway
- Connects one DX connection to multiple VPCs in **different Regions** (same account)
- Reduced number of Private VIFs (1 connection → multiple Regions/VPCs)

### High Availability with DX
- **Backup DX**: two circuits at different points of presence
- **DX + VPN as backup**: Site-to-Site VPN as failover (lower cost, higher latency)
- **LAG (Link Aggregation Group)**: multiple physical circuits aggregated as one logical

---

## Site-to-Site VPN

Encrypted (IPSec) connection between on-premises and a VPC:
- **Virtual Private Gateway (VGW)**: terminates the VPN on the VPC side
- **Customer Gateway (CGW)**: physical/software device on-premises
- VPN goes over the public internet (variable latency, encrypted)
- Throughput: up to ~1.25 Gbps (limited by the VGW)
- **Accelerated Site-to-Site VPN**: uses Global Accelerator to reduce latency (leaves the internet at the nearest PoP)

---

## VPC Flow Logs

- Captures IP traffic metadata (not the payload) on VPCs, subnets, or ENIs
- Destinations: **CloudWatch Logs** or **S3**
- Includes: source/destination IP, port, protocol, bytes, packets, action (ACCEPT/REJECT), ENI ID
- Does not capture: DNS queries, DHCP, metadata (169.254.x.x), AWS link-local
- Latency: ~10–15 min after capture (near-real-time)

---

## IPv6 in the VPC

- `/56` block assigned to the VPC (AWS chooses; there are no private IPv6 ranges)
- Subnets receive a `/64`
- **All IPv6 addresses are public** (there is no NAT for IPv6)
- **Egress-Only Internet Gateway**: allows IPv6 outbound without receiving inbound traffic (equivalent to NAT GW for IPv6)

---

## Elastic Network Interface (ENI)

Virtual network interface that can be:
- Moved between instances (for failover while keeping the private IP and security groups)
- Multiple ENIs per instance (multi-homed: e.g., mgmt network + app network)
- Each ENI can have multiple private IPs and one Elastic IP

---

## Common exam pitfalls

- **Security Group = stateful** — no return rule needed; **NACL = stateless** — needs both an inbound AND an outbound rule
- **VPC Peering is not transitive** — each pair needs its own peering; for many VPCs, use TGW
- **NAT Gateway per AZ** for HA — if all private subnets use a single NAT GW in one AZ, there is a single point of failure
- **Gateway Endpoint** (S3, DynamoDB) = free; **Interface Endpoint** = billed per hour
- **Direct Connect + VPN** = dual layer of redundancy (DX primary + VPN fallback)
- Interface Endpoints are accessible from on-prem via DX/VPN; Gateway Endpoints are not
- **TGW** supports transitivity — recommended when you have many VPCs (>5) to connect
- **Egress-only IGW** = NAT for IPv6 (outbound only, no public inbound)
- **VPC Flow Logs** does not capture packet contents — only metadata (IP, port, action)
- Flow Logs with REJECT = SG or NACL blocked it; Flow Logs with ACCEPT but a connectivity problem = application or OS problem

## Hands-on lab

For guided networking practice, use [09-VPC-e-Redes-Labs/lab.md](../09-VPC-e-Redes-Labs/lab.md).
Cost notes: prefer short test windows for NAT Gateway and endpoints, validate routes with few resources, and fully tear down when finished.

## Practice questions

- [questoes.md](./questoes.md)

## Quick review / cheatsheet

- [cheatsheet.md](./cheatsheet.md)
- [flashcards.md](./flashcards.md)
