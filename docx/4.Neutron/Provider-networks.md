# Open vSwitch: Provider networks

## 1. Kiến trúc

<img src=blob:https://imgur.com/e515ea12-98a5-4af5-bf58-77078f3d7657>

Kết nối giữa các thành phần

<img src=https://i.imgur.com/upRT34w.png>

## 2. Example configuration

Sử dụng mô hình kết nối các thành phần

### 2.1 Node Controller

**1. Install Networking service cung cấp neutron-server service và ML2 plug-in.**

**2. Sửa file cấu hình `/etc/neutron/neuron.conf`**
```sh
[DEFAULT]
core_plugin = ml2
auth_strategy = keystone
rpc_backend = rabbit
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
...
```
Disable `service plugins` vì provider networks không yêu cầu.
```sh
[DEFAULT]
service_plugins =
```
Enable 2 DHCP agents trên mỗi network để cả 2 Node Compute có thể cung cấp DHCP service provider networks.
```sh
[DEFAULT]
dhcp_agents_per_network = 2
```
**3. Cấu hình trong file `ml2_conf.ini`:**

Cấu hình drivers và network types:
```sh
[ml2]
type_drivers = flat,vlan
tenant_network_types =
mechanism_drivers = openvswitch
extension_drivers = port_security
```
Lưu ý: `tenant_network_types` không chứa giá trị nào bởi vì kiến trúc không hỗ trợ self-service networks.

Cấu hình network mappings:
```sh
[ml2_type_flat]
flat_networks = provider

[ml2_type_vlan]
network_vlan_ranges = provider
```

Cấu hình security group driver:
```sh
[securitygroup]
firewall_driver = iptables_hybrid
```

**4. Populate the database.**
```sh
$ su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```
**5. Start the following services:**
```sh
Server
```

### 2.1 Node Compute

**1. Install Networking service OVS layer-2 agent, DHCP agent, metadata agent.**

**2. Install OVS.**

**3. Sửa gile `neutron.conf` như sau:**
```sh
[DEFAULT]
core_plugin = ml2
auth_strategy = keystone
rpc_backend = rabbit
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
...
```

**4. Sửa file `openvswitch_agent.ini` để cấu hình OVS agent:**
```sh
[ovs]
bridge_mappings = provider:br-provider

[securitygroup]
firewall_driver = iptables_hybrid
```

**5. Sửa file `dhcp_agent.ini` để cấu hình DHCP agent:**
```sh
[DEFAULT]
interface_driver = openvswitch
enable_isolated_metadata = True
```
**6. Sửa file `metadata_agent.ini` để cấu hình metadata agent:**
```sh
[DEFAULT]
nova_metadata_ip = controller
metadata_proxy_shared_secret = METADATA_SECRET
```
Giá trị `METADATA_SECRET` phải trùng với giá trị trong cùng section `[neutron]` trong file `nova.conf`.

**7. Start OVS services:**
```sh
OVS
```
**8. Tạo OVS provider bridge br-provider:**
```sh
$ ovs-vsctl add-br br-provider
```
**9. Add provider network interface như 1 Port trên OVS provider bridge `br-provider`:
```sh
$ ovs-vsctl add-port br-provider PROVIDER_INTERFACE
```
Thay thế `PROVIDER_INTERFACE` với tên của Interface xử lý provider networks, ví dụ `eth0`.

**10. Start services:**
```sh
OVS agent
DHCP agent
Metadata agent
```
## Tài liệu tham khảo
- https://docs.openstack.org/mitaka/networking-guide/deploy-ovs-provider.html
