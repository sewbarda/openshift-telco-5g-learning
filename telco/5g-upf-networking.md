Paste this:
# 5G Core Cloud-Native Networking

## Overview

This section documents the transition from traditional NFV/VNF
architectures to cloud-native Network Functions (CNFs) running on
Kubernetes/OpenShift.

The focus is on 5G Core networking and the UPF (User Plane Function).

---

# 1. Traditional NFV / VNF Architecture

A simplified traditional telecom virtualisation model:

```text
Physical Infrastructure
        |
        v
 VMware / OpenStack
        |
        v
       VM
        |
        v
    Guest OS
        |
        v
       VNF

The VNF typically operates inside a virtual machine with virtual
network interfaces.
2. Cloud-Native CNF Architecture
A cloud-native network function can be deployed as a Kubernetes
workload.
Simplified architecture:
Infrastructure
      |
     Linux
      |
  OpenShift
      |
     Pod
      |
  Container
      |
     CNF

A CNF is not simply a container. It can consist of multiple
containers, Pods, networking resources, configuration and other
Kubernetes/OpenShift resources.
3. 5G UPF
The UPF (User Plane Function) is responsible for forwarding
5G user-plane traffic.
Simplified traffic path:
UE
 |
 v
gNB
 |
 | N3
 v
UPF
 |
 | N6
 v
Data Network

4. N3 Interface
N3 connects the:
gNB <---- N3 ----> UPF

N3 carries 5G user-plane traffic between the RAN and the UPF.
5. N4 Interface
N4 connects:
SMF <---- N4 / PFCP ----> UPF

The SMF uses PFCP to control the UPF and establish/manage
user-plane forwarding behaviour.
N4 is a control/session-management interface rather than the
high-volume user-plane path.
6. N6 Interface
N6 connects:
UPF <---- N6 ----> Data Network

The Data Network may provide access to external networks or
the Internet.
7. UPF Network Interfaces
A simplified cloud-native UPF may require multiple network
attachments.
Conceptually:
                     UPF Pod
                        |
                      Multus
                        |
            +-----------+-----------+
            |           |           |
           eth0        net1        net2
            |           |           |
       Management       N3          N6

The exact network design depends on the CNF implementation,
hardware and deployment architecture.
8. Multus
Multus is a CNI meta-plugin that allows a Pod to have multiple
network attachments/interfaces.
Instead of:
Pod
 |
eth0
 |
Default Pod Network

a telco workload can conceptually have:
Pod
 |
Multus
 |
+-------+-------+
|       |       |
eth0   net1    net2
 |      |       |
Mgmt    N3      N6

9. SR-IOV
SR-IOV (Single Root I/O Virtualization) allows an SR-IOV-capable
physical NIC to expose Virtual Functions (VFs).
Conceptually:
Physical NIC
      |
      PF
      |
+-----+-----+-----+
|     |     |     |
VF1   VF2   VF3   VF4

Virtual Functions can be allocated to workloads requiring
high-performance networking.
10. PF and VF
PF — Physical Function
The PF represents the physical NIC function and provides
management/configuration capabilities for the device and
its Virtual Functions.
VF — Virtual Function
A VF is a virtualised function provided by an SR-IOV-capable
physical NIC.
A VF can be assigned to a workload.
Example:
Physical NIC
     |
     PF
     |
     VF
     |
    UPF

11. SR-IOV Operator
OpenShift provides an SR-IOV Network Operator for managing
SR-IOV networking resources.
Conceptually:
OpenShift
    |
SR-IOV Network Operator
    |
Physical NIC
    |
    PF
    |
   VFs

The operator can manage SR-IOV resources on selected nodes
and expose them as allocatable resources.
12. SriovNetworkNodePolicy
A SriovNetworkNodePolicy defines how SR-IOV resources should
be configured on selected nodes.
A policy can define information such as:
- Nodes to which the policy applies
- Physical Function / NIC
- Number of Virtual Functions
- Driver
- Resource name
Conceptually:
SriovNetworkNodePolicy
          |
          v
   SR-IOV Operator
          |
          v
      Physical NIC
          |
          v
       PF / VFs

13. DPDK
DPDK (Data Plane Development Kit) provides optimised mechanisms
for high-performance packet processing.
Simplified architecture:
NIC
 |
SR-IOV VF
 |
DPDK
 |
User Plane Application

DPDK is particularly relevant to high-performance user-plane
workloads such as the 5G UPF.
14. Multus + SR-IOV + DPDK
These technologies solve different problems.
Multus
  |
  +-- Multiple network attachments

SR-IOV
  |
  +-- High-performance NIC Virtual Functions

DPDK
  |
  +-- High-performance packet processing

A conceptual UPF architecture:
                         UPF CNF
                            |
                          Multus
                            |
                 +----------+----------+
                 |                     |
                N3                    N6
                 |                     |
              SR-IOV                SR-IOV
                 |                     |
                VF                    VF
                 |                     |
               DPDK                  DPDK

This is a conceptual architecture. Actual production
implementations depend on the CNF vendor, hardware,
network architecture and OpenShift configuration.
15. CPU Pinning, Hugepages and NUMA
High-performance telco workloads can also require additional
optimisation.
CPU Pinning
Dedicated CPU cores can be assigned to latency-sensitive
workloads.
CPU 0 -> Operating System
CPU 1 -> Operating System
CPU 2 -> UPF
CPU 3 -> UPF

Hugepages
Large memory pages can be used by high-performance workloads.
Examples include:
4 KB
2 MB
1 GB

NUMA
NUMA (Non-Uniform Memory Access) describes the relationship
between CPUs, memory and hardware devices.
For high-performance workloads, CPU, memory and NIC placement
can be important.
16. VNF to CNF Evolution
Traditional:
VM
 |
Guest OS
 |
VNF
 |
vNIC

Cloud-native:
OpenShift
 |
Pod
 |
CNF
 |
Multus
 |
SR-IOV VF
 |
DPDK

The cloud-native architecture introduces Kubernetes/OpenShift
orchestration and cloud-native networking mechanisms while
retaining the telecom requirement for high-performance,
predictable network processing.
Key Takeaways
Multus
Multiple network attachments.
SR-IOV
High-performance Virtual Functions from a physical NIC.
DPDK
High-performance packet processing.
N3
gNB ↔ UPF user plane.
N4
SMF ↔ UPF control/session management using PFCP.
N6
UPF ↔ Data Network.
UPF
5G User Plane Function responsible for forwarding user-plane
traffic.
