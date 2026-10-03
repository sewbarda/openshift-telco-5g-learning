# OpenShift Telco 5G Learning Project

## Overview

Hands-on learning project focused on Red Hat OpenShift, Kubernetes,
containerisation, cloud-native networking and 5G Core CNF architecture.

The project documents practical exercises completed in a Red Hat
OpenShift Developer Sandbox and connects Kubernetes/OpenShift concepts
with telecommunications NFV and 5G Core technologies.

---

## Background

I have 20+ years of telecommunications engineering experience covering
mobile core networks, signalling, IMS, NFV and virtualised infrastructure.

This project focuses on extending that experience into cloud-native
technologies used for modern telco environments.

Areas of previous experience include:

- 2G / 3G / 4G / 5G
- Mobile Core Networks
- SS7 and Diameter
- IMS and VoLTE
- NFV / VNF
- VMware
- OpenStack
- Linux
- Network engineering

---

# Technologies

- Red Hat OpenShift
- Kubernetes
- Containers
- Linux
- YAML
- OpenShift CLI (`oc`)
- CNI
- OVN-Kubernetes
- Multus
- SR-IOV
- DPDK
- 5G Core / CNF concepts

---

# 1. OpenShift Fundamentals

## Deployment

Created a Kubernetes/OpenShift Deployment running an
unprivileged NGINX container.

```bash
oc create deployment my-first-app \
  --image=nginxinc/nginx-unprivileged:latest

#Verified the deployment and Pods using:
oc get deployments
oc get pods

#Scaled the application to three replicas:
oc scale deployment/my-first-app --replicas=3

#Verified the Pods:
oc get pods -o wide

#Deleted a running Pod:
oc delete pod <pod-name>

#Kubernetes automatically created a replacement Pod to maintain
#the desired replica count.
#This demonstrated the Kubernetes desired-state and reconciliation model.
#Concept:
Desired State
     |
     v
Deployment
     |
     v
ReplicaSet
     |
     v
Pods
     |
     X
Pod fails
     |
     v
Replacement Pod

#Created a Deployment using a YAML manifest.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yaml-demo
spec:
  replicas: 4
  selector:
    matchLabels:
      app: yaml-demo
  template:
    metadata:
      labels:
        app: yaml-demo
    spec:
      containers:
      - name: nginx
        image: nginxinc/nginx-unprivileged:latest
        ports:
        - containerPort: 8080

#Applied using:
oc apply -f yaml-demo.yaml


Observed the difference between:
- Node IP
- Pod IP
- Network interface
- Gateway
- OpenShift network
Example Pod:
Node IP:    10.0.15.56
Pod IP:     10.128.9.179
Interface:  eth0
Network:    OVN-Kubernetes

The Pod was assigned:
eth0
10.128.9.179/23

nvestigated the Container Network Interface (CNI) architecture.
The OpenShift environment uses OVN-Kubernetes for the default
Pod network.
Observed the following networking event:
AddedInterface
Add eth0 [10.128.9.179/23] from ovn-kubernetes
This provided practical exposure to the relationship between
Multus and the default OVN-Kubernetes network.
6. Kubernetes Services
Created and inspected a ClusterIP Service:
Service:
my-first-app

ClusterIP:
172.30.160.238

Port:
8080

The Service used the selector:
app=my-first-app

This demonstrated how a stable Service endpoint can provide
access to dynamically changing Pods.
7. EndpointSlices
Inspected the EndpointSlice associated with the Service:
oc get endpointslice \
  -l kubernetes.io/service-name=my-first-app -o wide

Observed the following Pod endpoints:
10.131.0.19
10.131.10.130
10.128.9.172

Conceptual architecture:
Service
172.30.160.238:8080
        |
        v
EndpointSlice
        |
   +----+----+
   |    |    |
  Pod  Pod  Pod

8. Multus Networking
Studied Multus as a CNI meta-plugin that allows Pods to have
multiple network attachments/interfaces.
Conceptual telco architecture:
UPF Pod
   |
 Multus
   |
   +---------+---------+
   |         |         |
  eth0      net1      net2
   |         |         |
Management   N3        N6

9. SR-IOV
Studied Single Root I/O Virtualization (SR-IOV) and the
relationship between Physical Functions (PFs) and Virtual Functions (VFs).
Concept:
Physical NIC
     |
     PF
     |
 +---+---+---+
 |   |   |   |
VF1 VF2 VF3 VF4

Key concepts:
- PF — Physical Function
- VF — Virtual Function
- SR-IOV — mechanism for creating Virtual Functions from an
  SR-IOV-capable NIC
- High-performance networking
10. DPDK
Studied Data Plane Development Kit (DPDK) and its role in
high-performance packet processing.
Conceptual architecture:
NIC
 |
SR-IOV VF
 |
DPDK
 |
User-plane application

DPDK is particularly relevant to high-performance telco
user-plane workloads such as the 5G UPF.
11. 5G Core Networking
Mapped Kubernetes/OpenShift networking concepts to a
simplified 5G Core architecture.
                 SMF
                  |
              N4 / PFCP
                  |
                  v
                 UPF
                /   \
              N3     N6
              /       \
            gNB       Data Network

N3
gNB ↔ UPF user-plane traffic.
N4
SMF ↔ UPF control/session management using PFCP.
N6
UPF ↔ external Data Network.
12. VNF to CNF Transition
Compared traditional NFV/VNF architecture with cloud-native
CNF architecture.
Traditional NFV
Physical Infrastructure
        |
 VMware / OpenStack
        |
       VM
        |
     Guest OS
        |
       VNF

Cloud-Native
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

Learning Roadmap


