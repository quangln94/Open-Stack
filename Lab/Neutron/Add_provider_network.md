# Hướng dẫn tạo thêm Network cho Openstack

## 1. Thực hiện trên Node Controller và Compute
Thêm dải Network cho phép giao tiếp giữa 2 loại máy chủ này.
Ví dụ: Thêm Card mạng ens5 với dải network là 10.10.10.0/24

## 2. Thực hiện trên Node Controller

### 2.1 Tạo brigde và gắn interface vlan cho brigde đó

- Tạo brigde

```sh
ovs-vsctl add-br br-ens5
ovs-vsctl add-port br-ens5 ens5
```

- Cấu hình network-script cho Vlan Interface

```sh
$ cat << EOF > /etc/sysconfig/network-scripts/ifcfg-ens5
DEVICE=ens5
ONBOOT=yes
DEVICETYPE=ovs
TYPE=OVSPort
OVS_BRIDGE=br-ens5
BOOTPROTO=none
EOF
```

- Cấu hình br-ens5

```sh
$ cat << EOF > /etc/sysconfig/network-scripts/ifcfg-br-ens5
DEVICE=br-ens5
ONBOOT=yes
DEVICETYPE=ovs
TYPE=OVSBridge
BOOTPROTO=none
IPADDR=10.10.10.11
NETMASK=255.255.255.0 
OVSDHCPINTERFACES=ens5
EOF
```

- Khởi động lại network

```sh
systemctl restart network
```

- Sửa file cấu hình `/etc/neutron/plugins/ml2/ml2_conf.ini` với nội dung sau:

Giả sử cần thêm dải mạng là ***vlannet***

```sh
[DEFAULT]
[ml2]
type_drivers = flat,vlan,vxlan,gre
tenant_network_types = vxlan,gre,vlan
mechanism_drivers = openvswitch,l2population
extension_drivers = port_security,qos
path_mtu = 1500

[ml2_type_flat]
flat_networks = physnet1

[ml2_type_vlan]
network_vlan_ranges = physnet1,vlannet

[ml2_type_vxlan]
vni_ranges = 200:2000

[ml2_type_gre]
tunnel_id_ranges = 2000:4000

[securitygroup]
enable_ipset = true
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
```

- Sửa file cấu hình `/etc/neutron/plugins/ml2/openvswitch_agent.ini` với nội dung sau:

```sh
[DEFAULT]
[agent]
tunnel_types = vxlan,gre
l2_population = True
extensions = qos
arp_responder = true
[ovs]
bridge_mappings = physnet1:br-provider,vlannet:br-ens5
local_ip = 192.168.20.11
datapath_type = system
[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
```

- Khởi động lại neutron-service

```sh
systemctl restart neutron-server.service neutron-openvswitch-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
```

## 3. Thực hiện trên Node Controller

### 3.1 Tạo brigde và gắn interface vlan cho brigde đó

- Tạo brigde

```sh
ovs-vsctl add-br br-ens5
ovs-vsctl add-port br-ens5 ens5
```

- Cấu hình network-script cho Vlan Interface

```sh
$ cat << EOF > /etc/sysconfig/network-scripts/ifcfg-ens5
DEVICE=ens5
ONBOOT=yes
DEVICETYPE=ovs
TYPE=OVSPort
OVS_BRIDGE=br-ens5
BOOTPROTO=none
EOF
```

- Cấu hình br-ens5

```sh
$ cat << EOF > /etc/sysconfig/network-scripts/ifcfg-br-ens5
DEVICE=br-ens5
ONBOOT=yes
DEVICETYPE=ovs
TYPE=OVSBridge
BOOTPROTO=none
IPADDR=10.10.10.12
NETMASK=255.255.255.0 
OVSDHCPINTERFACES=ens5
EOF
```

- Khởi động lại network

```sh
systemctl restart network
```

- Sửa file cấu hình `/etc/neutron/plugins/ml2/openvswitch_agent.ini` với nội dung sau:

```sh
[DEFAULT]
[agent]
tunnel_types = vxlan,gre
l2_population = True
extensions = qos
arp_responder = true
[ovs]
bridge_mappings = physnet1:br-provider,vlannet:br-ens5
local_ip = 192.168.20.11
datapath_type = system
[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
```

- Khởi động lại neutron-service

```sh
systemctl restart neutron-openvswitch-agent.service
```
