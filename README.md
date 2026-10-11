# RIP: Routing Information Protocol

**Networking & Security project: a study of a Layer 3 routing protocol**
Université Clermont Auvergne · April 2026

This repository contains our presentation (in French) and demo material on **RIP (Routing Information Protocol)**. It covers how the protocol works, a hands-on demo in Cisco Packet Tracer, and a critical look at its limits and security weaknesses.

## Contents

| File | Description |
|------|-------------|
| `RIP_Protocol_presentation.pptx` | The presentation (in French) |
| `packet-tracer/` | Packet Tracer lab file(s) *(add your `.pkt` here)* |
| `screenshots/` | Topology and packet captures *(optional)* |

## Presentation Outline

1. **Context & theory:** the role of RIP, its place in the OSI model, and the distance-vector principle
2. **How it works:** message format, metric, routing table, and timers
3. **Packet Tracer demo:** topology, router configuration, and RIP packet capture
4. **Critical analysis:** limits, security vulnerabilities, and modern alternatives

## RIP at a Glance

| Property | Value |
|----------|-------|
| Type | IGP (Interior Gateway Protocol), distance-vector |
| OSI layer | Layer 3 (Network) |
| Transport | UDP, port 520 |
| Metric | Hop count (maximum 15, 16 = unreachable) |
| Update interval | Every 30 seconds |
| Delivery | Broadcast (RIPv1) / multicast `224.0.0.9` (RIPv2) |
| Algorithm | Bellman-Ford |

### History

| Year | Version | RFC |
|------|---------|-----|
| 1988 | RIPv1 | RFC 1058 |
| 1997 | RIPng (IPv6) | RFC 2080 |
| 1998 | RIPv2 | RFC 2453 |

## How It Works

### Distance-vector principle

Each router only knows the distance (in hops) to each destination and the neighbor to send traffic through. Every 30 seconds it sends its whole routing table to its neighbors, which use it to update their own. Information spreads hop by hop until every router has a consistent view of the network. This is called **convergence**.

### RIPv2 message format

RIP messages travel in UDP datagrams (port 520) inside IP packets sent to `224.0.0.9`.

| Field | Size | Description |
|-------|------|-------------|
| Command | 1 byte | 1 = Request, 2 = Response |
| Version | 1 byte | 2 for RIPv2 |
| Routing entries | 20 bytes × N | Up to 25 routes per packet |
| AFI | 2 bytes | Address Family Identifier (2 = IPv4) |
| Route Tag | 2 bytes | Marks external routes |
| IP Address | 4 bytes | Destination network |
| Subnet Mask | 4 bytes | Mask (new in v2) |
| Next Hop | 4 bytes | Next hop (`0.0.0.0` = the sender) |
| Metric | 4 bytes | Hop count (1 to 15, 16 = infinity) |

**What RIPv2 adds over RIPv1:** VLSM/CIDR support, multicast instead of broadcast, MD5 authentication (RFC 2082), and route tags to tell internal and external routes apart.

### Timers

| Timer | Default | Role |
|-------|---------|------|
| Update | 30 s | Interval between periodic table broadcasts |
| Invalid | 180 s | With no news, the route is marked invalid (metric = 16) |
| Holddown | 180 s | Wait before accepting new information about a route |
| Flush | 240 s | Delay before the route is removed for good |

### Loop-prevention mechanisms

- **15-hop limit:** a metric of 16 means "infinity" (unreachable), which limits loop propagation.
- **Split horizon:** a router does not advertise a route out of the interface it learned it from.
- **Poison reverse:** the route is advertised back on the learning interface with metric 16.
- **Triggered updates:** changes are sent immediately, without waiting for the 30-second timer.

## Packet Tracer Demo

<!-- Add a screenshot of your topology here:
![Topology](screenshots/topology.png) -->

The lab uses Cisco ISR4331 routers, 2960 switches, and PCs. The routers are connected in a chain.

<!-- TODO: add the IP addressing plan and describe what the demo shows (e.g. convergence, hop-count limit, packet capture) -->

### Configuration of router R1 (Cisco IOS)

```
R1> enable
R1# configure terminal
R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# no auto-summary
R1(config-router)# network 10.1.1.0
R1(config-router)# network 192.168.12.0
R1(config-router)# network 192.168.13.0
R1(config-router)# end
R1# show ip route rip
```

- `version 2` enables RIPv2 (VLSM support and multicast updates).
- `no auto-summary` stops RIP from summarizing networks to their classful boundaries.
- Each `network` command enables RIP on the interfaces in that network.

### Running the lab

1. Open the `.pkt` file in Cisco Packet Tracer.
2. Check the RIP configuration on each router (`show running-config`).
3. Check the learned routes with `show ip route rip`.
4. Use Simulation mode with the RIP filter to capture the update packets.

## Limits of RIP

| Limit | Explanation |
|-------|-------------|
| Poor metric | Only the hop count matters, so a 10 Gb/s fiber link and a 64 kbps serial link look the same |
| Limited diameter | Maximum of 15 hops, which rules out large networks |
| Slow convergence | A failure takes at least 180 s to detect, and routing can be inconsistent for minutes |
| No hierarchy | No areas like OSPF, so it does not scale beyond small networks |
| Possible routing loops | When a route becomes invalid, the news spreads slowly and temporary loops can form |
| Bandwidth use | The full table is sent every 30 s even when nothing changes |

## Security Vulnerabilities

| Threat | Risk | Description | Mitigation |
|--------|------|-------------|------------|
| Fake route injection | 🔴 High | An attacker on the local segment sends RIP packets with a very low metric to attract traffic (Man-in-the-Middle) | MD5 authentication (RIPv2), which is off by default |
| Denial of service | 🟠 Medium | Flooding fake updates to overload routers or force constant table recalculation | ACL on UDP port 520, neighbor filtering |
| Passive listening | 🟠 Medium | Multicast `224.0.0.9` exposes the internal network topology to any machine on the segment | VLAN segmentation, passive interfaces on LANs without routers |

## Conclusion

RIP is **a teaching protocol, rarely a production choice**.

1. **Historical simplicity:** one of the first standardized IGPs, and still an excellent teaching tool.
2. **Structural limits:** a simple metric, slow convergence, and a 15-hop diameter make it unsuitable for medium and large modern networks.
3. **Replaced by OSPF / EIGRP:** in enterprise networks, link-state OSPF and Cisco's EIGRP converge faster, use better metrics, and offer real hierarchy.

## References

- [RFC 1058](https://datatracker.ietf.org/doc/html/rfc1058): Routing Information Protocol (RIPv1)
- [RFC 2453](https://datatracker.ietf.org/doc/html/rfc2453): RIP Version 2
- [RFC 2080](https://datatracker.ietf.org/doc/html/rfc2080): RIPng for IPv6
- [RFC 2082](https://datatracker.ietf.org/doc/html/rfc2082): RIP-2 MD5 Authentication
