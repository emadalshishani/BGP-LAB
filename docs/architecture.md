# Multi-AS BGP Lab Architecture

## Objective

The lab models an enterprise environment connected through multiple routing domains and two upstream ISP domains. The design is intended to demonstrate routing behavior, inter-AS connectivity, multi-vendor interoperability, and failure convergence.

## High-Level Topology

```text
                    +---------------- AS3000 / ISP1 ----------------+
                    |                                                |
                 AS1000                                             AS2000
               Enterprise                                          Enterprise
                 Transit                                             Transit
                 /   \                                               /   \
                /     \                                             /     \
            AS3000   AS4000                                       AS3000   AS4000
                \     /                                             \     /
                 \   /                                               \   /
                    upstream lab Internet / 10.142.13.1

       AS100 HQ ---------------- AS1000

       AS200 Branch ------------ AS2000
```

The diagram above is a logical representation of the relationships documented from the device configurations. The detailed physical topology is represented by the lab itself.

## Autonomous Systems

### AS100 — HQ

Juniper SRX edge. The HQ edge forms an eBGP session with AS1000 and provides the enterprise-originated HQ prefix used in route propagation tests.

### AS200 — Branch

Juniper SRX edge. The Branch edge forms an eBGP session with AS2000 and participates in internal OSPF toward the Branch routing domain.

### AS1000

Juniper-based enterprise transit/core domain.

Active published device set:
- R1-PE1
- R3-P1
- R4-P2
- R5-IGR1
- R6-IGR2

The published configuration uses IS-IS as the IGP and iBGP for internal route distribution.

R5 and R6 provide the dual external connections to both ISP domains. R1 provides the external connection toward HQ.

### AS2000

Cisco IOS enterprise transit/core domain.

The published device set contains:
- R1-PE1-AS2000
- R2-P1-AS2000
- R3-P2-AS2000
- R4-IGR1-AS2000
- R5-IGR2-AS2000

OSPF is used as the IGP and iBGP distributes BGP routes internally.

R1 provides the Branch-facing eBGP connection. R4 and R5 provide connectivity toward the two ISP domains.

### AS3000 — ISP1

Cisco IOS ISP domain with five eBGP relationships:
- two peers in AS1000
- two peers in AS2000
- one peer in AS4000

The ISP1 router also has a static default toward 10.142.13.1.

### AS4000 — ISP2

Cisco IOS ISP domain with five eBGP relationships:
- two peers in AS1000
- two peers in AS2000
- one peer in AS3000

The ISP2 router also has a static default toward 10.142.13.1.

## Design Characteristics

The lab demonstrates:
- multiple autonomous systems
- eBGP and iBGP
- IS-IS and OSPF as internal routing protocols
- dual ISP connectivity
- route propagation across several AS boundaries
- multi-vendor Juniper/Cisco interoperability
- control-plane and data-plane failure testing

## Publication Notes

The published configuration set is intentionally sanitized. Authentication secrets are not published.

The retired AS1000 R2 node is not part of the active documented topology. Stale AS1000 references to 2.2.2.2 and the R2-specific 10.2.3.0/30 and 10.2.4.0/30 links were removed from the published AS1000 configuration copies.
