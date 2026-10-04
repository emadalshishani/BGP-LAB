# BGP Attribute — Local Preference (Export)

## Objective

Verify the effect of the BGP **Local Preference** attribute on path selection inside **AS1000**.

The test was performed on **R5-IGR1** by applying a Local Preference value to routes exported from the **iBGP** group.

## Device

- Device: R5-IGR1
- Autonomous System: AS1000
- Platform: Juniper Junos
- iBGP group: `iBGP`

## Configuration Applied

The following policy was configured on R5-IGR1:

```text
set policy-options policy-statement local-pref term 1 then local-preference 99
set protocols bgp group iBGP export local-pref
```

This configuration applies Local Preference value **99** to routes exported from R5-IGR1 toward its iBGP peers.

---

## Baseline — Before Local Preference Change

Traffic was generated from **HQ PC3**:

- Source: `192.168.10.2`
- Destination: `8.8.8.8`

### Baseline Traceroute

```text
PC3> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   1.368 ms  1.090 ms  0.789 ms
 2   10.10.10.1   1.904 ms  1.146 ms  1.094 ms
 3   10.10.30.2   2.393 ms  1.997 ms  1.654 ms
 4   10.1.3.2   2.289 ms  1.884 ms  1.515 ms
 5   10.3.5.2   2.547 ms  1.952 ms  2.369 ms
 6   10.5.10.1   3.272 ms  2.416 ms  2.246 ms
 7   10.142.13.1   3.525 ms  3.318 ms  4.357 ms
 8   10.50.245.29   3.896 ms  4.694 ms  5.330 ms
```

The observed forwarding path through AS1000 was:

```text
HQ PC3
  ↓
R1-PE1
  ↓
R3-P1
  ↓
R5-IGR1
  ↓
ISP1 / AS3000
  ↓
10.142.13.1
  ↓
Internet
```

### Baseline Traffic Verification

Continuous ICMP traffic was also tested:

```text
PC3> ping 8.8.8.8 -t
```

20 replies were observed during the captured test interval.

No packet loss was observed.

---

## Local Preference Applied

The Local Preference policy was configured on R5-IGR1:

```text
set policy-options policy-statement local-pref term 1 then local-preference 99
set protocols bgp group iBGP export local-pref
```

### Resulting Traceroute

After the policy was applied, the forwarding path changed:

```text
PC3> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   1.274 ms  1.121 ms  0.944 ms
 2   10.10.10.1   1.837 ms  1.572 ms  1.238 ms
 3   10.10.30.2   2.081 ms  1.773 ms  1.924 ms
 4   10.1.4.2   2.529 ms  1.980 ms  1.960 ms
 5   10.4.6.2   2.777 ms  2.918 ms  2.407 ms
 6   10.6.10.1   2.993 ms  2.668 ms  2.821 ms
 7   10.142.13.1   3.548 ms  3.316 ms  3.599 ms
 8   10.50.245.29   5.555 ms  14.785 ms  5.635 ms
```

The observed path changed to:

```text
HQ PC3
  ↓
R1-PE1
  ↓
R4-P2
  ↓
R6-IGR2
  ↓
ISP1 / AS3000
  ↓
10.142.13.1
  ↓
Internet
```

The important change inside AS1000 was:

```text
Before:
R3-P1 → R5-IGR1 → ISP1

After Local Preference = 99 on R5 iBGP export:
R4-P2 → R6-IGR2 → ISP1
```

The test demonstrated that changing Local Preference on the routes exported by R5 influenced the path selected inside AS1000.

### Traffic Continuity During the Change

Continuous ping traffic was active during the test.

No packet loss was observed when the forwarding path changed.

---

## Policy Removal / Recovery

The Local Preference export policy was removed from R5-IGR1:

```text
[edit]
root@R5-IGR1# delete protocols bgp group iBGP export local-pref

[edit]
root@R5-IGR1# commit
commit complete
```

### Traffic Verification After Policy Removal

Continuous ICMP from HQ PC3 to `8.8.8.8` remained active.

The captured test showed replies for sequences 1 through 31.

No packet loss was observed.

### Traceroute After Reverting the Policy

```text
PC3> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   1.180 ms  0.788 ms  0.825 ms
 2   10.10.10.1   1.533 ms  1.251 ms  1.071 ms
 3   10.10.30.2   2.079 ms  1.988 ms  1.644 ms
 4   10.1.3.2   2.309 ms  2.980 ms  2.356 ms
 5   10.3.5.2   2.855 ms  2.669 ms  2.620 ms
 6   10.5.10.1   3.190 ms  2.826 ms  164.053 ms
 7   10.142.13.1   38.885 ms  8.738 ms  3.848 ms
 8   10.50.245.29   5.802 ms  5.496 ms  4.562 ms
```

The original forwarding path through R5-IGR1 was restored:

```text
R3-P1 → R5-IGR1 → ISP1
```

---

## Test Result

**PASS — Local Preference path-selection behavior verified.**

The test demonstrated three observable states:

1. **Baseline:** traffic used R5-IGR1 → ISP1.
2. **Local Preference = 99 on R5 iBGP export:** traffic shifted to R6-IGR2 → ISP1.
3. **Policy removed:** traffic returned to R5-IGR1 → ISP1.

Throughout the captured path changes, continuous ICMP traffic showed **no packet loss**.

## Evidence Boundary

This test verifies the effect of Local Preference in the specific configuration used on R5-IGR1.

The test does not claim that this is the preferred production design for all Local Preference use cases. It documents the observed behavior of this lab configuration.


---

## Device Verification Output

The following excerpts preserve the key CLI evidence from the test session. Dynamic counters are not reproduced here when they are not relevant to the attribute behavior.

### R5-IGR1 — BGP Route State

The relevant default-route state was inspected on R5-IGR1 while evaluating the path selected inside AS1000.

The important evidence was the Local Preference value and the selected next hop. The active forwarding path changed from the R5/ISP1 path to the R6/ISP1 path when the export policy was applied, and returned after the policy was removed.

### HQ PC3 — Traceroute Before Policy

```text
PC3> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   1.368 ms  1.090 ms  0.789 ms
 2   10.10.10.1   1.904 ms  1.146 ms  1.094 ms
 3   10.10.30.2   2.393 ms  1.997 ms  1.654 ms
 4   10.1.3.2   2.289 ms  1.884 ms  1.515 ms
 5   10.3.5.2   2.547 ms  1.952 ms  2.369 ms
 6   10.5.10.1   3.272 ms  2.416 ms  2.246 ms
 7   10.142.13.1   3.525 ms  3.318 ms  4.357 ms
 8   10.50.245.29   3.896 ms  4.694 ms  5.330 ms
```

### HQ PC3 — Traceroute After Export Policy

```text
PC3> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   1.274 ms  1.121 ms  0.944 ms
 2   10.10.10.1   1.837 ms  1.572 ms  1.238 ms
 3   10.10.30.2   2.081 ms  1.773 ms  1.924 ms
 4   10.1.4.2   2.529 ms  1.980 ms  1.960 ms
 5   10.4.6.2   2.777 ms  2.918 ms  2.407 ms
 6   10.6.10.1   2.993 ms  2.668 ms  2.821 ms
 7   10.142.13.1   3.548 ms  3.316 ms  3.599 ms
 8   10.50.245.29   5.555 ms  14.785 ms  5.635 ms
```

### HQ PC3 — Traceroute After Policy Removal

```text
PC3> trace 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   192.168.10.1   1.180 ms  0.788 ms  0.825 ms
 2   10.10.10.1   1.533 ms  1.251 ms  1.071 ms
 3   10.10.30.2   2.079 ms  1.988 ms  1.644 ms
 4   10.1.3.2   2.309 ms  2.980 ms  2.356 ms
 5   10.3.5.2   2.855 ms  2.669 ms  2.620 ms
 6   10.5.10.1   3.190 ms  2.826 ms  164.053 ms
 7   10.142.13.1   38.885 ms  8.738 ms  3.848 ms
 8   10.50.245.29   5.802 ms  5.496 ms  4.562 ms
```

> **Evidence note:** These traceroute outputs are the end-to-end evidence used to verify the path change. The route-selection conclusion is based on the observed next-hop changes and the Local Preference policy applied on R5-IGR1.
