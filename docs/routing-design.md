# Routing Design

## BGP Roles

The lab separates routing responsibilities by autonomous system.

### AS100 — HQ
eBGP is used between the HQ SRX and AS1000.

### AS200 — Branch
eBGP is used between the Branch SRX and AS2000.

### AS1000
- IS-IS provides the internal IP reachability used by the Juniper core.
- iBGP distributes BGP routes across the AS.
- R1 peers externally with HQ.
- R5 and R6 peer externally with both ISPs.
- R5 is configured with NEXT-HOP-SELF for its internal BGP export.

### AS2000
- OSPF provides internal IP reachability.
- iBGP distributes routes across the AS.
- R1 peers externally with Branch.
- R4 and R5 peer externally with ISP1 and ISP2 according to the device configurations.

### AS3000 / AS4000
Both ISP domains use eBGP to exchange routes with the enterprise domains and with each other.

Both have a static default route toward 10.142.13.1.

## Route Selection and Redundancy

The design provides multiple paths to external destinations.

For the default route, the documented baseline behavior shows a preferred active path through ISP2 / AS4000 and an alternate path through ISP1 / AS3000 on the affected enterprise routers.

When an ISP-facing eBGP link is withdrawn, the affected router can use:
- another directly connected ISP path, or
- an alternate path learned through the internal BGP topology and the remaining ISP domain.

## Multi-Vendor Behavior

The lab mixes:
- Juniper Junos for AS100, AS1000 and AS200
- Cisco IOS for AS2000, AS3000 and AS4000

The verification outputs demonstrate that BGP sessions are established across the Juniper/Cisco boundaries.

## Internal Reachability vs BGP Reachability

The IGPs are used to make loopback and internal next-hop addresses reachable inside each AS. BGP then carries inter-domain prefixes.

This separation is visible in the design:
- IS-IS inside AS1000
- OSPF inside AS2000
- eBGP at AS boundaries
- iBGP inside the BGP-speaking transit domains

## Default Route Propagation

The ISP domains originate a default route toward their connected enterprise peers in the lab. The enterprise routers then have visibility of multiple external paths.

The end-to-end ISP failure test confirms that a complete ISP2 failure can cause the data plane to move from the AS4000 path to the AS3000 path after BGP convergence.
