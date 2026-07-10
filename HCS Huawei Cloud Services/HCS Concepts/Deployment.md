# Deployment
.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Application *(6)* \
.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;│ \
Cloud Services (ECS, RDS, CCE, OBS, AI...) *(5)* \
 .&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;│ \
 .&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ManageOne + APIs *(4)* \
 .&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;│ \
.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FusionSphere OpenStack *(3)* \
 .&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;│ \
 .&nbsp;&nbsp;&nbsp;Compute + Storage + Network *(2)*\
 .&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;│ \
 Servers + Switches + Storage Arrays *(1)*


# 1. Huawei Cloud Stack Installation
### **HCC Trunkey**: auto installation tool
##### stndrd Scenario
HCC Turnkey is installed on a dedicated deployment server that is connected to the data center network through a single 10GE port. Using this one physical interface, Turnkey automatically creates multiple sub-interfaces, each assigned to a different VLAN representing a network plane. These include the Internal_Base plane for deployment traffic, the External_OM plane for administrators to manage the cloud after deployment, and it can also reach the BMC network through Layer 3 routing. Although these planes are separate VLANs (different Layer 2 broadcast domains), routers allow them to communicate at Layer 3. Therefore, the deployment server can access all required networks using one physical cable.

##### Secure Scenario

In the high-security scenario, HCC Turnkey uses two physical 10GE ports. The first port works the same as before: it connects to the Internal_Base network and creates a sub-interface for the External_OM VLAN. The second port is connected directly to the BMC network on its own Layer 2 connection. There is no Layer 3 routing between the External_OM and BMC networks, so administrators on the management network cannot directly access the BMC network. This physical separation provides better security because the hardware management network is isolated from the normal management network.

### **vDC Installation**: Logical grouping of resources (compute, network (no ip)) per department/province...

We first install a cloud platform using HCC Turnkey. After that, we create tenants. Inside each tenant, we set up VDCs and sub-VDCs (up to 5 levels), typically mapped to departments, services, or provinces. Each VDC is associated with one or more resource spaces, which are logically isolated portions of the overall resource pool. Each resource space is governed by quotas and quota units, which define and control the maximum resource consumption within that VDC and its sub-levels.


### VPC: an SDN with a private IP range
vDC is a group of resources, inside which a VPC is a software-defined network (SDN) with its own private IP range. Inside the VPC, subnets are subdivisions of that IP range. Private subnets have no direct route to the Internet; they communicate only with other subnets in the VPC through the VPC's internal router. Public subnets have a route to the Internet Gateway. Resources in a public subnet can access the Internet directly if they have a public IP address. Resources in a private subnet can still initiate outbound Internet connections through a NAT Gateway, but only if their route table points to it. Servers are accessible from the Internet only if they are in a public subnet, have a public IP address, and the security rules allow the inbound traffic.


So even if a route exists to the Internet Gateway and the security rules allow inbound traffic, a server that has only a private IP address cannot be reached from the Internet, because the Internet Gateway does not perform NAT. It only routes traffic to and from resources that already have public IP addresses. A NAT Gateway performs only source NAT (SNAT) for outbound connections and does not accept unsolicited inbound connections.

**Actions are controlled through user authentication → user groups → permissions/policies → resource space association → quota enforcement.**



































...
---
# 2. Architectures
1. Traditional **3-tier arch**: built for South-north traffic VS **Spine-Leaf arch**: built for east-west traffic:

    [DC Arch](https://raw.githubusercontent.com/4min4yed/Cloud-Computing/refs/heads/main/Images/DC-Network-Architecture.png)



# 3. Infrastructure
1. **Servers:** \
    i. &nbsp;Management Nodes: cloud controller - FusionSphere OpenStack - ManageOne \
    ii. Compute Nodes: UVP hv1 + NOVA + VMx: VM Lifecycle: Create
↓
Schedule
↓
Image download
↓
Network attach
↓
Storage attach
↓
Boot\
└─>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Rack physical server - Install FusionCompute HV - Install agent that connects it to management plane - Register it into the cluster\
    iii. Network Nodes \
    iv. Storage Nodes: Provide block storage. \
    v.  BMS \
    vi. ELB Nodes \
    vii. GW Nodes: to reach storage

# 3. ManageOne
Minimum 12 nodes / VMs | 
1. ServiceCenter: tenant UI 
2. OperationCenter: monitoring + alarms + gather data (from fusionsphere...)
3. Operations Command Center: brain
4. RESTful APIs:\
  i. Nourthbound: services/OperSS/BsnsSprtSrvc access M1\
  ii. Southband: M1 access other dvcs to integrate them
5. ManageOne DRStage: generate cloud service topologies | detect faults (real or drills)| Failover 
6. Service builder ∼ Terraform IaC: automatic deploymeny of infra