# Self-service network


## 2.Example configuration

### 2.1 Node Controller

**1. Sửa file `neutron.conf` như sau:

Enable routing và cho phép overlapping IP address ranges.**
```sh
[DEFAULT]
service_plugins = router
allow_overlapping_ips = True
```
**2. Sửa file `ml2_conf.ini` với nội dung sau:

Add vxlan vào `type drivers` và `project network types`.
```sh
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
```
Enable `layer-2 population` mechanism driver.
```sh
[ml2]
mechanism_drivers = openvswitch,l2population
```
Cấu hình VXLAN Network ID (VNI) range.
```sh
[ml2_type_vxlan]
vni_ranges = VNI_START:VNI_END
```
Thay `VNI_START` và `VNI_END` với giá trị thích hợp.

Restart services:
```sh
Server
```

### 2.1 Node Network

**1. Install Networking service OVS layer-2 agent và layer-3 agent.**

**2. Install OVS.**

**3. Sửa file `neutron.conf` với các option sau:**
```sh
[DEFAULT]
core_plugin = ml2
auth_strategy = keystone
...
```

**4. Start services:**
```sh
OVS
```

**5. Tạo OVS provider bridge br-provider:**
```sh
$ ovs-vsctl add-br br-provider
```
**6. Trong file `openvswitch_agent.ini` cấu hình layer-2 agent.
```sh
[ovs]
bridge_mappings = provider:br-provider
local_ip = OVERLAY_INTERFACE_IP_ADDRESS

[agent]
tunnel_types = vxlan
l2_population = True

[securitygroup]
firewall_driver = iptables_hybrid
```
Thay thế `OVERLAY_INTERFACE_IP_ADDRESS` với IP của interface xử lý VXLAN overlays cho self-service networks.

**7. Trong file `l3_agent.ini` cấu hình layer-3 agent như sau:
```sj
[DEFAULT]
interface_driver = openvswitch
external_network_bridge =
```
Lưu ý: `external_network_bridge` không có giá trị.

**8. Start services:**
```sh
Open vSwitch agent
Layer-3 agent
```

### 2.3 Node Compute

**1. Trong file `openvswitch_agent.ini`, enable VXLAN support bao gồm layer-2 population.
```sh
[ovs]
local_ip = OVERLAY_INTERFACE_IP_ADDRESS

[agent]
tunnel_types = vxlan
l2_population = True
```
Thay thế `OVERLAY_INTERFACE_IP_ADDRESS` với IP của interface xử lý VXLAN overlays cho self-service networks.

**2. Restart services:**
```sh
Open vSwitch agent
```

## Tài liệu tham khảo
- https://docs.openstack.org/install-guide/launch-instance-networks-selfservice.html
