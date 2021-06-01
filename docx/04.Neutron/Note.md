# Note 
```sh
dhcp_agents_per_network = 3
```
Khi VM được tạo trên 1 Network, Neutron sẽ tạo DHCP agents cho Network này. Số lượng DHCP server cho Network dựa vào file cấu hình. Nếu nhiều hơn 1 DHCP agent được yêu cầu cho mỗi Network sau đó Neutron sẽ tạo DHCP agents trên toàn bộ Network Node khả dụng trong Openstack

## Wor
