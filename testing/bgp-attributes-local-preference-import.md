# BGP Attribute — Local Preference (Import)

## Objective

Verify the effect of **Local Preference** when it is modified by an **import policy** on routes received from an external BGP neighbor.

The test was performed on **R5-IGR1** in **AS1000**. Local Preference was reduced for routes received from **ISP2 / AS4000**, causing the router to prefer an alternative path through **ISP1 / AS3000** when both paths were otherwise available.

The test was then extended with an **ISP1 link failure** to verify the resulting failover behavior and recovery.

## Device and Peering Context

- Device: R5-IGR1
- Autonomous System: AS1000
- Platform: Juniper Junos
- eBGP group: `ebgp_AS4000`
- ISP2 neighbor: `10.5.20.1`
- ISP2 Autonomous System: AS4000
- Alternate direct ISP1 next hop: `10.5.10.1`
- Internal alternate path via R6-IGR2: `10.5.6.2`

## Configuration Applied

The following policy was configured on R5-IGR1:

```text
set policy-options policy-statement local-pref term 1 then local-preference 99
set protocols bgp group ebgp_AS4000 neighbor 10.5.20.1 import local-pref
```

The policy changes the Local Preference of routes **received from ISP2** before those routes are used in AS1000 path selection.

---

## Baseline — Before Import Policy

Before the policy was applied, R5 had multiple BGP paths for the default route:

| Path | Local Preference | AS Path | Next Hop |
|---|---:|---|---|
| Direct from ISP2 | 100 | `4000 I` | `10.5.20.1` |
| Via R6-IGR2 | 100 | `4000 I` | `10.5.6.2` |
| Direct from ISP1 | 100 | `3000 I` | `10.5.10.1` |

The active default path before the import policy used the **direct ISP2 path**.

### Baseline End-to-End Traceroute

Traffic was generated from **HQ PC2**:

- Source: `192.168.10.3`
- Destination: `8.8.8.8`

Observed traceroute:

```text
1   192.168.10.1
2   10.10.10.1
3   10.10.30.2
4   10.1.3.2
5   10.3.5.2
6   10.5.20.1
7   10.142.13.1
8   10.50.245.29
```

Observed path:

```text
HQ PC2
  ↓
HQ / AS100
  ↓
R1-PE1
  ↓
R3-P1
  ↓
R5-IGR1
  ↓
ISP2 / AS4000
  ↓
10.142.13.1
  ↓
Internet
```

---

## Import Local Preference Applied

The import policy was applied to the R5-IGR1 ↔ ISP2 eBGP session:

```text
set policy-options policy-statement local-pref term 1 then local-preference 99
set protocols bgp group ebgp_AS4000 neighbor 10.5.20.1 import local-pref
```

### Resulting Route Selection

After the policy was applied:

| Path | Local Preference | Next Hop | Selection |
|---|---:|---|---|
| Direct from ISP2 | 99 | `10.5.20.1` | Not active |
| Via R6-IGR2 | 100 | `10.5.6.2` | Available |
| Direct from ISP1 | 100 | `10.5.10.1` | Active |

The direct ISP2 route was therefore deprioritized relative to the available ISP1 route.

### Resulting End-to-End Traceroute

Observed traceroute after applying the import policy:

```text
1   192.168.10.1
2   10.10.10.1
3   10.10.30.2
4   10.1.3.2
5   10.3.5.2
6   10.5.10.1
7   10.142.13.1
8   10.50.245.29
```

The observed path changed from:

```text
R3-P1 → R5-IGR1 → ISP2
```

to:

```text
R3-P1 → R5-IGR1 → ISP1
```

### Traffic Continuity

Continuous ICMP traffic from HQ PC2 to `8.8.8.8` was tested while the import policy was active.

Captured replies covered sequences 1 through 29.

No packet loss was observed during the policy-induced path change.

---

## Failure Test — ISP1 Link Failure

The purpose of this stage was to verify how the network behaved after the import policy had intentionally reduced the Local Preference of the direct ISP2 route.

Before the failure, the active path was through ISP1:

```text
R3-P1 → R5-IGR1 → ISP1
```

A continuous ping was started from HQ PC2 to `8.8.8.8`.

On **ISP1**, the Ethernet interface connected to R5-IGR1 was shut down:

```text
interface ethernet 0/0
shutdown
```

### ICMP Result During Convergence

The captured ping showed:

- Replies: sequences 1–3
- Timeouts: sequences 4–47
- First reply after convergence: sequence 48
- Consecutive packet loss during convergence: **44 packets**

This demonstrates that failover occurred, but the transition was **not lossless** in this test.

### Post-Failure Traceroute

After the ISP1 link failure, traceroute showed:

```text
1   192.168.10.1
2   10.10.10.1
3   10.10.30.2
4   10.1.4.2
5   10.4.6.2
6   10.6.20.1
7   10.142.13.1
8   10.50.245.29
```

The observed failover path was:

```text
HQ PC2
  ↓
HQ / AS100
  ↓
R1-PE1
  ↓
R4-P2
  ↓
R6-IGR2
  ↓
ISP2 / AS4000
  ↓
10.142.13.1
  ↓
Internet
```

### R5 Route State After ISP1 Failure

After ISP1 failure, the R5 active default route was learned through R6-IGR2:

- Active path: via `6.6.6.6`
- Local Preference: **100**
- AS Path: `4000 I`
- Next Hop: `10.5.6.2`

The direct ISP2 route remained available but kept the imported Local Preference of **99**:

- Local Preference: **99**
- Next Hop: `10.5.20.1`

The result shows why R5 preferred the route received from R6 over its own direct ISP2 route after ISP1 failed: the iBGP-learned route retained Local Preference **100**, while the route directly imported from ISP2 had been changed to **99**.

---

## Policy Removal and Recovery

The import policy was removed from the ISP2 eBGP neighbor, and the ISP1 interface was restored:

```text
delete protocols bgp group ebgp_AS4000 neighbor 10.5.20.1 import
```

On ISP1:

```text
no shutdown
```

The BGP sessions returned to the established state. The captured R5 BGP summary showed all six peers established with no down peers.

### Traffic Verification After Recovery

Continuous ICMP traffic from HQ PC2 to `8.8.8.8` was tested during recovery.

Captured sequences 1 through 33 received replies with no packet loss.

### Traceroute After Recovery

After removing the import policy and restoring ISP1, the observed path returned to:

```text
1   192.168.10.1
2   10.10.10.1
3   10.10.30.2
4   10.1.3.2
5   10.3.5.2
6   10.5.20.1
7   10.142.13.1
8   10.50.245.29
```

At recovery, the direct ISP2 route was again observed as the active default path with:

- Local Preference: **100**
- Next Hop: `10.5.20.1`

The direct ISP1 path was also established again.

---

## Test Result

**PASS — Import-based Local Preference behavior and failover path selection were observed for this lab configuration.**

The test demonstrated four observable states:

1. **Baseline:** direct ISP2 path was active.
2. **Import Local Preference = 99 from ISP2:** traffic shifted to direct ISP1.
3. **ISP1 failure while the import policy remained active:** traffic failed over through R4-P2 → R6-IGR2 → ISP2, with **44 consecutive ICMP losses** during convergence.
4. **Import policy removed and ISP1 restored:** BGP sessions recovered, packet loss was not observed during the captured recovery interval, and the active path returned to direct ISP2.

## Evidence Boundary

This document records the behavior observed in this specific lab topology and configuration.

It does not claim that the selected paths are universally optimal or that the failover is hitless/seamless.

The recorded packet loss during failover is explicitly included as part of the test evidence.
