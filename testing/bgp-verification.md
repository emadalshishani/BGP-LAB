# BGP Verification

## Verification Scope

The recorded verification output was reviewed across AS100, AS200, AS1000, AS2000, AS3000, and AS4000.

The purpose was to confirm:
- BGP session establishment
- route exchange
- inter-AS path visibility
- default-route learning
- alternate path visibility

## AS100 — HQ

BGP summary showed the HQ SRX with one eBGP peer:

- Local AS: 100
- Peer: 10.10.30.2
- Peer AS: 1000
- State: Established
- Table entry counters: 10 active / 11 received / 11 accepted

The HQ routing table showed external prefixes learned through AS1000, including paths toward AS3000, AS4000, and AS2000.

## AS200 — Branch

BGP summary showed:

- Local AS: 200
- Peer: 10.10.40.2
- Peer AS: 2000
- State: Established
- Table entry counters: 12 active / 13 received / 13 accepted

The Branch routing table showed external paths through AS2000 toward the ISP domains and HQ.

## AS1000

The active published topology contains five Juniper routers:

- R1-PE1
- R3-P1
- R4-P2
- R5-IGR1
- R6-IGR2

The configuration uses iBGP internally and eBGP toward:
- HQ / AS100
- ISP1 / AS3000
- ISP2 / AS4000

R5 and R6 each have direct eBGP connectivity to both ISP1 and ISP2.

The verification output showed established sessions and routes learned through both ISP domains.

## AS2000

The configuration contains five Cisco routers:

- R1-PE1-AS2000
- R2-P1-AS2000
- R3-P2-AS2000
- R4-IGR1-AS2000
- R5-IGR2-AS2000

OSPF provides internal reachability and iBGP distributes BGP routes.

The verification output showed established eBGP sessions toward Branch and the ISP domains, along with iBGP route distribution.

## AS3000 — ISP1

Cisco IOS 15.7.

BGP summary showed five established eBGP relationships:
- AS1000 via 10.5.10.2
- AS1000 via 10.6.10.2
- AS2000 via 20.4.10.2
- AS2000 via 20.5.10.2
- AS4000 via 30.10.20.2

The router originates/maintains the lab default path through its upstream connection.

## AS4000 — ISP2

Cisco IOS 15.7.

BGP summary showed five established eBGP relationships:
- AS1000 via 10.5.20.2
- AS1000 via 10.6.20.2
- AS2000 via 20.4.20.2
- AS2000 via 20.5.20.2
- AS3000 via 30.10.20.1

The router also has the lab upstream/Internet connection.

## Verification Result

The captured outputs demonstrate active BGP interconnection across the six-AS topology and route visibility across the enterprise and ISP domains.
