# Cheatsheet - Advanced Networking and Hybrid Connectivity

## Quick choice

| Scenario | Recommended solution |
|---|---|
| 2 VPCs with simple traffic exchange | VPC Peering |
| dozens of VPCs and accounts | Transit Gateway |
| publish an internal service to several accounts | PrivateLink |
| connect a branch office quickly | Site-to-Site VPN |
| high throughput and predictable latency | Direct Connect |

## Important rules

- VPC Peering: no transitive routing
- TGW: central routing with separate route tables
- PrivateLink: connects the consumer to the private service endpoint
- DX: use a redundant link and VPN as contingency
- Resolver inbound/outbound endpoints: hybrid DNS

## Exam pitfalls

- using peering in a large mesh (does not scale)
- forgetting overlapping CIDRs (blocks peering/TGW)
- assuming a Security Group filters internet traffic of a public subnet without the correct route
- confusing NACL stateful (it is actually stateless)

## Trigger phrases

- "multiple VPCs across accounts" => Transit Gateway
- "privately access service" => PrivateLink
- "consistent network performance" => Direct Connect
- "encrypted connection over internet" => VPN
