# HCS Huawei Cloud Service
Huawei made a service for each type of on-prem DC aspects (computing, network, storage...) to make it integrate seamlessly into **1 shared rsrc pool  (Cloud DC)**.\

==> Hybrid Cloud Solution: **physical distribution + logical unification**
**FusionSphere OpenStack**: mltpl fskl HW/DC share rsrcs (deploy ECSs..) ~ Vmware cloud foundation\
**FusionAccess**: make raw VMs into desktop --> VDI [desktop on the cloud: secure/non-persistnt\
**FusionCompute**: hypervisor ~VMware
**ManageOne**: Management Dashboard/ can manage rsrcs from mltpl clouds

## MIGRATION
- **REHOST**: Acquire cloud resources equivalent to the on-prem ones, copy apps and data on them, then reroute users
- **RELOCATE**: Move the Virtualisation env, not each vm
- ...


## Use Cases
#### Independent Global Zone
    turnkey: divide rsrc pools into stripped down parts for X focus
#### iMaster: smart network mngmnt
Receives API calls of network changes -> applies the necessary hardware changes to obey those calls.
#
# **Services**
![alt text](CloudServices.png)


### ※ COMPUTE ※
#### ECS Elastic Cloud Server
- Shared VM
- ECS **I**mg**M**gmnt**S**rvc
- HCEOS: Huawei's cloud ECS OS
- **AS**: Auto scaling

#### BMS Bare Metal Server
- fskl srvr
- Enterprise Resource Service (ERS)

#
### ※ STORAGE ※
```
Storage Topologies:
  - SAN: Storage Area Network (LAN for storage only works with blocks)
  - NAS: Network Attached storage (srvr in LAN)

Storage  Types:
  - File: strict hierarchical structure of folders and files / fastest / not scalable
  - Object: ≠objects for data and metadata / scalable
  - Block: data into fixed sized chunks / scalable
```

#### EVS Elastic Volume Service
- EVS disks = block storage to attacht to BMS & ECS (vSSD)
#### SFS Scalable File Service (Turbo)
- File storage (NAS) via nfs prtcl
#### OBS Object B Storage
- object storage / called via https api

#
### ※ NETWORK ※
#### VPC
Regional resource|mltpl AZs, private network with IPs, rules, / security
#### EIP Elastic IP
static public IP for a spcfc rsrc in VPC
#### Elastic Load Balance (ELB)
LB traffic to ECSs based on policies

#### VPN
connect user to VPC/ESC
#### Direct connect
connect DC to VPC on dedicated channel

  * ? vs VPN: the only difference is with a DC and one is with a user, then why not use a site-2-site vpn?
#### VPC endpoint
connects VPC to external services

  * ? this service allows an ESC for example to connect to a remote server ? it creates a connection and allocates a public IP or how?
#### Cloud Connect CC
a network between DCs

  * vs vpn and direct connect
#### ENS Enterprise Networking Service
connects clouds and cloud resources with IPs anc controlling them with uniform security policies

  * vs VPC?


#
### ※ Security ※
* **HSS Host Security Service**

  - Server, VM, CT protection\
  - Protect server workloads in hybrid clouds and multi-cloud DCs. It provides host security functions, Container Guard Service (CGS), and Web Tamper Protection (WTP).

* **DBAS Database Audit Service:** Logs reports and alerts every db action
* **DBAS platorm edition:** same but for the cloud db (? not the app db?)
* **KMS Key Management Service:** central key (DataEncK, CustorMastrK) management
* **DEW Data Encryption Workshop:**

  * ?uses Cloud-hosted Hardware Security Module (Cloud HSMs) to build password service resource pools and centrally schedule and manage the password resources. ?
  * Provides **VSM Virtual Security Module** to equip apps with security modules ()enc, certs
  * **CSMS Cloud Secret Management Service** credential manager to avoid hardcoding/plaintext creds, why is it useful when KMS, VSM, exist
  * **CCm Cloud Certificate Manager: include P**rvt**C**ert**A**uthority\*\*, SSl C\*\*ert **M**ngr
* **WAF** protect aginst web attacks (Challenge Collapsar (CC) attacks)
* **CloudFW:** fw + IDS/P + AI
* **CBH Cloud Bastion Host:** Central security n control mngmnt for all
* **SecMaster:** SOC control pltfrm: alert mngmnt + auto response
* **PBH Platform Bastion Host:** CBH for O\&M
* **N**twrk**DR:** protects L4-L7 traffic automatically ? vs CFW
* **PHSS Patform Host Sec Srvc:** mngs security of (v)servers on low lvl (mngmnt plane/IaaS srvcs
* **DSC Data Sec Center:** mngs data security
* **PWAF** ? vs PWAF
* **FBUS Fusion KnowledgeBase Upgrade Service:** ?Fusion KnowledgeBase Upgrade Service (FBUS) is a unified platform that manages feature databases of CFWforHCS, HSS, WAF (including PWAF), and Network Detection and Response (NDR) security services. FBUS uses a unified management solution to provide one-click upgrade and rollback for services depending on feature databases, improving feature data?


#
### ※ Backup & DR ※

- CSBS

- CSDR

- CSHA

- VHA


#
### ※ CTs ※

- **CCE cloud container enginer**

- **SWR Software Repo for Containers** 
#
### ※ App Services ※

- **Simple Message Notification (SMN)**

- **ROMA Connect:** eases integration of data and apps

- **DCS istributed Cache Service**

- **APM Application Performance Management**

- **AOM Application Operations Management** ?vs APM?

- **LTS Long Tank Srvc:** Logs/insights for bettr decision making

- **ServiceStage:** app deployment pltfrm

- **Astro Zero:** low code app builder

- **MAS Multi-site High Availibility Servce:**

- **Distributed Message Service (DMS):** is a message queuing service compatible with open-source Kafka and RocketMQ, providing instances with exclusive compute, storage, and bandwidth resources.

- **FunctionGraph:** code and conditions for when to deploy cloud rsrcs?


#
### ※ CodeArts ※

App lifecycle and env management

#
### Enterprise Intelligence EI

**MRS MapReduce**

**...**
#
### DB services
...
#


### Mngmnt Srvc

**Service Builder:** Backed by open service APIs, O&M automation capabilities, and the government and enterprise process adaptation engine, Service Builder provides a unified process and a robust ecosystem for provisioning IT capabilities as services. You can quickly apply for, provision, configure, and deploy IT resources and capabilities online.


#
### IoT services
#
#### Enterprise application service

**Workspace**

Huawei Cloud Workspace is a workspace service based on cloud computing. Unlike conventional PCs and VDIs, Workspace enables your organization to quickly build office environments without investing a large amount of money and spending days on deployment. Workspace supports multiple login options, allowing you to flexibly access files and use applications for mobile work.?

#### LVS: linux virtual server: 
L1 LB
#### Nginx 
GW
#### HAProxy
route traffic to backend + LB
#### API Gateway
expose services through it
#### SDR
charge records

## System Security



