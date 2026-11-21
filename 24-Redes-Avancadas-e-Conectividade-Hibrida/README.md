# 24 Advanced Networking and Hybrid Connectivity

## Module objectives

- master connectivity between VPCs, accounts, and the on-premises datacenter
- differentiate VPC Peering, Transit Gateway, PrivateLink, VPN, and Direct Connect
- design networks with resilience, security, and low latency
- solve exam questions focused on routing, isolation, and transfer cost

## Core concepts

In the SAA-C03, advanced networking appears when the scenario involves multiple VPCs, multiple accounts, hybrid connectivity, and private services.

## Key components

- VPC Peering: connects two VPCs directly; does not support transitive routing.
- AWS Transit Gateway (TGW): hub-and-spoke for many VPCs and VPNs.
- AWS PrivateLink: expose/consume a private service without full peering.
- Site-to-Site VPN: IPSec tunnel over the internet.
- AWS Direct Connect (DX): dedicated link with low variability to on-premises.
- Route 53 Resolver endpoints: hybrid DNS between AWS and on-premises.

## When to use each

- few VPCs and a simple topology: VPC Peering
- many VPCs/accounts with central routing: TGW
- publish an internal service to other accounts with a smaller attack surface: PrivateLink
- quick connectivity without a dedicated circuit: VPN
- stable high-throughput connectivity: Direct Connect (with backup VPN)

## Exam tips

- peering is not transitive; TGW solves that.
- if the question asks for access to a private service without opening the full CIDR, think PrivateLink.
- DX often appears with a more predictable latency requirement and high traffic.
- for hybrid high availability, combine DX + backup VPN.
- Security Group is stateful; NACL is stateless (recurring pitfall).

## Related links

- [Cheatsheet](./cheatsheet.md)
- [Use cases](./casos-de-uso.md)
- [Questions](./questoes.md)
- [Flashcards](./flashcards.md)
- [Lab](./lab.md)
- [Official links](./links.md)
