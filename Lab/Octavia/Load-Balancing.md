# Basic Load Balancing

## 1. Deploy a basic HTTP load balancer
Mô hình Lab
|Server|IP|subnet|
|------|--|------|
|Back-end 1|192.0.2.10|private-subnet|
|Back-end 2|192.0.2.11|private-subnet|
|||public-subnet|

-Tạo load balancer `lb1` on subnet `public-subnet`
```sh
openstack loadbalancer create --name lb1 --vip-subnet-id public-subnet
openstack loadbalancer show lb1
```
-Tạo listener `listener1`**
```sh
openstack loadbalancer listener create --name listener1 --protocol HTTP --protocol-port 80 lb1
```
- Tạo pool `pool1` làm default pool cho `listener1`
```sh
openstack loadbalancer pool create --name pool1 --lb-algorithm ROUND_ROBIN --listener listener1 --protocol HTTP
```
- Add members 192.0.2.10 and 192.0.2.11 trên `private-subnet` vào `pool1`
```sh
openstack loadbalancer member create --subnet-id private-subnet --address 192.0.2.10 --protocol-port 80 pool1
openstack loadbalancer member create --subnet-id private-subnet --address 192.0.2.11 --protocol-port 80 pool1
```
## 2. Deploy a basic HTTP load balancer with a health monitor

## Tài liệu tham khảo
- https://docs.openstack.org/octavia/train/user/guides/basic-cookbook.html
