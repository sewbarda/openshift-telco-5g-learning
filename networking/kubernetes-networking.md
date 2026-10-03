
# Kubernetes / OpenShift Networking Lab

## Objective

Investigate how OpenShift provides networking to Pods and how
Kubernetes Services connect to Pod endpoints.

---

## 1. Pod Networking

A running `yaml-demo` Pod was inspected using:

```bash
oc describe pod yaml-demo-9cdb87f9f-rkqfk
The Pod was running on the following OpenShift node:
Node:
ip-10-0-15-56.ec2.internal

Node IP:
10.0.15.56

The Pod received:
Pod IP:
10.128.9.179

The Pod's primary network interface was:
Interface:
eth0

The network was:
OVN-Kubernetes

2. CNI / Multus
The Pod events showed:
AddedInterface
Add eth0 [10.128.9.179/23] from ovn-kubernetes

This demonstrated that Multus was involved in the network
attachment process and that OVN-Kubernetes provided the default
Pod network.
The network-status annotation showed:
name: ovn-kubernetes
interface: eth0
ip: 10.128.9.179
default: true

3. Pod Network Architecture
Conceptually:
OpenShift Node
10.0.15.56
      |
      v
+----------------------+
| Pod: yaml-demo       |
|                      |
| eth0                 |
| 10.128.9.179/23      |
+----------+-----------+
           |
           v
    OVN-Kubernetes

4. Kubernetes Service
A ClusterIP Service was created for my-first-app.
Command:
oc get svc my-first-app -o wide

Observed:
Service:
my-first-app

Type:
ClusterIP

Cluster IP:
172.30.160.238

Port:
8080

Selector:
app=my-first-app

The Service provides a stable logical endpoint for the application
while the underlying Pods can change.
5. EndpointSlice
The Service's EndpointSlice was inspected using:
oc get endpointslice \
  -l kubernetes.io/service-name=my-first-app -o wide

Observed endpoints:
10.131.0.19
10.131.10.130
10.128.9.172

These corresponded to the three running application Pods.
6. Service → EndpointSlice → Pods
The observed architecture was:
                 Service
           172.30.160.238:8080
                     |
                     v
               EndpointSlice
                     |
          +----------+----------+
          |          |          |
          v          v          v
    10.131.0.19  10.131.10.130  10.128.9.172
          |          |          |
       Pod 1       Pod 2       Pod 3

The Pods were distributed across multiple OpenShift worker nodes.
7. Self-Healing
A running my-first-app Pod was deliberately deleted.
Kubernetes automatically created a replacement Pod to maintain
the desired replica count.
This demonstrated:
- Desired state
- ReplicaSet reconciliation
- Pod replacement
- Kubernetes self-healing
8. Telco Networking Concepts
The Kubernetes networking concepts were mapped to cloud-native
telecommunications architecture.
A simplified 5G UPF architecture:
                    SMF
                     |
                  N4/PFCP
                     |
                     v
                    UPF
                   /   \
                 N3     N6
                 /       \
               gNB       Data Network

For a high-performance UPF deployment, additional network
interfaces may be required.
Conceptually:
UPF Pod
   |
 Multus
   |
   +---------+---------+
   |         |         |
  eth0      net1      net2
   |         |         |
Management   N3        N6

9. Multus, SR-IOV and DPDK
Multus
Allows a Pod to have multiple network attachments/interfaces.
SR-IOV
Allows an SR-IOV-capable physical NIC to expose Virtual Functions
(VFs) that can be assigned to workloads.
DPDK
Provides optimized mechanisms for high-performance packet processing.
Conceptual architecture:
Physical NIC
     |
     PF
     |
    VFs
     |
   Multus
     |
    UPF
     |
   DPDK

Key Learning
This lab demonstrated the relationship between:
Pod
 ↓
CNI
 ↓
Multus
 ↓
OVN-Kubernetes
 ↓
Pod network interface

and:
Service
 ↓
EndpointSlice
 ↓
Pod IPs

These Kubernetes networking concepts provide the foundation for``
understanding more advanced telco networking technologies such as
Multus, SR-IOV and DPDK.

5. Commit it with:

```text
Document OpenShift networking lab

After committing
Your repository should now look like:
openshift-telco-5g-learning/
│
├── README.md
│
├── manifests/
│   └── yaml-demo.yaml
│
└── networking/
    └── kubernetes-networking.md
