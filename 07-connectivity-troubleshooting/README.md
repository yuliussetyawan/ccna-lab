# Project 7: Connectivity Troubleshooting

## Project Overview
This project focuses on a systematic approach to network troubleshooting. The lab involves diagnosing a connectivity issue between two end-user PCs across a multi-router topology. Utilizing tools like `ping` and `traceroute`, the fault is isolated to a specific router, and the root cause—a missing OSPF network advertisement—is identified and resolved.

## Network Topology
The network consists of a 5-router enterprise topology (R1-R5) running OSPF, connecting various end-user subnets.

![Lab Topology](./topology.jpg)

---

## Lab Tasks & Configuration Logic

### Part 1: Fault Identification & Isolation

**1) Use ping to test connectivity from PC1 to PC3.**
*Answer:* Connectivity is down. The ping requests either time out or return "Destination host unreachable."
```cmd
C:\>ping 10.1.2.10

Pinging 10.1.2.10 with 32 bytes of data:
Request timed out.
Request timed out.
Reply from 10.1.0.1: Destination host unreachable.
Reply from 10.1.0.1: Destination host unreachable.

Ping statistics for 10.1.2.10:
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**2) Use traceroute to determine where the problem is likely to be.**
*Answer:* The traceroute successfully reaches hop 3 (`10.1.0.1` on R3) but fails to proceed further. This indicates that the routing break is likely at or just after R3, making it a good place to start troubleshooting.
```cmd
C:\>tracert 10.1.2.10

Tracing route to 10.1.2.10 over a maximum of 30 hops:
  1    0 ms    0 ms    0 ms  10.0.1.1
  2    0 ms    0 ms    0 ms  10.0.0.2
  3    0 ms    0 ms    0 ms  10.1.0.1
  4    * * * Request timed out.
```

---

### Part 2: Diagnosis & Resolution

**3) Determine the issue and fix it to restore connectivity between PC1 and PC3.**

**Diagnosis:**
Since traffic stops at R3, we first check R3's routing table to see if it knows how to reach the `10.1.2.0/24` network.
```bash
R3# show ip route
# Output shows connected routes for 10.1.0.0/24 and 10.1.1.0/24, but the 10.1.2.0/24 network is completely MISSING.
```
Next, check the running configuration on R3 to verify the OSPF routing setup.
```bash
R3# show run
...
router ospf 1
 network 10.1.0.0 0.0.0.255 area 0
 network 10.1.1.0 0.0.0.255 area 0
!
```
*Root Cause:* R3 is missing the OSPF network statement for the `10.1.2.0/24` subnet. It is not participating in OSPF for that interface, which means it isn't advertising the network to the rest of the topology, causing the "Destination host unreachable" error.

**Configuration Fix:**
Add the missing network statement to the OSPF process on R3.
```bash
R3# configure terminal
R3(config)# router ospf 1
R3(config-router)# network 10.1.2.0 0.0.0.255 area 0
```

**Final Verification:**
Test the ping from PC1 to PC3 again to confirm the fix was successful.
```cmd
C:\>ping 10.1.2.10

Pinging 10.1.2.10 with 32 bytes of data:
Reply from 10.1.2.10: bytes=32 time=1ms TTL=124
Reply from 10.1.2.10: bytes=32 time=0ms TTL=124
Reply from 10.1.2.10: bytes=32 time=0ms TTL=124
Reply from 10.1.2.10: bytes=32 time=0ms TTL=124

Ping statistics for 10.1.2.10:
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```
*Status:* ✅ **Successful.** Connectivity is fully restored across the OSPF domain.

---