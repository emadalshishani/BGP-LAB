# BGP Attribute — MED (Policy-Option Export & Metric-Out)

## Objective

Verify the behavior of the BGP **Multi-Exit Discriminator (MED)** attribute in **AS1000** using two configuration approaches on **R5-IGR1**:

1. Direct BGP export configuration.
2. Policy-based MED configuration using `policy-options`.

The test was performed while continuous ICMP traffic was running from **HQ PC2 (192.168.10.3)** toward **8.8.8.8**.

A second phase was deliberately performed **without removing the configuration from the first phase**, in order to observe what happens when both configuration methods are present.

---

## Test Environment

- Source: HQ PC2
- Source IP: `192.168.10.3`
- Destination: `8.8.8.8`
- Device under test: **R5-IGR1**
- Autonomous System: **AS1000**
- Platform: **Juniper Junos**
- iBGP group: `iBGP`

The relevant AS1000 paths in this test were:

**R5 path**
```text
R1-PE1 → R3-P1 → R5-IGR1 → ISP2 / AS4000
```

**R6 path**
```text
R1-PE1 → R4-P2 → R6-IGR2 → ISP2 / AS4000
```

---

# Step 1 — Direct BGP MED Export

## Configuration Applied

The first test used direct BGP configuration:

```text
root@R5-IGR1# set protocols bgp group iBGP export MED
```

No policy-based MED configuration was added at this stage.

---

## Baseline — Before Step 1

Before applying the MED configuration, HQ PC2 was traced toward the Internet.

### HQ PC2 — Baseline Traceroute

```text
PC2> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   0.773 ms  0.580 ms  0.490 ms
 2   10.10.10.1   1.063 ms  0.843 ms  0.604 ms
 3   10.10.30.2   1.375 ms  1.209 ms  1.092 ms
 4   10.1.3.2   1.496 ms  1.474 ms  1.522 ms
 5   10.3.5.2   1.933 ms  1.878 ms  1.932 ms
 6   10.5.20.1   2.204 ms  2.069 ms  2.002 ms
 7   10.142.13.1   2.628 ms  2.504 ms  2.642 ms
 8   10.50.245.29   4.376 ms  4.023 ms  4.135 ms
```

Observed path:

```text
PC2
 ↓
HQ Gateway
 ↓
HQ-SRX
 ↓
R1-PE1
 ↓
R3-P1
 ↓
R5-IGR1
 ↓
ISP2
 ↓
10.142.13.1
 ↓
Internet
```

---

## Continuous Ping During Step 1

Continuous ICMP traffic was running while the MED configuration was applied.

```text
PC2> ping 8.8.8.8 -t
84 bytes from 8.8.8.8 icmp_seq=1 ttl=109 time=52.970 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=109 time=53.357 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=109 time=53.968 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=109 time=53.519 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=109 time=53.655 ms
84 bytes from 8.8.8.8 icmp_seq=6 ttl=109 time=53.021 ms
84 bytes from 8.8.8.8 icmp_seq=7 ttl=109 time=181.616 ms
84 bytes from 8.8.8.8 icmp_seq=8 ttl=109 time=54.382 ms
84 bytes from 8.8.8.8 icmp_seq=9 ttl=109 time=53.822 ms
84 bytes from 8.8.8.8 icmp_seq=10 ttl=109 time=53.187 ms
84 bytes from 8.8.8.8 icmp_seq=11 ttl=109 time=53.379 ms
84 bytes from 8.8.8.8 icmp_seq=12 ttl=109 time=55.065 ms
84 bytes from 8.8.8.8 icmp_seq=13 ttl=109 time=53.674 ms
84 bytes from 8.8.8.8 icmp_seq=14 ttl=109 time=55.654 ms
84 bytes from 8.8.8.8 icmp_seq=15 ttl=109 time=53.186 ms
84 bytes from 8.8.8.8 icmp_seq=16 ttl=109 time=54.853 ms
^C
```

No packet loss was observed in this captured interval.

A single transient increase in RTT was observed at sequence 7 (181.616 ms).

---

## R1-PE1 — BGP Route State Before Step 1

The default route was inspected on R1-PE1 using:

```text
root@R1-PE1# run show route protocol bgp extensive 0.0.0.0
```

The full captured output is preserved below.

```text
inet.0: 26 destinations, 33 routes (26 active, 0 holddown, 0 hidden)
0.0.0.0/0 (2 entries, 1 announced)
TSI: KRT in-kernel 0.0.0.0/0 -> {indirect(262142)}
Page 0 idx 0, (group eBGP type External) Type 1 val 0x99244a0 (adv_entry)
   Advertised metrics:
     Nexthop: Self
     AS path: [1000] 4000 I
     Communities:
    Advertise: 00000001
Path 0.0.0.0 from 5.5.5.5 Vector len 4.  Val: 0
        *BGP    Preference: 170/-101
                Next hop type: Indirect, Next hop index: 0
                Address: 0x78e59a4
                Next-hop reference count: 18
                Kernel Table Id: 0
                Source: 5.5.5.5
                Next hop type: Router, Next hop index: 571
                Next hop: 10.1.3.2 via ge-0/0/0.0, selected
                Session Id: 0
                Protocol next hop: 5.5.5.5
                Indirect next hop: 0x791dee8 262142 INH Session ID: 0
                State: <Active Int Ext>
                Local AS:  1000 Peer AS:  1000
                Age: 5:47
                Metric: 0       Metric2: 20
                Validation State: unverified
                ORR Generation-ID: 0
                Task: BGP_1000.5.5.5.5
                Announcement bits (3): 0-KRT 3-BGP_RT_Background 4-Resolve tree 4
                AS path: 4000 I
                Accepted
                Localpref: 100
                Router ID: 5.5.5.5
                Thread: junos-main
                Indirect next hops: 1
                        Protocol next hop: 5.5.5.5 Metric: 20 ResolvState: Resolved
                        Indirect next hop: 0x791dee8 262142 INH Session ID: 0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 10.1.3.2 via ge-0/0/0.0
                                Session Id: 0
                5.5.5.5/32 Originating RIB: inet.0
                                  Metric: 20 Node path count: 1
                                  Forwarding nexthops: 1
                                        Next hop type: Router
                                        Next hop: 10.1.3.2 via ge-0/0/0.0
                                        Session Id: 0
         BGP    Preference: 170/-101
                Next hop type: Indirect, Next hop index: 0
                Address: 0x78e5d24
                Next-hop reference count: 11
                Kernel Table Id: 0
                Source: 6.6.6.6
                Next hop type: Router, Next hop index: 570
                Next hop: 10.1.4.2 via ge-0/0/2.0, selected
                Session Id: 0
                Protocol next hop: 6.6.6.6
                Indirect next hop: 0x791e218 262143 INH Session ID: 0
                State: <NotBest Int Ext>
                Inactive reason: Not Best in its group - Router ID
                Local AS:  1000 Peer AS:  1000
                Age: 15:14:24   Metric: 0       Metric2: 20
                Validation State: unverified
                ORR Generation-ID: 0
                Task: BGP_1000.6.6.6.6
                AS path: 4000 I
                Accepted
                Localpref: 100
                Router ID: 6.6.6.6
                Thread: junos-main
                Indirect next hops: 1
                        Protocol next hop: 6.6.6.6 Metric: 20 ResolvState: Resolved
                        Indirect next hop: 0x791e218 262143 INH Session ID: 0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 10.1.4.2 via ge-0/0/2.0
                                Session Id: 0
                                6.6.6.6/32 Originating RIB: inet.0
                                  Metric: 20 Node path count: 1
                                  Forwarding nexthops: 1
                                        Next hop type: Router
                                        Next hop: 10.1.4.2 via ge-0/0/2.0
                                        Session Id: 0
```

### Baseline Route Observation

Before Step 1:

- R5 route: **Metric 0**, Local Preference 100, AS path `4000 I`
- R6 route: **Metric 0**, Local Preference 100, AS path `4000 I`
- R5 was the active route.
- R6 was inactive because Junos reported: **Not Best in its group - Router ID**.

---

# Step 1 Result — Direct MED Export

After applying:

```text
set protocols bgp group iBGP export MED
```

the route selection changed.

## R1-PE1 — BGP Route State After Step 1

```text
root@R1-PE1# run show route protocol bgp extensive 0.0.0.0
```

Captured output:

```text
inet.0: 26 destinations, 33 routes (26 active, 0 holddown, 0 hidden)
0.0.0.0/0 (2 entries, 1 announced)
TSI: KRT in-kernel 0.0.0.0/0 -> {indirect(262143)}
Page 0 idx 0, (group eBGP type External) Type 1 val 0x99244a0 (adv_entry)
   Advertised metrics:
     Nexthop: Self
     AS path: [1000] 4000 I
     Communities:
    Advertise: 00000001
Path 0.0.0.0 from 6.6.6.6 Vector len 4.  Val: 0
        *BGP    Preference: 170/-101
                Next hop type: Indirect, Next hop index: 0
                Address: 0x78e5d24
                Next-hop reference count: 18
                Kernel Table Id: 0
                Source: 6.6.6.6
                Next hop type: Router, Next hop index: 570
                Next hop: 10.1.4.2 via ge-0/0/2.0, selected
                Session Id: 0
                Protocol next hop: 6.6.6.6
                Indirect next hop: 0x791e218 262143 INH Session ID: 0
                State: <Active Int Ext>
                Local AS:  1000 Peer AS:  1000
                Age: 15:16:41   Metric: 0       Metric2: 20
                Validation State: unverified
                ORR Generation-ID: 0
                Task: BGP_1000.6.6.6.6
                Announcement bits (3): 0-KRT 3-BGP_RT_Background 4-Resolve tree 4
                AS path: 4000 I
                Accepted
                Localpref: 100
                Router ID: 6.6.6.6
                Thread: junos-main
                Indirect next hops: 1
                        Protocol next hop: 6.6.6.6 Metric: 20 ResolvState: Resolved
                        Indirect next hop: 0x791e218 262143 INH Session ID: 0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 10.1.4.2 via ge-0/0/2.0
                                Session Id: 0
                                6.6.6.6/32 Originating RIB: inet.0
                                  Metric: 20 Node path count: 1
                                  Forwarding nexthops: 1
                                        Next hop type: Router
                                        Next hop: 10.1.4.2 via ge-0/0/2.0
                                        Session Id: 0
         BGP    Preference: 170/-101
                Next hop type: Indirect, Next hop index: 0
                Address: 0x78e59a4
                Next-hop reference count: 11
                Kernel Table Id: 0
                Source: 5.5.5.5
                Next hop type: Router, Next hop index: 571
                Next hop: 10.1.3.2 via ge-0/0/0.0, selected
                Session Id: 0
                Protocol next hop: 5.5.5.5
                Indirect next hop: 0x791dee8 262142 INH Session ID: 0
                State: <NotBest Int Ext>
                Inactive reason: Not Best in its group - Route Metric or MED comparison
                Local AS:  1000 Peer AS:  1000
                Age: 36         Metric: 2       Metric2: 20
                Validation State: unverified
                ORR Generation-ID: 0
                Task: BGP_1000.5.5.5.5
                AS path: 4000 I
                Accepted
                Localpref: 100
                Router ID: 5.5.5.5
                Thread: junos-main
                Indirect next hops: 1
                        Protocol next hop: 5.5.5.5 Metric: 20 ResolvState: Resolved
                        Indirect next hop: 0x791dee8 262142 INH Session ID: 0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 10.1.3.2 via ge-0/0/0.0
                                Session Id: 0
                                5.5.5.5/32 Originating RIB: inet.0
                                  Metric: 20 Node path count: 1
                                  Forwarding nexthops: 1
                                        Next hop type: Router
                                        Next hop: 10.1.3.2 via ge-0/0/0.0
                                        Session Id: 0
```

### Step 1 Observation

The important change was:

```text
R5 route:
Metric 0  →  Metric 2

R6 route:
Metric 0  →  Metric 0
```

R1-PE1 then reported the R5 route as:

```text
State: <NotBest Int Ext>
Inactive reason: Not Best in its group - Route Metric or MED comparison
Metric: 2
```

while the R6 route remained active with:

```text
State: <Active Int Ext>
Metric: 0
```

This is the direct evidence that the observed route-selection decision changed from the R5 path to the R6 path after the direct MED export configuration.

---

## HQ PC2 — Traceroute After Step 1

```text
PC2> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   0.534 ms  0.380 ms  0.286 ms
 2   10.10.10.1   1.006 ms  0.537 ms  0.553 ms
 3   10.10.30.2   1.921 ms  1.386 ms  1.056 ms
 4   10.1.4.2   2.109 ms  1.342 ms  1.538 ms
 5   10.4.6.2   1.997 ms  1.805 ms  1.712 ms
 6   10.6.20.1   1.967 ms  2.078 ms  1.864 ms
 7   10.142.13.1   3.058 ms  2.422 ms  2.735 ms
 8   10.50.245.29   3.257 ms  4.158 ms  4.287 ms
```

Observed forwarding path after Step 1:

```text
PC2
 ↓
HQ Gateway
 ↓
HQ-SRX
 ↓
R1-PE1
 ↓
R4-P2
 ↓
R6-IGR2
 ↓
ISP2
 ↓
10.142.13.1
 ↓
Internet
```

---

## Step 1 Result

**PASS — Direct MED export path-selection behavior observed.**

The captured route output and traceroute show that after enabling:

```text
set protocols bgp group iBGP export MED
```

the R5-derived default route changed from **Metric 0 to Metric 2**, while the R6-derived route remained at **Metric 0**.

R6 therefore became the selected path in the observed lab state.

---

# Step 2 — Policy-Based MED with Step 1 Configuration Still Active

## Test Purpose

The second test was deliberately performed **with the Step 1 configuration still present**.

The additional configuration was:

```text
root@R5-IGR1#set policy-options policy-statement MED then metric 1
root@R5-IGR1# set protocols bgp group iBGP export MED
```

The purpose was to observe whether the policy-based MED value would be reflected while the direct BGP MED export configuration remained active.

The experiment was also intended to determine whether the two configuration methods would appear to work together or whether one would determine the resulting value.

---

## Step 2 — R1-PE1 BGP Route State

The resulting default-route state was checked on R1-PE1:

```text
[edit]
root@R1-PE1# run show route protocol bgp extensive 0.0.0.0

inet.0: 26 destinations, 33 routes (26 active, 0 holddown, 0 hidden)
0.0.0.0/0 (2 entries, 1 announced)
TSI: KRT in-kernel 0.0.0.0/0 -> {indirect(262143)}
Page 0 idx 0, (group eBGP type External) Type 1 val 0x99244a0 (adv_entry)
   Advertised metrics:
     Nexthop: Self
     AS path: [1000] 4000 I
     Communities:
    Advertise: 00000001
Path 0.0.0.0 from 6.6.6.6 Vector len 4.  Val: 0
        *BGP    Preference: 170/-101
                Next hop type: Indirect, Next hop index: 0
                Address: 0x78e5d24
                Next-hop reference count: 18
                Kernel Table Id: 0
                Source: 6.6.6.6
                Next hop type: Router, Next hop index: 570
                Next hop: 10.1.4.2 via ge-0/0/2.0, selected
                Session Id: 0
                Protocol next hop: 6.6.6.6
                Indirect next hop: 0x791e218 262143 INH Session ID: 0
                State: <Active Int Ext>
                Local AS:  1000 Peer AS:  1000
                Age: 15:18:37   Metric: 0       Metric2: 20
                Validation State: unverified
                ORR Generation-ID: 0
                Task: BGP_1000.6.6.6.6
                Announcement bits (3): 0-KRT 3-BGP_RT_Background 4-Resolve tree 4
                AS path: 4000 I
                Accepted
                Localpref: 100
                Router ID: 6.6.6.6
                Thread: junos-main
                Indirect next hops: 1
                        Protocol next hop: 6.6.6.6 Metric: 20 ResolvState: Resolved
                        Indirect next hop: 0x791e218 262143 INH Session ID: 0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 10.1.4.2 via ge-0/0/2.0
                                Session Id: 0
                                6.6.6.6/32 Originating RIB: inet.0
                                  Metric: 20 Node path count: 1
                                  Forwarding nexthops: 1
                                        Next hop type: Router
                                        Next hop: 10.1.4.2 via ge-0/0/2.0
                                        Session Id: 0
         BGP    Preference: 170/-101
                Next hop type: Indirect, Next hop index: 0
                Address: 0x78e59a4
                Next-hop reference count: 11
                Kernel Table Id: 0
                Source: 5.5.5.5
                Next hop type: Router, Next hop index: 571
                Next hop: 10.1.3.2 via ge-0/0/0.0, selected
                Session Id: 0
                Protocol next hop: 5.5.5.5
                Indirect next hop: 0x791dee8 262142 INH Session ID: 0
                State: <NotBest Int Ext Changed>
                Inactive reason: Not Best in its group - Route Metric or MED comparison
                Local AS:  1000 Peer AS:  1000
                Age: 17         Metric: 1       Metric2: 20
                Validation State: unverified
                ORR Generation-ID: 0
                Task: BGP_1000.5.5.5.5
                AS path: 4000 I
                Accepted
                Localpref: 100
                Router ID: 5.5.5.5
                Thread: junos-main
                Indirect next hops: 1
                        Protocol next hop: 5.5.5.5 Metric: 20 ResolvState: Resolved
                        Indirect next hop: 0x791dee8 262142 INH Session ID: 0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 10.1.3.2 via ge-0/0/0.0
                                Session Id: 0
                                5.5.5.5/32 Originating RIB: inet.0
                                  Metric: 20 Node path count: 1
                                  Forwarding nexthops: 1
                                        Next hop type: Router
                                        Next hop: 10.1.3.2 via ge-0/0/0.0
                                        Session Id: 0
```

---

## Step 2 Observation

Compared with Step 1:

```text
R5 route:
Metric 2  →  Metric 1

R6 route:
Metric 0  →  Metric 0
```

The R5 route remained non-best because Junos reported:

```text
Inactive reason: Not Best in its group - Route Metric or MED comparison
```

The active R6 route remained:

```text
Metric: 0
```

Therefore, the observed route state after adding the policy was:

```text
R6 = Metric 0 → Active
R5 = Metric 1 → NotBest
```

### Important Evidence Limitation

The captured output proves that the **final observed R5 Metric was 1** after the policy-based configuration was added while the Step 1 configuration remained present.

It does **not**, by itself, prove the internal Junos precedence mechanism between the direct BGP MED export configuration and the policy-based metric operation.

The test therefore documents the **observed result**, rather than claiming that both mechanisms are independently additive or that one universally overrides the other.

---

## Comparison of the Two Phases

| Phase | R5 Metric | R6 Metric | Selected Path | Evidence |
|---|---:|---:|---|---|
| Baseline | 0 | 0 | R5 | R1-PE1 route table + baseline traceroute |
| Step 1 — Direct `export MED` | **2** | **0** | **R6** | R1-PE1 route table + post-change traceroute |
| Step 2 — Policy metric 1 with Step 1 active | **1** | **0** | **R6** | R1-PE1 route table |

---

## Test Result

**PASS — MED behavior was observed and documented with CLI evidence.**

The test produced three clearly distinguishable route-state observations:

1. **Baseline:** R5 and R6 both showed Metric 0; R5 was selected.
2. **Direct MED export:** R5 changed to Metric 2 while R6 remained at Metric 0; R6 became selected.
3. **Policy-based MED added while Step 1 remained active:** R5 changed from Metric 2 to Metric 1 while R6 remained at Metric 0; R6 remained selected.

The end-to-end traceroute confirms the forwarding-path change in Step 1.

Continuous ping during Step 1 showed no packet loss in the captured interval.

No Step 2 traceroute or Step 2 continuous-ping output was included in the captured evidence, so no additional Step 2 traffic-continuity claim is made here.

---

## Evidence and Transparency

This document intentionally preserves the captured CLI outputs used to reach the conclusions.

The route-selection conclusions are based on:

- R1-PE1 `show route protocol bgp extensive 0.0.0.0` output.
- HQ PC2 traceroute before and after Step 1.
- HQ PC2 continuous-ping output during Step 1.
- The exact configuration commands recorded during both phases.

Where the captured evidence does not establish an internal implementation detail, the document explicitly avoids making that claim.

## Evidence Boundary

This test verifies the observed MED behavior in the specific **AS1000 / R5-IGR1 / R1-PE1** topology and configuration used in this lab.

It is not intended to claim that the observed numeric MED values or configuration interaction are universal for every Junos design.

