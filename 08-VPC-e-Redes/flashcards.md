# Flashcards — Module 07: VPC and Networking

---

**Q:** How many IPs does AWS reserve in each subnet, and which are they?
**A:** 5 IPs: .0 (network address), .1 (VPC router), .2 (DNS — Amazon DNS), .3 (reserved for future use), .255 (broadcast — not used but reserved)

---

**Q:** Are Security Groups stateful or stateless? And NACLs?
**A:** Security Groups: **stateful** (response traffic for allowed traffic flows in/out automatically). NACLs: **stateless** (need explicit rules for both inbound AND outbound)

---

**Q:** What is the maximum CIDR block size that can be assigned to a VPC?
**A:** /16 (65,536 addresses). Minimum: /28 (16 addresses, but only 11 usable). A VPC can have up to 5 associated CIDR blocks

---

**Q:** What is an Internet Gateway (IGW) and what is needed for an instance to be reachable from the internet?
**A:** An IGW is a horizontally scaled, redundant component that connects the VPC to the internet. For an instance to be reachable: 1) subnet with a 0.0.0.0/0 → IGW route; 2) instance with a public IP/EIP; 3) SG allowing inbound traffic

---

**Q:** What is the difference between NAT Gateway and NAT Instance?
**A:** NAT Gateway: managed by AWS, highly available within the AZ, 45 Gbps, no management. NAT Instance: an EC2 you manage, can also be used as a bastion, cheaper for low traffic, you manage HA

---

**Q:** For HA with NAT Gateway, how many gateways should you create?
**A:** One NAT Gateway **per AZ**. A NAT GW is zonal — if the AZ fails, instances in other AZs that depend on that NAT GW lose connectivity. Each AZ's private subnet should have a route to its own AZ's NAT GW

---

**Q:** Is VPC Peering transitive? How do you achieve transitive communication?
**A:** Peering is **non-transitive**. A↔B and B↔C does not allow A↔C via B. For transitive communication: use **Transit Gateway** (hub-and-spoke, each VPC has 1 attachment to the TGW, and the TGW routes between all of them)

---

**Q:** What is the difference between Gateway Endpoint and Interface Endpoint (PrivateLink)?
**A:** Gateway Endpoint: free, S3 and DynamoDB only, added as a route in the route table, not accessible from on-prem. Interface Endpoint: billed ($/hour + $/GB), creates an ENI with a private IP, supports all services, accessible from on-prem

---

**Q:** What are the 3 types of Virtual Interface (VIF) in Direct Connect?
**A:** Private VIF: private resources in a VPC. Public VIF: AWS public endpoints (S3, EC2 public). Transit VIF: connects to Transit Gateway for multiple VPCs. (Hosted VIF: Virtual Interface with capacity ≤ 1 Gbps shared via a partner)

---

**Q:** What is a Link Aggregation Group (LAG) in Direct Connect?
**A:** It groups multiple physical DX connections (1 Gbps or 10 Gbps) as a single logical link, increasing bandwidth and redundancy (automatic failover between the group's connections). All connections must be at the same DX location and the same speed

---

**Q:** What does VPC Flow Logs capture? And what does it NOT capture?
**A:** Captures: source/dest IP, port, protocol, bytes, packets, action (ACCEPT/REJECT), duration. Does NOT capture: payload (contents), DHCP traffic, DNS traffic to the Route 53 Resolver, instance traffic to IMDS (169.254.169.254)

---

**Q:** What is the function of the Egress-Only Internet Gateway?
**A:** It allows IPv6 instances to initiate connections to the internet (outbound) but blocks connections initiated from the internet (inbound). It is the equivalent of NAT Gateway but for IPv6. IPv6 addresses are public by nature, so the Egress-Only IGW provides inbound privacy

---

**Q:** What is AWS PrivateLink?
**A:** Technology that lets you expose services (SaaS, internal) via VPC Endpoint (Interface Endpoint) to other VPCs without peering and without traffic over the internet. The provider creates an NLB; the consumer creates an Interface Endpoint. There is no CIDR overlap restriction

---

**Q:** What is the difference between Site-to-Site VPN and Direct Connect?
**A:** Site-to-Site VPN: encrypted over the public internet (IPSec), minutes to provision, variable throughput (~1.25 Gbps per tunnel). Direct Connect: dedicated physical link, weeks to provision, consistent latency, 1/10/100 Gbps throughput

---

**Q:** What is Accelerated Site-to-Site VPN?
**A:** A Site-to-Site VPN that uses AWS Global Accelerator: VPN traffic goes to the nearest AWS edge location over the AWS private backbone (not the public internet). It reduces latency and improves VPN stability

---

**Q:** In a VPC, what is a secondary CIDR and why add one?
**A:** VPCs can have up to 5 associated CIDRs (1 primary + 4 secondary). Useful when the original IP space is exhausted without needing to recreate the VPC. Limitations: it must not overlap with CIDRs of peered VPCs or on-prem

---

**Q:** Which networking components isolate security boundaries within a VPC?
**A:** The combination of: **Subnets** (L3 segmentation), **Security Groups** (stateful firewall on the ENI), **NACLs** (stateless firewall on the subnet), and **Route Tables** (traffic control). Public/private subnets separate exposed resources from internal ones

---

**Q:** What is the maximum size of a VPC CIDR block?
**A:** /16 (65,536 addresses). Prefer using CIDRs from the RFC 1918 ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) to avoid conflicts with the company's other CIDRs

---

**Q:** What is the Transit Gateway Route Table?
**A:** Each TGW has one or more route tables that control how traffic is distributed among attachments. It enables isolation: e.g., production VPCs communicate only with each other (separate route table) and not with development VPCs

---

**Q:** What happens if two VPCs with overlapping CIDRs try to peer?
**A:** The peering fails — VPC Peering requires that the two VPCs' CIDRs **do not overlap**. An identical or partially overlapping CIDR prevents the peering from being created. Solution: re-address one of the VPCs or use AWS PrivateLink (which does not require routability between CIDRs)
