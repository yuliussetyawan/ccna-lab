# Project 6: Dynamic Routing Protocols

## Project Overview

This project examines features common to all Interior Gateway Protocols (IGPs). The exercises involve configuring, verifying, and comparing Distance Vector (RIP), Link-State (OSPF), and Advanced Distance Vector (EIGRP) routing protocols. It also explores concepts such as equal-cost load balancing, administrative distance, floating static routes, loopback interfaces, and passive interfaces.

## Network Topology

The lab consists of routers (R1-R5) interconnected via FastEthernet links.

- **Routers:** 5x Cisco Routers (R1, R2, R3, R4, R5)
- **Routing Protocols Explored:** RIPv1, RIPv2, OSPF, EIGRP

![Lab Topology](./topology.jpg)

---

## Lab Tasks & Configuration Logic

### Part 1: Routing Protocol Updates (RIP)

**1) Provision a basic RIPv1 configuration and enable RIP on every interface.**

```bash
Router(config)# router rip
Router(config-router)# network 10.0.0.0
Router(config-router)# no auto-summary
```

**2) Debug the routing protocol updates on R1 with the 'debug ip rip' command. What kind of traffic is used?**
_Answer:_ The updates are being sent on the broadcast address `255.255.255.255`. All hosts on the subnet must process the packets.

```bash
R1# debug ip rip
# RIP: sending v1 update to 255.255.255.255
```

**3) Enable RIPv2 on every router.**

```bash
Router(config)# router rip
Router(config-router)# version 2
```

**4) What kind of traffic is used for the updates now?**
_Answer:_ The updates are being sent on the RIPv2 multicast address `224.0.0.9`. Only RIPv2 routers will process the packets beyond layer 3.

**5) Turn off all debugging on R1.**

```bash
R1# undebug all
```

**6) Check that RIP routes have been added to R1 and it has a route to every subnet.**

```bash
R1# show ip route
# Routes marked with 'R' indicate RIP.
```

**7) Why are there two routes to the 10.1.1.0/24 network in the routing table?**
_Answer:_ Two paths to `10.1.1.0/24` have an equal metric - a hop count of 2. Both routes are installed in the routing table and the router will perform Equal Cost Load Balancing between the next hops of 10.0.3.2 and 10.0.0.2.

---

### Part 2: Comparing Routing Protocols (RIP vs OSPF)

**8) View the RIP database on R1.**

```bash
R1# show ip rip database
```

**9) Provision a basic OSPF configuration and enable OSPF on every interface.**

```bash
Router(config)# router ospf 1
Router(config-router)# network 10.0.0.0 0.255.255.255 area 0
```

**10) Compare OSPF and RIP based on database structure.**
_Answer:_ RIP is a Distance Vector routing protocol so it only knows its directly connected neighbors and the lists of networks those neighbors have advertised. OSPF is a Link State routing protocol so it knows the state of every link on every router in its area.

**11) View the OSPF database on R1.**

```bash
R1# show ip ospf database
```

**12) Give OSPF time to converge. Are RIP routes included in the routing table on R1 now? Why or why not?**
_Answer:_ The RIP routes are replaced by OSPF because its Administrative Distance of 110 is preferred to RIP's AD of 120.

**13) Remove the OSPF configuration on each router.**

```bash
Router(config)# no router ospf 1
```

---

### Part 3: Administrative Distance & Floating Static Routes

**14) View the routing table on R1 again. Have the RIP routes been re-inserted?**
_Answer:_ Yes, now that the preferred OSPF routes have been removed, the RIP routes are re-inserted.

**15) Configure floating static routes on R1 to all 10.1.x.x networks via R2 with an AD of 95.**

```bash
R1(config)# ip route 10.1.0.0 255.255.255.0 10.0.0.2 95
R1(config)# ip route 10.1.1.0 255.255.255.0 10.0.0.2 95
R1(config)# ip route 10.1.2.0 255.255.255.0 10.0.0.2 95
R1(config)# ip route 10.1.3.0 255.255.255.0 10.0.0.2 95
```

**16) Are the static routes or the RIP routes in the routing table now? Why?**
_Answer:_ The static routes. They have an Administrative Distance of 95 which is preferred over RIP's 120.

**17) Shut down the FastEthernet 0/0 interface on R1.**

```bash
R1(config)# interface f0/0
R1(config-if)# shutdown
```

**18) What routes to the 10.1.x.x networks are in the routing table now? Why?**
_Answer:_ The RIP routes via R5. The static routes were removed from the routing table because the outgoing interface `FastEthernet 0/0` went down.

**19) Why is there only one route on R1 to the 10.1.1.0/24 network now?**
_Answer:_ Because the interface to R2 is down, the only valid path is via R5.

**20) Bring the FastEthernet 0/0 interface back up on R1.**

```bash
R1(config)# interface f0/0
R1(config-if)# no shutdown
```

**21) Notice that there are still only RIP routes to the 10.1.x.x networks in R1's routing table. Why is this?**
_Answer:_ Interface FastEthernet 0/0 on R2 is still shut down so no routes go through it.

**22) Make the required change so that the floating static routes to 10.1.x.x are preferred again.**

```bash
R2(config)# interface f0/0
R2(config-if)# no shutdown
```

**23) Verify the static routes are back in R1's routing table.**

```bash
R1# show ip route
# 'S' routes are now present.
```

**24) Remove the static routes you just configured on R1.**

```bash
R1(config)# no ip route 10.1.0.0 255.255.255.0 10.0.0.2 95
R1(config)# no ip route 10.1.1.0 255.255.255.0 10.0.0.2 95
R1(config)# no ip route 10.1.2.0 255.255.255.0 10.0.0.2 95
R1(config)# no ip route 10.1.3.0 255.255.255.0 10.0.0.2 95
```

---

### Part 4: Advanced Administrative Distance Config

**25) What is the default Administrative Distance of EIGRP internal routes?**
_Answer:_ 90

**26) Configure a floating summary static route on R1 to the 10.1.0.0/16 networks via R2.**

```bash
R1(config)# ip route 10.1.0.0 255.255.0.0 10.0.0.2 95
```

**27) Configure another floating summary static route on R1 to the 10.0.0.0/16 networks via R5.**

```bash
R1(config)# ip route 10.0.0.0 255.255.0.0 10.0.3.2 95
```

_(Note: As per lab logic, AD 95 is used to prepare for EIGRP later, ensuring static routes aren't preferred over EIGRP)._

**28) What changes do you expect to see to the routing table on R1?**
_Answer:_ No changes. The RIP routes have a more specific `/24` prefix length. Based on the **Longest Prefix Match** rule, the `/24` RIP routes will be preferred over the `/16` static summary routes, even though the static routes have a better Administrative Distance.

**29) Verify the expected behavior on R1.**

```bash
R1# show ip route
# The routing table still uses the more specific RIP /24 routes.
```

**30) Remove the static summary route to 10.0.0.0/16 on R1.**

```bash
R1(config)# no ip route 10.0.0.0 255.255.0.0 10.0.3.2 95
```

**31) Trace the route from PC1 to PC3.**

```cmd
C:\> tracert 10.1.2.10
# Traces through R1, R2, R3, R4.
```

**32) Provision a basic EIGRP configuration on R1 to R4.**

```bash
Router(config)# router eigrp 100
Router(config-router)# no auto-summary
Router(config-router)# network 10.0.0.0 0.255.255.255
```

**33) Give EIGRP time to converge. Trace the route from PC1 to PC3 again.**

```cmd
C:\> tracert 10.1.2.10
```

**34) Has the path changed? Why or why not?**
_Answer:_ Yes. The path now goes via R1 > R5 > R4. When the routes were learned via RIP, the path was via R2 because it had a lower hop count. Now that the routes are learned via EIGRP, the path is via R5 because those links have a higher bandwidth (FastEthernet vs Ethernet).

**35) Remove the EIGRP configuration from R1 to R4.**

```bash
Router(config)# no router eigrp 100
```

**36) Why did we not enable EIGRP on R5 previously?**
_Answer:_ To observe how the introduction of EIGRP on the other routers changed the path selection away from R2 towards R5 based on EIGRP's bandwidth/delay metric compared to RIP's hop count.

**37) Provision a basic EIGRP configuration and enable EIGRP on R5.**

```bash
R5(config)# router eigrp 100
R5(config-router)# no auto-summary
R5(config-router)# network 10.0.0.0 0.255.255.255
```

---

### Part 5: Loopback and Passive Interfaces

**38) Configure loopback interface 0 on each router.**

```bash
R1(config)# interface loopback 0
R1(config-if)# ip address 192.168.0.1 255.255.255.255
# Repeat for R2 (.2), R3 (.3), R4 (.4), R5 (.5)
```

**39) Give the routing protocols time to converge. Are the loopback interfaces reachable from the other routers?**
_Answer:_ No. They have not been added to a routing protocol yet.

**40) Include the loopback interfaces in EIGRP.**

```bash
Router(config)# router eigrp 100
Router(config-router)# network 192.168.0.0 0.0.0.255
```

**41) Verify the EIGRP neighbors on R1.**

```bash
R1# show ip eigrp neighbors
```

**42) What command is used to prevent EIGRP from sending Hello packets out of an interface?**

```bash
Router(config-router)# passive-interface <interface>
```

**43) Configure R1 so it does not send EIGRP Hello packets down to the PCs.**

```bash
R1(config)# router eigrp 100
R1(config-router)# passive-interface f0/1
R1(config-router)# passive-interface f1/0
```

**44) Verify that traffic from R5 to the directly connected interfaces on R1 goes via the FastEthernet 0/1 interface.**

```bash
R5# show ip route
# Look for the 'D' (EIGRP) routes for R1's networks.
```

**45) Shut down the FastEthernet 0/1 interface on R5.**

```bash
R5(config)# interface f0/1
R5(config-if)# shutdown
```

**46) Notice that the connection from R5 to R1 is now via the static summary route through R4.**

```bash
R5# show ip route
# The routing table uses the 'S' summary route because the direct EIGRP path is down.
```

**47) Bring the interface back up and verify the preferred routes are reinstated.**

```bash
R5(config)# interface f0/1
R5(config-if)# no shutdown
R5# show ip route
```

---
