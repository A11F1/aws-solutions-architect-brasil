# Lab - TGW + VPN (guided, conceptual)

## Objective
Set up centralized connectivity between VPCs and prepare hybrid connectivity.

## Steps

1. Create two VPCs (App and Shared).
2. Create a Transit Gateway.
3. Attach both VPCs to the TGW.
4. Adjust the subnet route tables for inter-VPC traffic via the TGW.
5. Validate private communication between instances.
6. (Optional) Create a simulated VPN connection with a Customer Gateway to study the parameters.
7. Review connectivity metrics and logs.

## Validation

- communication between VPCs via TGW routes
- isolation via route tables when necessary
- understanding of the VPN failover flow

## Cleanup

- remove attachments and the TGW
- remove VPCs and auxiliary resources
