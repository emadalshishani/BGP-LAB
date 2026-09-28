# BGP Failover Testing

## Test Method

Each control-plane failure test was documented using:

1. Baseline state
2. Failure injection
3. BGP convergence
4. Route verification
5. Recovery

The end-to-end test additionally used continuous ICMP and traceroute to verify the data plane.

---

## Test 1 — R5 ↔ ISP1 eBGP Link Failure

### Failure Injection

On R5-IGR1:

```text
configure
set interfaces ge-0/0/3 disable
commit
```

Interface ge-0/0/3 is the R5 connection toward ISP1 (AS3000).

### Observed During Failure

- R5's eBGP session to 10.5.10.1 (AS3000) went Idle.
- R5's ISP2 session to 10.5.20.1 (AS4000) remained Established.
- R6 remained connected to both ISP domains.
- The active route to 10.5.10.0/30 moved to the internal R6 path via 10.5.6.2 with AS path 3000 I.
- The HQ prefix remained reachable through an alternate path via R6.

### Recovery

```text
configure
delete interfaces ge-0/0/3 disable
commit
exit
```

The R5 ↔ ISP1 session returned to Established and the direct route was restored.

### Result

**PASS — BGP failure detection, convergence, and recovery verified.**

No end-to-end traffic continuity claim is made for this test.

---

## Test 2 — R6 ↔ ISP1 eBGP Link Failure

### Failure Injection

On R6-IGR2:

```text
configure
set interfaces ge-0/0/3 disable
commit
```

### Observed During Failure

- R6's eBGP session to 10.6.10.1 (AS3000) went Idle.
- R6's ISP2 session to 10.6.20.1 (AS4000) remained Established.
- The default route moved to the AS4000 path.
- 10.5.10.0/30 was reachable through AS4000 with AS path 4000 3000 I.
- 10.6.10.0/30 remained reachable through the alternate R5 path with AS path 3000 I.
- R5 retained an alternate path to the HQ prefix.

### Recovery

```text
configure
delete interfaces ge-0/0/3 disable
commit
exit
```

R6 returned to zero down peers, and the ISP1 session was re-established.

### Result

**PASS — R6 ↔ ISP1 eBGP failure, alternate-path selection, and recovery verified.**

No end-to-end traffic continuity claim is made for this test.

---

## Test 3 — Full ISP2 Failure / End-to-End Failover

### Baseline

HQ PC2:

- IP: 192.168.10.3/29
- Gateway: 192.168.10.1

Baseline ICMP to 8.8.8.8 succeeded at approximately 65–66 ms.

Baseline traceroute:

```text
1  192.168.10.1
2  10.10.10.1
3  10.10.30.2
4  10.1.3.2
5  10.3.5.2
6  10.5.20.1
7  10.142.13.1
8  10.50.245.29
```

The hop 10.5.20.1 is ISP2 / AS4000.

### Failure Injection

On ISP2:

```text
configure
interface range ethernet 0/0-1
shutdown
```

This removed both ISP2-facing links toward the AS1000 routers.

### BGP Convergence

On R5 and R6:

- ISP2 sessions moved to Connect.
- ISP1 sessions remained Established.
- The active default route moved to ISP1 / AS3000.
- On R5, the active default path was via 10.5.10.1 with AS path 3000 I.
- On R6, the active default path was via 10.6.10.1.

### Data Plane Observation

Continuous ICMP from PC2 to 8.8.8.8 showed:

- replies through sequence 6
- timeouts from sequence 7 through 48
- replies resumed at sequence 49
- replies continued afterward

This represents **42 consecutive packet losses during convergence**.

The test demonstrates convergence with measurable packet loss; it was not a hitless failure event.

### Post-Failure Traceroute

After convergence, the path changed to:

```text
1  192.168.10.1
2  10.10.10.1
3  10.10.30.2
4  10.1.3.2
5  10.3.5.2
6  10.5.10.1
7  10.142.13.1
8  10.50.245.29
```

The path moved from ISP2 / AS4000 (10.5.20.1) to ISP1 / AS3000 (10.5.10.1).

### Recovery

On ISP2:

```text
no shutdown
do wr
```

The ISP2 BGP sessions returned to Established and the routers reported zero down peers.

### Result

**PASS — End-to-end ISP failover verified.**

The test verified actual traffic-path movement from ISP2 / AS4000 to ISP1 / AS3000 after BGP convergence.

No post-recovery PC2 ping/traceroute capture was recorded, so the documentation does not claim a post-recovery end-to-end test beyond BGP/session restoration.
