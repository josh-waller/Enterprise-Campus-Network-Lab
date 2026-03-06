# **Enterprise Campus Design Lab (BRKENS‑2031)**

Implemented a multi‑VLAN, multilayer enterprise campus using routed core–distribution links, Rapid PVST+, HSRP, EIGRP, DHCP, NAT, and access‑layer security controls, following the Cisco Live BRKENS‑2031 design principles.
The Network will follow this design: https://www.ciscolive.com/c/dam/r/ciscolive/emea/docs/2023/pdf/BRKENS-2031.pdf

---

## **L3 Port‑Channel and VLANs**

Built a Layer‑3 Port‑Channel between the core switches to provide increased bandwidth and fast convergence without relying on Spanning Tree.

Configured VLANs 3, 5, 9, 14 and 34, 37, 42, 46 across the campus, using them as separate broadcast domains for different user groups and services.

---

## **SVIs, DHCP, and HSRP**

Created SVIs for each VLAN on the distribution switches, using the 192.168.X.0/24 scheme with:

- Virtual gateway IPs: 192.168.X.1
- Physical distribution switch IPs: 192.168.X.2 and 192.168.X.3

Configured DHCP on the distribution layer by:

- Excluding 192.168.X.1–192.168.X.3 for all VLANs 3, 5, 9, 14, 34, 37, 42, 46
- Creating DHCP pools `VLAN3`, `VLAN5`, `VLAN9`, `VLAN14`, `VLAN34`, `VLAN37`, `VLAN42`, `VLAN46` with gateway 192.168.X.1, domain `JMW.com`, and DNS 8.8.8.8

Implemented HSRP on each VLAN using the above addressing format to provide default‑gateway redundancy and, in the data centre, two HSRP instances for gateway load‑balancing.

---

## **Trunking and STP Root Manipulation**

Used routed ports (no switchport) between core and distribution to remove the need for STP there and to allow optimal load balancing and fast convergence.

On VLAN trunks where needed, aligned HSRP primaries with the STP root bridges so that the active gateway is also the Layer‑2 root for efficient traffic flows.

---

## **Access Layer Configuration**

On the access‑layer switches, configured user‑facing ports with:

- `switchport mode access` and `nonegotiate` to prevent unwanted trunking
- Port security with sticky MAC, maximum 2 MAC addresses, violation mode `restrict`
- BPDU Guard and STP PortFast to protect STP and speed host convergence
- Storm control to limit broadcast/multicast storms

These controls enforced **port‑level security** and protected the campus from common Layer‑2 attacks and misconfigurations.

---

## **EIGRP, NAT, and Route Summarization**

Enabled EIGRP between distribution and core, advertising VLAN and inter‑switch networks.

Summarized routes on the distribution layer towards the core to reduce routing table size and CPU load at the core.

Implemented NAT at the edge so internal VLAN subnets could access external networks while hiding internal addressing.

---

## **Design Notes and Behaviours**

- Core–distribution links as **router ports (no switchport)** eliminated STP in the core path, enabling faster convergence and more deterministic routing.
- Core‑to‑core **Layer‑3 Port‑Channel** increased bandwidth and resilience while keeping the control plane simple.
- Distribution switches acted as DHCP servers for access VLANs, centralizing address management at the distribution tier.
- All switches ran **Rapid PVST+** to provide per‑VLAN spanning tree with improved convergence times.
- Summarizing distribution routes towards the core reduced SPF and CPU load, making the design more scalable.

---

## **Final Commands Example (DHCP and STP Mode)**

Example of DHCP and spanning‑tree configuration work done on the distribution switches:

- `ip dhcp excluded-address 192.168.X.1 192.168.X.3` for all listed VLAN subnets
- `ip dhcp pool VLANX` with network 192.168.X.0/24, default‑router 192.168.X.1, domain `JMW.com`, DNS 8.8.8.8
- `spanning-tree mode rapid-pvst` to enable Rapid STP across the campus

These steps tied addressing, redundancy, and fast convergence together in line with the BRKENS‑2031 campus design guidance
