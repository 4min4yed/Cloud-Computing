# Concepts


## Security is a shared responsibility
##### **Biggest Security risks**

- **Virtualization/Hypervisor compromise**
- **API**
- **Residual Data Leakage**
- **Isolation escapes**
- **IAM**

**Security as a Service (SECaaS)** \
**Unified Policy: Applied on all related resources** \
**North-South**: Traffic entering DC \
**East-West**: Traffic between srvcs in DC \
s

## CHS scalability

<!-- ├── Region 1  (40K VMs max) \
│    ├── AZ 1 \
     |   ├── Resource Pools \
     │   │       ├── Compute Pool \
     │   │       ├── Storage Pool \
     │   │       └── Network Pool \
     │   │ \
     │   └── Host Groups \
..   .. \
│    └── AZ 32 (240 max ttl) \
├── Region 120 \
..
-->
**Global Zone:** ManageOne + IAM &emsp;&emsp;(Maximum host groups = 256 fzkl servers) | KVM Nodes (fzkl hypervisosr) Max = 2,000 \
| \
├──&nbsp;&nbsp;&nbsp;&nbsp;**Region 1** / *Primary Region* ≈ 1 DC &nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;L<50ms &nbsp;&nbsp;|&nbsp; (40K VMs max) | ⚠️ L_DC > 2ms => move to region 2 ⚠️ [heavy control traffic] <br>
│&nbsp;&nbsp;&nbsp;&nbsp;├── **AZ 1**  &nbsp;&nbsp;Group of ressources built on seperated infra so failure in AZ 1 won't affect AZ 2 ⚠️VMs can only attach disks inside the same AZ.⚠️<br> 
│&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;├── **Resource Pools** <br>
│&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;├── Compute Pool <br>
│&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;├── Storage Pool <br>
│&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;└── Network Pool <br>
│&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;│ <br>
│&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;└── **Host Groups**: &emsp;&emsp;**Similar Physical Servers (AI/GPU accel..) &emsp;{&emsp; Host Groups &emsp;{&emsp; Resource Pool &emsp;{&emsp; Virtual Machines**
<br>
│&nbsp;&nbsp;&nbsp;&nbsp;⁝ <br>
│&nbsp;&nbsp;&nbsp;&nbsp;└── **AZ 32** (240 max ttl) <br>
└── **Region 120**
- **Create an other region when:**
   - regulations require seperation
    - latency > 50
    - different SDN arch required

## Virtualization
Communication:\
OpenStack \
     ↓ \
libvirt (API) \
     ↓ \
KVM (Kernel Based Virtualisation) Linux Hypervisor\

**VM Lifecycle:** managed by OpenStack:\
.&nbsp;Create
↓
Boot
↓
Running
↓
Stop
↓
Restart
↓
Snapshot
↓
Resize
↓
Migrate
↓
Delete
## Hardware
**UMA** multiple unified CPUs share RAM through bus => Traffic Jam! \
**NUMA** Non-Uniform Memory Access: each CPU has a ram portion =? fix Jam X delay when data is in a remote RAM \
**NUMA-Aware dplmnt** fix a fct to use 1 local socket + CPU pinning (spcfc cores for nfv )

Hardware remote Access: IPMI to turn on/off BMS | OS-independant | communicates with BMC (small computer in BMS)

PXE (Preboot Execution Environment) installation: boot and install OS over LAN

## Networking
Open vSwitch | Linux Bridge\
Floating IP: moving IP between VMs\
VXLAN: Overlay ntwrk - L3 - 16 million networks\
LACP (Link Aggregation Control Protocol): N fskl links -> 1 logical faster + redundant


##### **Routing**: 
- prvt ntwrk routing: OSPF, EIGRP, RIP: fastest/shortest path based.
- internet routing: BGP Border Gateway Protocol: Policy based.
- VFR Virtual Routing and Forwarding: multiple routing tables for diffrnt tenants in 1 router.
- MPLS (Multiprotocol Label Switching): routing traffic based on tags | more security, isolation and QoS
- EVPN Ethernet VPN
## OpenStack


| OpenStack Component | Huawei Cloud Stack Equivalent | Core Role |
| :--- | :--- | :--- |
| **Nova** | ECS (Elastic Cloud Server) | Creates and manages VMs. |
| **Neutron** | VPC (Virtual Private Cloud) | Handles subnets, routers, and floating IPs. |
| **Cinder** | EVS (Elastic Volume Service) | Provides block storage. |
| **Glance** | IMS (Image Management Service) | Acts as the VM image repository. |
| **Keystone** | IAM / ManageOne | Manages identity and authentication. |
| **Horizon** | Huawei Web Consoles | Serves as the user dashboard interface. |
| **Swift** | OBS (Object Storage Service) | Stores unstructured object data. |
| **Heat** | RFS / Orchestration | Automates infrastructure deployment. |


All OpenStack services communicate using message queues (RabbitMQ)\
Hyper Converged Infrastructure: 1 hardware - multiple VMs (compute, storage...) : edge - less hw

## IAM
**Domain** = Account
**Project** = Region \
**delegation**: delegate account parts to ressources / other accounts.\
**AccessKeyID** & **SecretAccessKey** required for APIs

to-learn
-----
KVM libvirt NUMA CPU pinning Huge Pages SR-IOV KVM virtualization and VM lifecycle VXLAN Linux Bridge Floating IP SSD/HDD tiers IOPS PXE Boot IPMI BMC Hardware provisioning Spine-leaf TOR-switches LACP GRE Overlay networking BGP  MPLS  VRF  EVPN \
Core Switch
       │
Aggregation
       │
Top of Rack
       │
Servers \
GRE Overlay networking BGP  MPLS  VRF  EVPN \
Openstack services X HCS' \
Cloud lifecycle