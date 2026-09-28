# Addressing and BGP Adjacencies

This document records the addressing and adjacencies explicitly present in the published lab configuration.

## AS1000 Internal Links

| Link | Prefix | Endpoints |
|---|---|---|
| R1–R3 | 10.1.3.0/30 | R1 10.1.3.1 / R3 10.1.3.2 |
| R1–R4 | 10.1.4.0/30 | R1 10.1.4.1 / R4 10.1.4.2 |
| R3–R5 | 10.3.5.0/30 | R3 10.3.5.1 / R5 10.3.5.2 |
| R3–R4 | 10.3.4.0/30 | R3 10.3.4.1 / R4 10.3.4.2 |
| R4–R6 | 10.4.6.0/30 | R4 10.4.6.1 / R6 10.4.6.2 |
| R5–R6 | 10.5.6.0/30 | R5 10.5.6.1 / R6 10.5.6.2 |

## AS1000 External Links

| Device | Peer Domain | Local | Peer |
|---|---|---:|---:|
| R1-PE1 | HQ / AS100 | 10.10.30.2 | 10.10.30.1 |
| R5-IGR1 | ISP1 / AS3000 | 10.5.10.2 | 10.5.10.1 |
| R5-IGR1 | ISP2 / AS4000 | 10.5.20.2 | 10.5.20.1 |
| R6-IGR2 | ISP1 / AS3000 | 10.6.10.2 | 10.6.10.1 |
| R6-IGR2 | ISP2 / AS4000 | 10.6.20.2 | 10.6.20.1 |

## AS2000 External Links

| Device | Peer Domain | Local | Peer |
|---|---|---:|---:|
| R1-PE1-AS2000 | Branch / AS200 | 10.10.40.2 | 10.10.40.1 |
| R4-IGR1-AS2000 | ISP1 / AS3000 | 20.4.10.2 | 20.4.10.1 |
| R4-IGR1-AS2000 | ISP2 / AS4000 | 20.4.20.2 | 20.4.20.1 |
| R5-IGR2-AS2000 | ISP2 / AS4000 | 20.5.20.2 | 20.5.20.1 |
| R5-IGR2-AS2000 | ISP1 / AS3000 | 20.5.10.2 | 20.5.10.1 |

## ISP1 / ISP2 Interconnection

| Relationship | Prefix | Endpoints |
|---|---|---|
| AS3000 ↔ AS4000 | 30.10.20.0/30 | ISP1 30.10.20.1 / ISP2 30.10.20.2 |

## ISP Upstream

Both ISP domains use the lab upstream/Internet gateway:

`10.142.13.1`

## Loopbacks

Selected routing loopbacks:
- AS1000: R1 1.1.1.1/32, R3 3.3.3.3/32, R4 4.4.4.4/32, R5 5.5.5.5/32, R6 6.6.6.6/32
- AS2000: R1 100.1.1.1/32, R2 100.1.1.2/32, R3 100.1.1.3/32, R4 100.1.1.4/32, R5 100.1.1.5/32

The 2.2.2.2 loopback and the associated AS1000 R2 links are excluded from the published topology because that AS1000 R2 node was retired from the lab.
