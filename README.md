# DHCP Configuration for Multiple LANs (Cisco Packet Tracer)

A CCNA lab where three Cisco routers act as **DHCP servers** for six separate LANs (36 PCs in total). Every PC gets its IP address and default gateway automatically, and PCs in different LANs can ping each other across the routers.

> **Tool:** Cisco Packet Tracer | **Topic:** DHCP, IP addressing, inter-LAN routing | **Level:** CCNA

---

## 1. Project Overview

| Item | Details |
|---|---|
| Routers | 3 (R1, R2, R3), connected in a chain |
| Switches | 6 x Cisco 2950T-24 (one per LAN) |
| End devices | 36 PCs (6 per LAN) |
| LANs | LAN10, LAN20, LAN30, LAN40, LAN50, LAN60 |
| DHCP server | Each router serves the two LANs directly connected to it |
| Router-to-router links | /30 point-to-point subnets |

**Goal:** Configure DHCP pools on each router so that all PCs receive addresses automatically, then verify connectivity between LANs with ping.

---

## 2. Network Topology

![Topology](screenshots/01-topology.png)

```
 LAN10 ---+                                                  +--- LAN40
          |                                                  |
          +-- R1 ----------(10.0.0.0/30)---------- R2 -------+  (R2 serves LAN20 and LAN50)
 LAN40 ---+                                          |
                                               (10.0.0.4/30)
                                                     |
                                                    R3
                                                     |
                                    LAN30 and LAN60 attach here
```

Left side of the diagram: LAN10, LAN20, LAN30 (PC0 - PC17).
Right side of the diagram: LAN40, LAN50, LAN60 (PC0(1) - PC17(1)).

---

## 3. IP Addressing Table

### LAN networks

| LAN | Network | Mask | Gateway (router interface) | Router | Router interface | PCs |
|---|---|---|---|---|---|---|
| LAN10 | 192.168.10.0 | 255.255.255.0 | 192.168.10.1 | R1 | Fa1/0 | PC0 - PC5 |
| LAN20 | 192.168.20.0 | 255.255.255.0 | 192.168.20.1 | R2 | Fa0/1 | PC6 - PC11 |
| LAN30 | 192.168.30.0 | 255.255.255.0 | 192.168.30.1 | R3 | Fa0/1 | PC12 - PC17 |
| LAN40 | 192.168.40.0 | 255.255.255.0 | 192.168.40.1 | R1 | Fa0/1 | PC0(1) - PC5(1) |
| LAN50 | 192.168.50.0 | 255.255.255.0 | 192.168.50.1 | R2 | Fa1/0 | PC6(1) - PC11(1) |
| LAN60 | 192.168.60.0 | 255.255.255.0 | 192.168.60.1 | R3 | Fa1/0 | PC12(1) - PC17(1) |

### Router-to-router links

| Link | Network | Mask | Side A | Side B |
|---|---|---|---|---|
| R1 - R2 | 10.0.0.0 | 255.255.255.252 (/30) | R1 Fa0/0 = 10.0.0.1 | R2 Fa0/0 = 10.0.0.2 |
| R2 - R3 | 10.0.0.4 | 255.255.255.252 (/30) | R2 Fa1/1 = 10.0.0.5 | R3 Fa0/0 = 10.0.0.6 |

---

## 4. Configuration

### 4.1 Interface configuration

**R1**
```
enable
configure terminal
hostname R1

interface FastEthernet1/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface FastEthernet0/1
 ip address 192.168.40.1 255.255.255.0
 no shutdown
interface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown
```

**R2**
```
enable
configure terminal
hostname R2

interface FastEthernet0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
interface FastEthernet1/0
 ip address 192.168.50.1 255.255.255.0
 no shutdown
interface FastEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
interface FastEthernet1/1
 ip address 10.0.0.5 255.255.255.252
 no shutdown
```

**R3**
```
enable
configure terminal
hostname R3

interface FastEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown
interface FastEthernet1/0
 ip address 192.168.60.1 255.255.255.0
 no shutdown
interface FastEthernet0/0
 ip address 10.0.0.6 255.255.255.252
 no shutdown
```

### 4.2 DHCP pools

**R1**
```
ip dhcp pool LAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
exit
ip dhcp pool LAN40
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
exit
```

**R2**
```
ip dhcp pool LAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
exit
ip dhcp pool LAN50
 network 192.168.50.0 255.255.255.0
 default-router 192.168.50.1
exit
```

**R3**
```
ip dhcp pool LAN30
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
exit
ip dhcp pool LAN60
 network 192.168.60.0 255.255.255.0
 default-router 192.168.60.1
exit
```

### 4.3 Routing between LANs

> **Note:** Ping between LANs on different routers requires routes. Replace this section with the routing you actually configured (static or RIP). Example with static routes:

```
! R1
ip route 192.168.20.0 255.255.255.0 10.0.0.2
ip route 192.168.50.0 255.255.255.0 10.0.0.2
ip route 192.168.30.0 255.255.255.0 10.0.0.2
ip route 192.168.60.0 255.255.255.0 10.0.0.2

! R2
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.40.0 255.255.255.0 10.0.0.1
ip route 192.168.30.0 255.255.255.0 10.0.0.6
ip route 192.168.60.0 255.255.255.0 10.0.0.6

! R3
ip route 192.168.10.0 255.255.255.0 10.0.0.5
ip route 192.168.40.0 255.255.255.0 10.0.0.5
ip route 192.168.20.0 255.255.255.0 10.0.0.5
ip route 192.168.50.0 255.255.255.0 10.0.0.5
```

### 4.4 PC configuration

On every PC: **Desktop > IP Configuration > select DHCP.**

---

## 5. Screenshots

| Description | Screenshot |
|---|---|
| R1 interface setup | ![R1 interfaces](screenshots/10-r1-interfaces.png) |
| R2 interface setup | ![R2 interfaces](screenshots/09-r2-interfaces.png) |
| R3 interface setup | ![R3 interfaces](screenshots/08-r3-interfaces.png) |
| R1 link to R2 | ![R1 link](screenshots/07-r1-link.png) |
| R2 links to R1 and R3 | ![R2 links](screenshots/06-r2-links.png) |
| R3 link to R2 | ![R3 link](screenshots/05-r3-link.png) |
| R1 DHCP pools | ![R1 DHCP](screenshots/04-r1-dhcp.png) |
| R2 DHCP pools | ![R2 DHCP](screenshots/03-r2-dhcp.png) |
| R3 DHCP pools | ![R3 DHCP](screenshots/02-r3-dhcp.png) |

---

## 6. Verification

Commands to check the setup:

```
show ip interface brief
show ip dhcp pool
show ip dhcp binding
show ip route
```

On a PC: `ipconfig` (Command Prompt) to confirm the address and gateway came from DHCP.

### Ping tests (all successful)

| Source | Destination | Protocol | Result |
|---|---|---|---|
| PC2 (LAN10) | PC0(1) (LAN40) | ICMP | Successful |
| PC8 (LAN20) | PC6(1) (LAN50) | ICMP | Successful |
| PC14 (LAN30) | PC12(1) (LAN60) | ICMP | Successful |

---

## 7. How DHCP Works Here (DORA)

1. **Discover** - the PC broadcasts a request for an IP address.
2. **Offer** - the router connected to that LAN offers an address from the matching pool.
3. **Request** - the PC accepts the offer.
4. **Acknowledge** - the router confirms the lease.

Because each router is directly connected to the LANs it serves, no DHCP relay (`ip helper-address`) is needed.

---

## 8. Possible Improvements

- Exclude the gateway and reserved addresses from the pools, e.g. `ip dhcp excluded-address 192.168.10.1 192.168.10.10`
- Add `dns-server 8.8.8.8` to each pool so clients can resolve names
- Replace static routes with RIP or OSPF
- Convert to a single switch per site using real VLANs (802.1Q) and router-on-a-stick

---

## 9. How to Open the Project

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free Networking Academy account required).
2. Download `dhcp_configured_multiple_lans.pkt` from this repository.
3. Open it in Packet Tracer and test with ping or by renewing DHCP on a PC.

---



## Author

**Bhagyesh Dhadiwal** - [GitHub](https://github.com/your-username)
