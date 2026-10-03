# BGP Attribute — AS-Path (Juniper)

## Objective

Verify and troubleshoot **BGP AS-Path manipulation** on **R6-IGR2** using a Junos routing policy and BGP export.

The AS-Path test was performed while continuous ICMP traffic was running from:

- Source: **HQ PC2**
- Source IP: `192.168.10.3`
- Destination: `8.8.8.8`

The test used:

```text
policy-options → as-path-prepend
protocols bgp → export
```

The test also investigated the effect of prepending an AS number that matches the local AS of the receiving routers.

---

## Test Environment

- Device under test: **R6-IGR2**
- Autonomous System: **AS1000**
- Platform: **Juniper Junos**
- BGP group used for the export policy: `iBGP`
- Source traffic: HQ PC2 `192.168.10.3`
- Destination: `8.8.8.8`

Relevant internal paths:

**R5 path**

```text
R1-PE1 → R3-P1 → R5-IGR1
```

**R6 path**

```text
R1-PE1 → R4-P2 → R6-IGR2
```

## Lab Steering Context — MED

Before the AS-Path test, the lab was intentionally placed in a specific forwarding state for educational purposes.

In the normal state of this lab, with no BGP path attribute being used to influence the competing default routes, the R5 and R6 paths had equal relevant values and the route selection could fall through to the Router ID tie-breaker. R5-IGR1 uses Router ID/loopback **5.5.5.5**, while R6-IGR2 uses **6.6.6.6**. The lower Router ID on R5 therefore causes the R5 path to be selected in the normal baseline condition.

For the MED exercise, MED was deliberately configured so that:

```text
R5 → Metric 2
R6 → Metric 0
```

This was **lab steering**, not part of the AS-Path manipulation itself. The purpose was to move the Internet traffic path away from R5-IGR1 and make it pass through **R6-IGR2** so that the AS-Path manipulation could be tested specifically on R6.

The MED configuration remained active throughout the AS-Path test. It was removed **during verification/cleanup after the AS-Path test**, not before it.

---

# Baseline — Before AS-Path Manipulation

## R1-PE1 — BGP Route State

The default route was checked on R1-PE1:

```text
[edit]
root@R1-PE1# run show route protocol bgp extensive 0.0.0.0
```

Relevant captured output:

```text
inet.0: 26 destinations, 33 routes (26 active, 0 holddown, 0 hidden)
0.0.0.0/0 (2 entries, 1 announced)
TSI:
KRT in-kernel 0.0.0.0/0 -> {indirect(262143)}
Page 0 idx 0, (group eBGP type External) Type 1 val 0x99244a0 (adv_entry)
   Advertised metrics:
     Nexthop: Self
     AS path: [1000] 4000 I
     Communities:
    Advertise: 00000001
Path 0.0.0.0
from 6.6.6.6
Vector len 4.  Val: 0
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
                Age: 17:24:00   Metric: 0       Metric2: 20
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
                Age: 4:06       Metric: 2       Metric2: 20
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

### Baseline Observation

Before the AS-Path manipulation:

- R6 route: `from 6.6.6.6`, **Metric 0**, AS path `4000 I`, Active.
- R5 route: `from 5.5.5.5`, **Metric 2**, AS path `4000 I`, NotBest.
- R5's route was reported as non-best because of **Route Metric or MED comparison**.

This state was intentional. The MED configuration was still active to keep the Internet forwarding path through **R6-IGR2** during the AS-Path exercise. The Metric 2 on R5 was therefore a pre-existing lab steering mechanism, not an AS-Path configuration.

---

## HQ PC2 — Baseline Traceroute

```text
PC2> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   0.624 ms  0.560 ms  0.457 ms
 2   10.10.10.1   1.044 ms  0.537 ms  0.699 ms
 3   10.10.30.2   1.584 ms  1.163 ms  1.218 ms
 4   10.1.4.2   1.557 ms  1.464 ms  1.232 ms
 5   10.4.6.2   1.722 ms  1.647 ms  2.162 ms
 6   10.6.20.1   1.988 ms  1.732 ms  1.838 ms
 7   10.142.13.1   2.182 ms  1.858 ms  2.514 ms
 8   10.50.245.29   4.527 ms  4.437 ms  3.941 ms
```

Observed forwarding path before the AS-Path test:

```text
PC2
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

# Continuous Ping During the Test

The AS-Path configuration was changed while continuous ICMP traffic was running from HQ PC2:

```text
PC2> ping 8.8.8.8 -t
```

Captured replies:

```text
84 bytes from 8.8.8.8 icmp_seq=1 ttl=109 time=54.380 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=109 time=52.890 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=109 time=53.567 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=109 time=53.586 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=109 time=53.319 ms
84 bytes from 8.8.8.8 icmp_seq=6 ttl=109 time=55.262 ms
84 bytes from 8.8.8.8 icmp_seq=7 ttl=109 time=52.704 ms
84 bytes from 8.8.8.8 icmp_seq=8 ttl=109 time=53.665 ms
84 bytes from 8.8.8.8 icmp_seq=9 ttl=109 time=56.268 ms
84 bytes from 8.8.8.8 icmp_seq=10 ttl=109 time=53.540 ms
84 bytes from 8.8.8.8 icmp_seq=11 ttl=109 time=55.132 ms
84 bytes from 8.8.8.8 icmp_seq=12 ttl=109 time=54.615 ms
84 bytes from 8.8.8.8 icmp_seq=13 ttl=109 time=52.823 ms
84 bytes from 8.8.8.8 icmp_seq=14 ttl=109 time=53.777 ms
84 bytes from 8.8.8.8 icmp_seq=15 ttl=109 time=52.765 ms
84 bytes from 8.8.8.8 icmp_seq=16 ttl=109 time=54.101 ms
84 bytes from 8.8.8.8 icmp_seq=17 ttl=109 time=53.803 ms
84 bytes from 8.8.8.8 icmp_seq=18 ttl=109 time=52.753 ms
84 bytes from 8.8.8.8 icmp_seq=19 ttl=109 time=53.761 ms
84 bytes from 8.8.8.8 icmp_seq=20 ttl=109 time=54.608 ms
```

No packet loss was observed in the captured interval.

---

## MED Context During the AS-Path Test

The MED configuration remained active during the AS-Path test so that Internet traffic continued through **R6-IGR2** for the duration of the exercise.

The steering state was:

```text
R5 route: Metric 2
R6 route: Metric 0
```

This was intentionally configured for the previous MED lab and kept in place while testing AS-Path on R6. After the AS-Path verification was completed, the MED configuration was removed during the verification/cleanup stage.

---

# Test 1 — AS-Path Prepend with Local AS Present

## Configuration Applied

The first AS-Path policy attempted to prepend:

```text
root@R6-IGR2# set policy-options policy-statement AS-Path then as-path-prepend "1000 99 1000"
root@R6-IGR2# set protocols bgp group iBGP export AS-Path
```

The objective was to modify the AS-Path advertised from R6-IGR2 toward its iBGP peers.

## Observed Problem

After applying the configuration, the route learned from **R6-IGR2 (6.6.6.6)** was no longer visible on the other AS1000 routers.

The user then checked R3-P1:

```text
[edit]
root@R3-P1# run show route protocol bgp

inet.0: 26 destinations, 27 routes (26 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[BGP/170] 00:01:22, MED 0, localpref 100, from 5.5.5.5
                      AS path: 4000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
10.6.10.0/30       *[BGP/170] 00:01:22, MED 0, localpref 100, from 5.5.5.5
                      AS path: 3000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
10.6.20.0/30       *[BGP/170] 00:01:22, MED 0, localpref 100, from 5.5.5.5
                      AS path: 4000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
10.10.30.0/30      *[BGP/170] 17:34:02, localpref 100, from 1.1.1.1
                      AS path: I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
10.10.40.0/30       [BGP/170] 00:01:22, localpref 100, from 5.5.5.5
                      AS path: 4000 2000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
20.4.10.0/30       *[BGP/170] 00:01:22, MED 0, localpref 100, from 5.5.5.5
                      AS path: 3000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
20.4.20.0/30       *[BGP/170] 00:01:22, MED 0, localpref 100, from 5.5.5.5
---(more)---[abort]
```

The R6-derived default route was not shown in the displayed R3-P1 output.

### First Diagnosis

The original prepend included:

```text
1000 99 1000
```

The important issue identified during troubleshooting was the presence of **AS1000**, which is the local AS of the AS1000 routers receiving the route.

An AS-Path containing the receiving router's own AS can trigger BGP loop prevention, causing the route to be rejected rather than installed.

---

# Test 2 — Remove the 99 but Keep AS1000 in the Prepend

To determine whether the problem was specifically caused by `99`, the prepend was changed to:

```text
root@R6-IGR2# set policy-options policy-statement AS-Path then as-path-prepend "1000 1000 1000"
```

The problem remained: the R6-derived routes still did not appear on the other AS1000 routers.

### Troubleshooting Conclusion

This second attempt isolated the problem further.

The value `99` was not the fundamental issue.

The critical value was still:

```text
1000
```

because **AS1000 is the local AS of the routers in the receiving AS1000 domain**.

Thus, prepending the local AS caused the route to fail AS-Path loop prevention when evaluated by another AS1000 BGP speaker.

No separate raw route-table output for this intermediate attempt was captured in the source material; the documented result is therefore based on the observed absence of the R6 route described during the troubleshooting process.

---

# Test 3 — Prepend AS3000 Instead

## Corrected Configuration

The prepend was then changed to:

```text
root@R6-IGR2# set policy-options policy-statement AS-Path then as-path-prepend "3000 3000 3000"
```

The route was then visible again across the AS1000 BGP topology.

---

## R1-PE1 — Verification After the Corrected Configuration

```text
[edit]
root@R1-PE1# run show route protocol bgp extensive 0.0.0.0

inet.0: 26 destinations, 33 routes (26 active, 0 holddown, 0 hidden)
0.0.0.0/0 (2 entries, 1 announced)
TSI:
KRT in-kernel 0.0.0.0/0 -> {indirect(262142)}
Page 0 idx 0, (group eBGP type External) Type 1 val 0x99244a0 (adv_entry)
   Advertised metrics:
     Nexthop: Self
     AS path: [1000] 4000 I
     Communities:
    Advertise: 00000001
Path 0.0.0.0
from 5.5.5.5
Vector len 4.  Val: 0
        *BGP    Preference: 170/-101
                Next hop type: Indirect, Next hop index: 0
                Address: 0x78e59a4
                Next-hop reference count: 18
                Kernel Table Id: 0
                Source: 5.5.5.5
                Next hop type: Router, Next hop index: 571
                Next hop: 10.1.4.2 via ge-0/0/2.0, selected
                Session Id: 0
                Protocol next hop: 5.5.5.5
                Indirect next hop: 0x791dee8 262142 INH Session ID: 0
                State: <Active Int Ext>
                Local AS:  1000 Peer AS:  1000
                Age: 38:17      Metric: 0       Metric2: 30
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
                        Protocol next hop: 5.5.5.5 Metric: 30 ResolvState: Resolved
                        Indirect next hop: 0x791dee8 262142 INH Session ID: 0
                        Indirect path forwarding next hops: 1
                                Next hop type: Router
                                Next hop: 10.1.4.2 via ge-0/0/2.0
                                Session Id: 0
                                5.5.5.5/32 Originating RIB: inet.0
                                  Metric: 30 Node path count: 1
                                  Forwarding nexthops: 1
                                        Next hop type: Router
                                        Next hop: 10.1.4.2 via ge-0/0/2.0
                                        Session Id: 0
         BGP    Preference: 170/-101
                Next hop type: Indirect, Next hop index: 0
                Address: 0x78e59a4
                Next-hop reference count: 11
                Kernel Table Id: 0
                Source: 6.6.6.6
                Next hop type: Router, Next hop index: 570
                Next hop: 10.1.4.2 via ge-0/0/2.0, selected
                Session Id: 0
                Protocol next hop: 6.6.6.6
                Indirect next hop: 0x791e218 262143 INH Session ID: 0
                State: <Int Ext Changed>
                Inactive reason: AS path
                Local AS:  1000 Peer AS:  1000
                Age: 36:19      Metric: 0       Metric2: 20
                Validation State: unverified
                ORR Generation-ID: 0
                Task: BGP_1000.6.6.6.6
                AS path: 3000 3000 3000 4000 I
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
```

### R1-PE1 Observation

The R6 route was now accepted:

```text
Source: 6.6.6.6
AS path: 3000 3000 3000 4000 I
Accepted
Inactive reason: AS path
```

The R6 route was present but not selected.

The R5 route remained active.

---

## R3-P1 — Verification After the Corrected Configuration

```text
[edit]
root@R3-P1# run show route protocol bgp

inet.0: 28 destinations, 36 routes (28 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[BGP/170] 00:02:03, MED 0, localpref 100, from 5.5.5.5
                      AS path: 4000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
                    [BGP/170] 00:00:06, MED 0, localpref 100, from 6.6.6.6
                      AS path: 3000 3000 3000 4000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
10.5.10.0/30       *[BGP/170] 00:00:06, MED 0, localpref 100, from 6.6.6.6
                      AS path: 3000 3000 3000 3000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
10.5.20.0/30       *[BGP/170] 00:00:06, MED 0, localpref 100, from 6.6.6.6
                      AS path: 3000 3000 3000 4000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
10.6.10.0/30       *[BGP/170] 00:02:03, MED 0, localpref 100, from 5.5.5.5
                      AS path: 3000 I, validation-state: unverified
                    >  to 10.1.3.1 via ge-0/0/0.0
10.6.20.0/30       *[BGP/170] 00:02:03, MED 0, localpref 100, from 5.5.5.5
```

The captured output confirms that R3-P1 received the default route from both R5 and R6:

```text
R5:
AS path: 4000 I

R6:
AS path: 3000 3000 3000 4000 I
```

Both paths were present simultaneously.

For the default route, R5 remained the active path while the R6 route was also installed as an available BGP path.

---

# Final Analysis

## What the Test Proved

The test demonstrated the practical effect of AS-Path manipulation in this Junos lab.

### 1. Prepending the Local AS Can Trigger Loop Prevention

Using:

```text
as-path-prepend "1000 99 1000"
```

resulted in the R6-derived route disappearing from the other AS1000 routers during the test.

Changing it to:

```text
as-path-prepend "1000 1000 1000"
```

did not resolve the issue.

The common factor was the presence of **AS1000**, which is the local AS of the receiving BGP domain.

### 2. Prepending AS3000 Allowed the Route to Propagate

Using:

```text
as-path-prepend "3000 3000 3000"
```

resulted in the R6 route being visible on R1-PE1 and R3-P1 with:

```text
AS path: 3000 3000 3000 4000 I
```

The route was accepted and present as an alternate path.

### 3. AS-Path Length Influenced the Resulting Best-Path Decision

On R3-P1, the two observed default-route paths were:

```text
R5 → AS path: 4000 I
R6 → AS path: 3000 3000 3000 4000 I
```

The R5 path remained active while the longer R6 path was also present.

This is consistent with the observed route-selection state in the lab.

---

## Test Result

**PASS — AS-Path manipulation, loop-prevention behavior, and path-length impact were demonstrated with CLI evidence.**

The MED configuration used before and during this test was a deliberate lab-steering mechanism to keep traffic through R6-IGR2. It was not part of the AS-Path manipulation itself and was removed during verification/cleanup after the AS-Path test.

The test followed an actual troubleshooting sequence:

```text
Initial prepend
     ↓
R6 route disappears from AS1000 peers
     ↓
Remove 99 but keep AS1000
     ↓
Problem remains
     ↓
Use AS3000 instead
     ↓
R6 route returns
     ↓
R6 path contains 3000 3000 3000 4000
     ↓
R5 shorter path remains active
```

---

## Evidence and Transparency

This document preserves the captured command outputs and distinguishes them from observations made during troubleshooting.

The evidence includes:

- Initial R1-PE1 BGP route state.
- HQ PC2 baseline traceroute.
- HQ PC2 continuous-ping output during the test.
- R3-P1 output showing the R6 route was not present during the failed configuration.
- The intermediate configuration attempt using `1000 1000 1000`.
- Final corrected configuration using `3000 3000 3000`.
- Full R1-PE1 verification output after the corrected configuration.
- R3-P1 verification showing both the R5 and R6 default-route paths.

No raw route-table output was captured for the intermediate `1000 1000 1000` attempt, so that stage is documented as an observed troubleshooting result rather than represented as a raw CLI excerpt.

## Evidence Boundary

This test documents the behavior observed in the specific **AS1000 / R6-IGR2 / R1-PE1 / R3-P1** topology and configuration used during the session.

The test directly demonstrates that:

- a prepend containing AS1000 caused the R6 route to disappear from the receiving AS1000 BGP views in this lab;
- a prepend using AS3000 allowed the route to propagate;
- the resulting longer AS-Path kept the R6 path from becoming the selected default route on R3-P1.

The document does not claim that every aspect of Junos AS-Path policy processing is universally identical across all Junos releases or topologies.
