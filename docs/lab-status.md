# Lab Status

## Current Completed Work

### Topology
- Six autonomous systems represented.
- Juniper and Cisco platforms integrated.
- HQ and Branch connected through separate enterprise transit domains.
- Dual ISP domains included.

### Routing
- eBGP configured between the main AS boundaries.
- iBGP configured inside AS1000 and AS2000.
- IS-IS configured inside AS1000.
- OSPF configured inside AS2000.
- ISP1 and ISP2 interconnected with eBGP.

### Verification
- BGP neighbor state captured for the active topology.
- Route tables captured across the AS domains.
- External prefix propagation verified in the recorded outputs.

### Failure Testing
- R5 ↔ ISP1 link failure tested.
- R6 ↔ ISP1 link failure tested.
- Full ISP2 failure tested end-to-end from HQ PC2 to 8.8.8.8.

## Important Evidence Boundaries

The documented tests distinguish control-plane convergence from actual traffic continuity.

R5 and R6 single-link tests verify BGP failure detection, route changes, and recovery.

The ISP2 full-failure test verifies actual traffic-path movement using continuous ICMP and traceroute.

No claim is made that the failure tests are hitless.

## Not Yet Documented / Not Claimed

- Reverse end-to-end ISP1 full-failure test
- Additional convergence-time measurements beyond the observed packet-loss window
- Any production performance result

These items can be added after they are actually executed and captured.
