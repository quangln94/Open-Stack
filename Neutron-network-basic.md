# Neutron network type
## 1.Provider Networks
Provider networks được thiết kế để map trực tiếp tới networks trong mạng của bạn. Ví dụ về Provider network là 1 VLAN hoặc physical (flat) network trong data center bạn có thể tương tác với OpenStack. 

Một provider network trong Neutron có thể là flat, VLAN, GRE, VXLAN. Ở đây sẽ tập trung vào flat và VLAN. Để tạo 1 provider network bạn cần chỉ định cấu hình physical network mà provider sử dụng.

This configuration is defined as part of the How to Configure OpenStack Neutron in Platform9 Managed OpenStack. The physical network refers to the unique label associated with the provider network config, and the ‘segmentation ID’ refers to the VLAN ID corresponding to this physical network that you’d like to utilize for this provider network. This VLAN ID must fall in the range of VLAN IDs that you supplied as part of the physical network config.

## 2. Tenant Networks
Tenant networks là mạng riêng với người dùng nhất định, và được tạo bởi user hoặc 1 group của users trong tenant. Không có router, networks này cô lập với người khác do đó VM được tạo với networks này không thể route traffic ra ngoài network.

Note: Không giống provider networks, tenant networks không cung cấp option chỉ định VLAN ID vì tenant networks được dùng bởi self-service users cho việc sử dụng như triển khai 1 private network. Khi triển kahi 1 tenant network, 1 VLAN ID sẽ tự chọn cho nó từ pool của VLAN IDs bên dưới physical network.

## Network Interfaces and Ports
Mỗi Neutron network thường sẽ có 1 hoặc nhiều {Network Interface, Port} được gắn với nó. 1 interface và 1 port trên network sẽ ánh xạ duy nhất tới 1 device trong OpenStack. Device có thể là: VM, Router, DHCP server.

## 3. External Networks
External networks thường tương ứng với physical networks trong data center của bạn có thể định tuyến và được tuy cập Internet

VM của bạn có thể  định tuyến các packets từ internal network ra Internet

Bạn có thể gán IPs cho VM của bạn và public địa chỉ từ Internet

## Neutron Router/Gateway
Neutron routers cho phép định tuyến traffic giữa 2 hoặc nhiều Neutron networks. 1 router có khả năng định tuyến traffic giữa Neutron networks của external, provider, tenant. Khi 1 Router ánh xạ 1 internal network tời 1 external network.

# Private/Shared Networks and Multi-Tenancy
Bạn có thể có nhận thấy rằng mỗi network trong Neutron được tạo trong bối cảnh 1 số tenant mặc định là chủ sở hữu của network đó. 1 network có thể được shared sẽ giúp truy cập tới tất cả tenants trong OpenStack.
