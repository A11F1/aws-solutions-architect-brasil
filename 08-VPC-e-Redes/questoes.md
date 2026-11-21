# Exam Questions — Module 07: VPC and Networking

<!-- SAA-C03 Domain: Design Secure Architectures / Design Resilient Architectures -->

---

**1.** A company has VPCs in 10 AWS Regions and needs all of them to communicate with each other. What is the most scalable solution?

- A) VPC Peering between all pairs (N*(N-1)/2 connections)
- B) Transit Gateway with inter-region peering
- C) Site-to-Site VPN connecting all VPCs
- D) Direct Connect with a hosted connection for each VPC

<details>
<summary>Answer</summary>
**B — Transit Gateway**
With 10 VPCs, VPC Peering would require 45 connections (10×9/2) and is **not transitive** (each VPC needs a direct connection to every other one). Transit Gateway centralizes routing in a hub-and-spoke model: each VPC has only 1 attachment to the TGW, and the TGW is transitive. TGW Peering allows connecting TGWs across Regions.
</details>

---

**2.** What is the fundamental difference between Security Groups and Network ACLs (NACLs)?

- A) Security Groups operate at the subnet; NACLs operate at the instance
- B) Security Groups are stateful; NACLs are stateless
- C) NACLs allow Allow rules; Security Groups only do Deny
- D) Security Groups process all rules; NACLs stop at the first match

<details>
<summary>Answer</summary>
**B — Security Groups are stateful; NACLs are stateless**
Stateful SG: if inbound traffic is allowed, the outbound response is automatically allowed (no outbound rule needed). Stateless NACL: inbound and outbound traffic are evaluated independently — you need explicit rules for both directions. SGs operate at the **instance/ENI**; NACLs operate at the **subnet**.
</details>

---

**3.** An EC2 instance in a private subnet needs to download software updates from the internet. Which component is required?

- A) Internet Gateway attached to the private subnet
- B) NAT Gateway in the public subnet with a route in the private subnet
- C) Egress-Only Internet Gateway for IPv4
- D) VPC Endpoint for the software repository

<details>
<summary>Answer</summary>
**B — NAT Gateway in the public subnet**
A NAT Gateway lets instances in a private subnet initiate connections to the internet (outbound) but blocks connections initiated from the internet (inbound). It must be created in the **public** subnet (with an IGW), and the private subnet's 0.0.0.0/0 route points to the NAT GW. Egress-Only IGW is for **IPv6** only.
</details>

---

**4.** A company wants EC2 instances in a VPC to access Amazon S3 without traffic going over the internet. What is the simplest and most cost-effective solution?

- A) VPC Interface Endpoint for S3
- B) NAT Gateway + route to S3
- C) VPC Gateway Endpoint for S3
- D) AWS PrivateLink for S3

<details>
<summary>Answer</summary>
**C — VPC Gateway Endpoint for S3**
Gateway Endpoints for S3 and DynamoDB are **free** and are added as an entry in the route table (they do not use an ENI). Interface Endpoints (PrivateLink) cost per hour + per GB. For S3, the Gateway Endpoint is the simplest and most cost-effective solution. An S3 Interface Endpoint is only needed when you require access from on-prem via DX/VPN.
</details>

---

**5.** VPC Peering does not allow transitive traffic. What does this mean in practice?

- A) VPC A peers with B, and B with C: A cannot communicate with C via B
- B) Traffic over the peering uses asymmetric encryption
- C) Peering does not work between different Regions
- D) Peering does not allow UDP traffic, only TCP

<details>
<summary>Answer</summary>
**A — Correct**
Non-transitive: A↔B and B↔C exist, but A cannot reach C through B. For A↔C, you need a direct A↔C peering. For transitive routing, use **Transit Gateway**. Peering can be cross-region (inter-region peering). There is no protocol restriction (TCP, UDP, ICMP all work).
</details>

---

**6.** A company wants to connect its data center to AWS with 10 Gbps of dedicated bandwidth and consistently low latency. Which service?

- A) Site-to-Site VPN over the public internet
- B) AWS Direct Connect (Dedicated Connection)
- C) AWS Direct Connect (Hosted Connection)
- D) Accelerated VPN with Global Accelerator

<details>
<summary>Answer</summary>
**B — Direct Connect Dedicated Connection**
Dedicated Connection: 1 Gbps, 10 Gbps, or 100 Gbps over fiber connected directly to the AWS DX location. High capacity, consistently low latency, no public internet. Hosted Connection (via a partner) offers smaller capacities (50 Mbps to 10 Gbps via a partner). VPN uses the public internet — variable latency and lower throughput.
</details>

---

**7.** What is the difference between Public VIF, Private VIF, and Transit VIF in Direct Connect?

- A) Public VIF → AWS public endpoints; Private VIF → VPC via VGW; Transit VIF → multiple VPCs via TGW
- B) Private VIF is encrypted; the others are not
- C) Transit VIF allows internet access; the others do not
- D) All are equivalent, just different names for billing

<details>
<summary>Answer</summary>
**A — Correct**
- **Private VIF**: accesses private resources in a VPC (via Virtual Private Gateway or Direct Connect Gateway)
- **Public VIF**: accesses AWS public endpoints (S3, EC2 public IPs, API endpoints) without the internet
- **Transit VIF**: connects to the AWS Transit Gateway to access multiple VPCs in one or more Regions
No VIF is encrypted by default — add an IPSec VPN over DX for encryption.
</details>

---

**8.** A company needs high availability for Direct Connect. In which configuration is failover automatic?

- A) A single DX connection with a Link Aggregation Group (LAG)
- B) Two DX connections at two different DX Locations + Site-to-Site VPN as backup
- C) DX Hosted Connection with multiple VIFs
- D) DX with CloudFront as a backup CDN

<details>
<summary>Answer</summary>
**B — Two DX Locations + VPN Backup**
For maximum resilience: 2 physical connections at **2 different DX locations** (protection against physical failure). Site-to-Site VPN as a backup for DX failure (automatic failover via BGP). LAG groups multiple connections at the same location — it protects against port/fiber failure but not location failure.
</details>

---

**9.** An on-premises server needs to access internal services in the VPC via an Interface Endpoint (PrivateLink). What needs to be enabled?

- A) Nothing — Interface Endpoints are accessible from on-prem by default
- B) Private DNS resolution and a connection via Direct Connect/VPN
- C) Gateway Endpoint with BGP routing
- D) VPC Peering between on-prem and the VPC

<details>
<summary>Answer</summary>
**B — Private DNS + DX or VPN**
Interface Endpoints with Private DNS enabled resolve the service name to the private IP of the endpoint's ENI. On-premises can access it via Direct Connect or VPN. Gateway Endpoints (S3/DynamoDB) are NOT accessible from on-prem — only Interface Endpoints are. VPC Peering does not connect on-prem.
</details>

---

**10.** What does VPC Flow Logs capture?

- A) Packet contents (payload) of all connections
- B) IP traffic metadata (address, port, protocol, bytes, ACCEPT/REJECT)
- C) Logs of DNS queries made inside the VPC
- D) CloudTrail API calls for resources inside the VPC

<details>
<summary>Answer</summary>
**B — IP traffic metadata**
VPC Flow Logs captures metadata: source/destination IP, port, protocol, bytes, packets, duration, action (ACCEPT/REJECT). It **does not capture the payload** (packet contents). It can be sent to CloudWatch Logs or S3. DNS queries are captured by Route 53 Resolver DNS Logs. CloudTrail covers API calls, not network traffic.
</details>

---

**11.** An EC2 instance in your VPC needs to access a partner's SaaS service hosted in a different VPC (different account), without exposing the service to the internet. Which solution?

- A) VPC Peering between the accounts
- B) AWS PrivateLink (Interface Endpoint for the partner's service)
- C) Transit Gateway with RAM (Resource Access Manager) across accounts
- D) Site-to-Site VPN between the two VPCs

<details>
<summary>Answer</summary>
**B — AWS PrivateLink**
PrivateLink lets a provider expose its service via an NLB, and consumers create Interface Endpoints in their own VPC to access it. Traffic does not go over the internet and there is no CIDR overlap risk (unlike VPC Peering). Ideal for multi-tenant SaaS services across accounts.
</details>

---

**12.** A company wants to allow bidirectional IPv6 communication between instances in the VPC and the internet. Which component needs to be added?

- A) NAT Gateway with IPv6 enabled
- B) Internet Gateway (IGW) — already supports IPv6 bidirectionally
- C) Egress-Only Internet Gateway only
- D) NAT64 Gateway

<details>
<summary>Answer</summary>
**B — Internet Gateway**
The IGW natively supports IPv6 and allows **bidirectional traffic** (inbound and outbound from the internet for IPv6). Egress-Only IGW is for IPv6 **outbound only** (like NAT for IPv6 — blocking inbound from the internet). There is no NAT Gateway for IPv6 (NAT is not needed because IPv6 has abundant address space).
</details>

---

**13.** What is the maximum number of available IPs in a /28 subnet on AWS?

- A) 16
- B) 14
- C) 11
- D) 13

<details>
<summary>Answer</summary>
**C — 11**
A /28 subnet = 16 total addresses. AWS reserves **5 IPs** per subnet: .0 (network), .1 (VPC router), .2 (DNS), .3 (future), .255 (broadcast). 16 - 5 = **11 usable IPs**.
</details>

---

**14.** A company uses Site-to-Site VPN to connect on-prem to AWS. It needs very low latency and maximum VPN throughput. Which feature should be enabled?

- A) BGP dynamic routing
- B) Accelerated VPN with Global Accelerator
- C) VPN with Dead Peer Detection (DPD)
- D) Multiple VPN tunnels in a LAG

<details>
<summary>Answer</summary>
**B — Accelerated VPN**
Accelerated Site-to-Site VPN uses AWS Global Accelerator to route VPN traffic through AWS edge locations (the AWS backbone network) to the VGW, reducing latency by up to 60% vs. the public internet. DPD detects peer failure, not performance. LAG does not apply to VPN tunnels.
</details>

---

**15.** An organization created 3 isolated VPCs: Dev, Staging, and Prod. They need to communicate with a shared services VPC (DNS, monitoring, etc.) but NOT with each other. What is the simplest architecture?

- A) Transit Gateway with separate route tables blocking communication between Dev/Staging/Prod
- B) 3 VPC Peerings: Dev↔Shared, Staging↔Shared, Prod↔Shared
- C) Site-to-Site VPN between each VPC and Shared
- D) Interface Endpoints in the Dev/Staging/Prod VPCs for Shared services

<details>
<summary>Answer</summary>
**B — Direct VPC Peerings with Shared**
3 direct peerings is simple for this case: Dev↔Shared, Staging↔Shared, Prod↔Shared. Since peering is not transitive, Dev, Staging, and Prod are automatically isolated from each other (there is no Dev→Prod route via Shared). Transit Gateway would work but is more complex and costs more for just 4 VPCs with simple routing.
</details>
