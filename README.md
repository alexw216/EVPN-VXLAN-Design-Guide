# **EVPN & VXLAN Design Guide**  
*A comprehensive reference covering VXLAN use cases, EVPN route‑type mapping, decision logic, and a troubleshooting matrix.*

---

## **1. VXLAN Use Case Scenarios**

VXLAN enables scalable, flexible, and multi‑tenant network fabrics by extending Layer‑2 networks over a routed Layer‑3 underlay. The following scenarios represent the most common and impactful real‑world applications.

### **Datacenter Virtualization & Mobility**
- L2 adjacency for virtual machines and containers.
- Rack‑independent VLANs enabling workload placement anywhere.
- Host mobility with deterministic MAC/IP learning.

### **Layer‑2 Extension Across Layer‑3 Underlays**
- Stretching L2 segments across routed fabrics.
- Multi‑building or multi‑floor campus extension.
- Metro‑scale L2 extension for active‑active datacenters.

### **Multi‑Tenant Segmentation**
- 16M VNIs for large‑scale tenant isolation.
- Per‑application segmentation.
- VRF‑per‑tenant architectures.

### **EVPN Control‑Plane Enhancements**
- Control‑plane MAC learning.
- ARP/ND suppression.
- EVPN multihoming (ESI‑LAG).
- Deterministic convergence.

### **Hybrid Cloud & Multi‑Site**
- On‑prem ↔ cloud bridging.
- Disaster recovery site extension.
- Cloud bursting.

### **Security & Zero‑Trust**
- Microsegmentation using VNIs and VRFs.
- Service chaining for firewalls and load balancers.
- Identity‑based segmentation.

### **Lab & Simulation**
- Virtual fabrics in EVE‑NG, CML, GNS3.
- Multi‑tenant test environments.

### **Multi‑Site Mobility & Active‑Active DCs**
- Host mobility across sites.
- Stretched L2/L3 services.
- Active‑active datacenter designs.

### **Network Virtualization & SDN**
- Logical/physical decoupling.
- SDN‑driven overlays (ACI, NSX‑T, Apstra).
- Tenant self‑service provisioning.

### **Campus Fabric Modernization**
- Eliminating spanning tree.
- Unified wired/wireless segmentation.
- Scalable campus core fabrics.

### **Service Provider & Carrier**
- Wholesale L2 services.
- Mobile backhaul segmentation.
- Multi‑tenant hosting.

---

## **2. EVPN Route‑Type Mapping**

EVPN uses BGP to distribute MAC, IP, and service information. Each scenario maps to specific route types:

| Scenario | Required Route Types |
|---------|----------------------|
| L2 bridging | RT‑2, RT‑3 |
| L3 routing (VRF/L3VNI) | RT‑5 |
| Host mobility | RT‑2 (with sequence numbers) |
| ARP/ND suppression | RT‑2 (with IP binding) |
| Multihoming | RT‑1, RT‑4, RT‑2 |
| Multi‑site L2 | RT‑2, RT‑3 |
| Multi‑site L3 | RT‑5 |
| Service chaining | RT‑2, RT‑5 |
| SP L2 services | RT‑2, RT‑3 |
| SP L3 services | RT‑5 |
| Multicast (PIM‑less) | RT‑7 |

**Route Type Summary**
- **RT‑1** — Ethernet Auto‑Discovery (multihoming, DF election)  
- **RT‑2** — MAC/IP Advertisement (L2VNI, mobility, ARP suppression)  
- **RT‑3** — Inclusive Multicast Ethernet Tag (BUM replication)  
- **RT‑4** — Ethernet Segment Route (multihoming split‑horizon)  
- **RT‑5** — IP Prefix Route (L3VNI/VRF routing)  
- **RT‑7** — Multicast Route (vendor‑specific PIM‑less multicast)

---

## **3. EVPN Route‑Type Decision Tree**

This decision tree determines which EVPN route types are required for any VXLAN design.

### **1. Are you providing L2 services?**
Use **RT‑2** and **RT‑3**.

### **2. Are you providing L3 services?**
Use **RT‑5**.

### **3. Are you doing EVPN multihoming?**
Use **RT‑1**, **RT‑4**, and **RT‑2**.

### **4. Do you need host mobility tracking?**
Use **RT‑2** with mobility sequence numbers.

### **5. Do you need ARP/ND suppression?**
Use **RT‑2** with IP bindings.

### **6. Are you stretching services across sites?**
- L2 stretch → **RT‑2**, **RT‑3**  
- L3 stretch → **RT‑5**  
- Mobility → **RT‑2**

### **7. Are you doing multicast without PIM?**
Use **RT‑7**.

### **8. Are you inserting firewalls or load balancers?**
Use **RT‑5** (and **RT‑2** if L2 segments involved).

### **9. Are you using SDN controllers?**
Use **RT‑2**, **RT‑5**, and optionally **RT‑1/RT‑4**.

### **10. Are you a service provider delivering L2/L3 services?**
- L2 → **RT‑2**, **RT‑3**  
- L3 → **RT‑5**  
- Multihoming → **RT‑1**, **RT‑4**

---

## **4. EVPN Troubleshooting Matrix**

A symptom‑driven guide for diagnosing EVPN/VXLAN issues.

| Symptom | Likely Issue | Route Types / Constructs | What to Verify |
|--------|--------------|--------------------------|----------------|
| Hosts in same VNI cannot communicate | Missing L2 EVPN routes | RT‑2, RT‑3 | RT‑2/RT‑3 present? RT import/export correct? |
| VTEPs do not build VXLAN tunnels | Missing IMET routes | RT‑3 | RT‑3 advertised? Same RT across VTEPs? |
| ARP flooding, no suppression | Missing IP bindings | RT‑2 | RT‑2 includes IP? ARP‑suppress enabled? |
| Inter‑VNI routing fails | L3VNI not advertising prefixes | RT‑5 | L3VNI bound to VRF? RT‑5 present? |
| Dual‑homed server flaps or unreachable | Incomplete multihoming | RT‑1, RT‑4, RT‑2 | ESI consistent? DF election correct? |
| Blackholing after vMotion | Mobility not converging | RT‑2 | Sequence numbers correct? Stale RT‑2 filtered? |
| Some VNIs work, others fail | Per‑VNI RT mismatch | RT‑2, RT‑3, RT‑5 | RTs consistent across VTEPs? |
| Multicast receivers silent | Missing multicast routes | RT‑7 | RT‑7 supported and advertised? |
| Traffic fails across DCs | Multisite RT filtering | RT‑2, RT‑5 | Border gateways re‑advertising RTs? |
| No EVPN routes at all | Control‑plane down | All | BGP EVPN AFI/SAFI enabled? Underlay reachability? |

---

## **5. Quick Reference Summary**

- **L2VNI** → RT‑2 + RT‑3  
- **L3VNI/VRF** → RT‑5  
- **Multihoming** → RT‑1 + RT‑4 + RT‑2  
- **Mobility** → RT‑2 with sequence numbers  
- **ARP suppression** → RT‑2 with IP bindings  
- **Multisite** → RT‑2/RT‑3 (L2), RT‑5 (L3)  
- **Multicast** → RT‑7  

---

# EVPN & VXLAN Design Guide (Diagram‑Rich Edition)

A comprehensive reference covering VXLAN use cases, EVPN route‑type mapping, decision logic, and a troubleshooting matrix — enhanced with ASCII diagrams suitable for GitHub.

---

## 1. VXLAN Use Case Scenarios

VXLAN enables scalable, flexible, and multi‑tenant network fabrics by extending Layer‑2 networks over a routed Layer‑3 underlay.

### Datacenter Virtualization & Mobility

+---------------- Spine Layer ----------------+
|                 EVPN/BGP                    |
+-------+-------+                           +--------+-------+
|   Leaf 1      |                           |    Leaf 2      |
| VTEP 10.1.1.1 |                           | VTEP 10.1.1.2   |
+-------+-------+                           +--------+-------+
|                                              |
+--+--+                                        +--+--+
| VM |   <---- L2VNI 1001 (stretched) ---->     | VM |
+-----+                                        +-----+


### Layer‑2 Extension Across Layer‑3 Underlays

+-----------+       L3 Underlay       +-----------+
|  Site A   |--------------------------|  Site B   |
| L2VNI 10  |<------ VXLAN/EVPN ------>| L2VNI 10  |
+-----------+                          +-----------+


### Multi‑Tenant Segmentation

Tenant A: VNI 10010  --> VRF-A
Tenant B: VNI 10020  --> VRF-B
Tenant C: VNI 10030  --> VRF-C


### EVPN Control‑Plane Enhancements

MAC/IP Learning:
Host MAC/IP ---> RT-2 ---> All VTEPs

BUM Replication:
IMET Route ---> RT-3 ---> Flood List


### Hybrid Cloud & Multi‑Site

On-Prem DC ---------------- Cloud VPC
\                         /
\--- EVPN/VXLAN Tunnel--/


### Security & Zero‑Trust

+---------+      +-----------+      +---------+
| TenantA |----->| Firewall  |----->| TenantB |
+---------+      +-----------+      +---------+
L2/L3 segmentation via VNIs + VRFs


---

## 2. EVPN Route‑Type Mapping

| Scenario | Required Route Types |
|---------|----------------------|
| L2 bridging | RT‑2, RT‑3 |
| L3 routing (VRF/L3VNI) | RT‑5 |
| Host mobility | RT‑2 (with sequence numbers) |
| ARP/ND suppression | RT‑2 (with IP binding) |
| Multihoming | RT‑1, RT‑4, RT‑2 |
| Multi‑site L2 | RT‑2, RT‑3 |
| Multi‑site L3 | RT‑5 |
| Service chaining | RT‑2, RT‑5 |
| SP L2 services | RT‑2, RT‑3 |
| SP L3 services | RT‑5 |
| Multicast (PIM‑less) | RT‑7 |

---

## 3. EVPN Route‑Type Decision Tree (ASCII)


+----------------------+
|  Start: VXLAN Fabric |
+----------+-----------+
|
v
+----------------+----------------+
| Are you providing L2 bridging? |
+----------------+----------------+
|
Yes          |          No
v           |           v
+--------+-------+   |   +-------+--------+
| Use RT-2, RT-3 |   |   | Are you routing |
+----------------+   |   | (VRF/L3VNI)?    |
|   +-------+--------+
|           |
|    Yes    |    No
|     v     |     v
| +---+-----+--+  |
| | Use RT-5   |  |
| +------------+  |
|                 |
|                 |
v                 v
+-----------------------------------------------+
| Do you need EVPN Multihoming (ESI-LAG)?       |
+----------------------+--------------------------+
|
Yes     |      No
v      |       v
+-----------+------+--+   |
| Use RT-1, RT-4, RT-2 |   |
+-----------------------+   |
|
v
+-------------------------------------------+
| Do you need host mobility tracking?       |
+----------------------+----------------------+
|
Yes     |      No
v      |       v
+-----------+------+--+   |
| Use RT-2 (mobility) |   |
+----------------------+   |
|
v
+-------------------------------------------+
| Do you need ARP/ND suppression?           |
+----------------------+----------------------+
|
Yes     |      No
v      |       v
+-----------+------+--+   |
| Use RT-2 (IP+MAC) |     |
+-------------------+     |
|
v
+-------------------------------------------+
| Do you need multicast without PIM?        |
+----------------------+----------------------+
|
Yes     |      No
v      |       v
+-----------+------+--+   |
| Use RT-7           |    |
+--------------------+    |
|
v
+-------------------------------------------+
| End: Route Types Determined               |
+-------------------------------------------+


---

## 4. EVPN Troubleshooting Matrix

| Symptom | Likely Issue | Route Types / Constructs | What to Verify |
|--------|--------------|--------------------------|----------------|
| Hosts in same VNI cannot communicate | Missing L2 EVPN routes | RT‑2, RT‑3 | RT‑2/RT‑3 present? RT import/export correct? |
| VTEPs do not build VXLAN tunnels | Missing IMET routes | RT‑3 | RT‑3 advertised? Same RT across VTEPs? |
| ARP flooding, no suppression | Missing IP bindings | RT‑2 | RT‑2 includes IP? ARP‑suppress enabled? |
| Inter‑VNI routing fails | L3VNI not advertising prefixes | RT‑5 | L3VNI bound to VRF? RT‑5 present? |
| Dual‑homed server flaps or unreachable | Incomplete multihoming | RT‑1, RT‑4, RT‑2 | ESI consistent? DF election correct? |
| Blackholing after vMotion | Mobility not converging | RT‑2 | Sequence numbers correct? Stale RT‑2 filtered? |
| Some VNIs work, others fail | Per‑VNI RT mismatch | RT‑2, RT‑3, RT‑5 | RTs consistent across VTEPs? |
| Multicast receivers silent | Missing multicast routes | RT‑7 | RT‑7 supported and advertised? |
| Traffic fails across DCs | Multisite RT filtering | RT‑2, RT‑5 | Border gateways re‑advertising RTs? |
| No EVPN routes at all | Control‑plane down | All | BGP EVPN AFI/SAFI enabled? Underlay reachability? |

---

## 5. Quick Reference Summary

- L2VNI → RT‑2 + RT‑3  
- L3VNI/VRF → RT‑5  
- Multihoming → RT‑1 + RT‑4 + RT‑2  
- Mobility → RT‑2 with sequence numbers  
- ARP suppression → RT‑2 with IP bindings  
- Multisite → RT‑2/RT‑3 (L2), RT‑5 (L3)  
- Multicast → RT‑7  

---

## A full ASCII EVPN/VXLAN fabric topology.
It shows:

Dual‑spine, multi‑leaf fabric

L2VNI + L3VNI

Multihoming with ESI

Multi‑tenant VRFs

A simple WAN/DR extension

                         ┌───────────────────────────────────────┐
                         │           CORE / WAN / DR             │
                         │         (MPLS / IP Backbone)          │
                         └───────────────┬───────────────────────┘
                                         │
                                         │
                             +-----------+-----------+
                             |     DC BORDER GW      |
                             |  EVPN GW / DCI VTEP   |
                             |  L3VNI, RT-5 export   |
                             +-----------+-----------+
                                         │
                                         │
                 =========================================================
                 =                 VXLAN EVPN FABRIC                     =
                 =========================================================

                         ┌──────────── Spine Layer ────────────┐
                         │                                     │
                 +───────+────────+                   +────────+───────+
                 |    SPINE 1     |                   |    SPINE 2     |
                 |  EVPN/BGP RR   |                   |  EVPN/BGP RR   |
                 +--------+-------+                   +--------+-------+
                          |                                     |
          -----------------+-----------------   -----------------+-----------------
          |                                 |   |                                 |
          |                                 |   |                                 |
   +------+--------+                 +------+--------+                 +----------+------+
   |    LEAF 1     |                 |    LEAF 2     |                 |    LEAF 3      |
   |  VTEP 10.0.0.1|                 |  VTEP 10.0.0.2|                 |  VTEP 10.0.0.3 |
   |               |                 |               |                 |                |
   | VRF-TENANT-A  |                 | VRF-TENANT-A  |                 | VRF-TENANT-B   |
   |  L3VNI 50010  |                 |  L3VNI 50010  |                 |  L3VNI 50020   |
   +------+--------+                 +------+--------+                 +--------+-------+
          |                                 |                                  |
          |                                 |                                  |
   -------+--------                   -------+--------                  -------+--------
   |              |                   |              |                  |              |
   |              |                   |              |                  |              |
+--+--+       +---+---+           +---+---+      +---+---+          +---+---+      +---+---+
| VM1 |       | VM2   |           | APP1  |      | APP2  |          | DB1  |      | DB2  |
| A   |       | A     |           | A     |      | A     |          | B    |      | B    |
+-----+       +-------+           +-------+      +-------+          +-------+      +------+
  |              |                    |              |                 |               |
  | L2VNI 10010  |                    | L2VNI 10010  |                 | L2VNI 10020   |
  | (Tenant A)   |                    | (Tenant A)   |                 | (Tenant B)    |
  +--------------+--------------------+--------------+-----------------+---------------+


        ┌─────────────────────────────────────────────────────────────────────┐
        │                    EVPN MULTIHOMING EXAMPLE                         │
        └─────────────────────────────────────────────────────────────────────┘

                 +------------------- LAG / ESI ETHERNET SEGMENT -------------------+
                 |                                                                  |
          +------+--------+                                                +--------+------+
          |    LEAF 1     |                                                |    LEAF 2     |
          |  VTEP 10.0.0.1|                                                |  VTEP 10.0.0.2|
          |  ESI 0001     |                                                |  ESI 0001     |
          +------+--------+                                                +--------+------+
                 |                                                                  |
                 |                                                                  |
                 +-------------------+    MULTI-HOMED SERVER    +-------------------+
                                     |  (Active/Active NICs)   |
                                     +-------------------------+


        ┌─────────────────────────────────────────────────────────────────────┐
        │                    TENANT / VRF / VNI MAPPING                        │
        └─────────────────────────────────────────────────────────────────────┘

Tenant A:
  - VRF-TENANT-A
  - L3VNI: 50010
  - L2VNIs:
      * 10010: APP/WEB segment
      * 10011: DB segment

Tenant B:
  - VRF-TENANT-B
  - L3VNI: 50020
  - L2VNIs:
      * 10020: APP/WEB segment
      * 10021: DB segment


        ┌─────────────────────────────────────────────────────────────────────┐
        │                    EVPN ROUTE-TYPE FLOW (SIMPLIFIED)                │
        └─────────────────────────────────────────────────────────────────────┘

Host Attachment (Tenant A, L2VNI 10010):
  LEAF 1 learns MAC/IP:
    - MAC: aa:aa:aa:aa:aa:01
    - IP : 10.10.10.11
  → Advertises RT-2 (MAC/IP) into EVPN

BUM Handling (ARP, unknown unicast):
  LEAF 1 advertises RT-3 (IMET) for VNI 10010
  Other VTEPs join flood list for VNI 10010

Inter-Subnet Routing (Tenant A):
  LEAFs advertise prefixes for VRF-TENANT-A via RT-5
  L3VNI 50010 used for routing between L2VNIs 10010 and 10011

Multihoming:
  LEAF 1 and LEAF 2 advertise:
    - RT-1: ESI auto-discovery
    - RT-4: Ethernet Segment route
  Host MAC/IP still via RT-2


        ┌─────────────────────────────────────────────────────────────────────┐
        │                    DC TO WAN / DR SITE (HIGH LEVEL)                 │
        └─────────────────────────────────────────────────────────────────────┘

                 +-------------------+          +-------------------+
                 |   DC BORDER GW    |          |   DR BORDER GW    |
                 |  EVPN / L3VNI     |          |  EVPN / L3VNI     |
                 +---------+---------+          +---------+---------+
                           |                              |
                           |   IP/MPLS / EVPN / RT-5      |
                           +--------------+---------------+
                                          |
                                          v
                                 WAN / DCI / MPLS Core
