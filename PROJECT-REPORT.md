# BGP Lab Project Report

## Repository

This is the dedicated repository for the Multi-AS BGP Lab:

`emadalshishani/BGP-LAB`

## Scope

The lab consists of six autonomous systems:

- AS100 — HQ
- AS200 — Branch
- AS1000 — Enterprise transit/core
- AS2000 — Enterprise transit/core
- AS3000 — ISP1
- AS4000 — ISP2

The environment combines Juniper and Cisco platforms and uses eBGP, iBGP, IS-IS, and OSPF.

## Configuration Set

### AS1000
- R1-PE1
- R3-P1
- R4-P2
- R5-IGR1
- R6-IGR2

### AS2000
- R1-PE1-AS2000
- R2-P1-AS2000
- R3-P2-AS2000
- R4-IGR1-AS2000
- R5-IGR2-AS2000

### AS3000
- ISP1

### AS4000
- ISP2

## Documentation Added

### Design
- `docs/architecture.md`
- `docs/addressing.md`
- `docs/routing-design.md`
- `docs/lab-status.md`

### Verification
- `testing/bgp-verification.md`

### Failure Testing
- `testing/failover-testing.md`

## Sanitization

Encrypted Junos authentication material was removed from the published configuration copies.

The retired AS1000 R2 references were removed from the published AS1000 configurations, including the stale `2.2.2.2` iBGP references and R2-specific link references.

The AS2000 device named `R2-P1-AS2000` remains because it is a separate device belonging to AS2000.

## Verified Tests

### R5 ↔ ISP1 Failure
Verified eBGP failure detection, alternate route selection, and recovery.

### R6 ↔ ISP1 Failure
Verified eBGP failure detection, alternate route selection, and recovery.

### Full ISP2 Failure
Verified actual end-to-end traffic movement from ISP2 / AS4000 to ISP1 / AS3000.

The test used continuous ICMP from HQ PC2 (`192.168.10.3`) to `8.8.8.8` and a pre/post traceroute comparison.

Observed packet loss during convergence:
**42 consecutive ICMP packets.**

## Evidence Boundary

Only performed and captured tests are documented as completed.

The reverse full-ISP1 end-to-end failure test is not documented as completed because no corresponding execution evidence is present.

No production-performance or hitless-failover claim is made.
