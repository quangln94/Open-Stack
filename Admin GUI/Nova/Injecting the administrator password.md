# Injecting administrator password

Compute có thể generate 1 random administrator (root) password và truyền password đến 1 instance. Nếu tính năng này được bật, users có thể ssh tới 1 instance mà không cần `ssh keypair`. Random password xuất hiện ở output của command `openstack server create`. Có thể view và set admin password từ dashboard.

Password injection using the dashboard

Mặc định dashboard sẽ hiển thị admin password và cho phép user đổi password

Nếu bạn không muốn support password injection, disable trường password bằng cách sửa trong file `local_settings.py`.
```sh
OPENSTACK_HYPERVISOR_FEATURES = {
...
    'can_set_password': False,
}
```

## 1. Pre-condition

Cài đặt `libguestfs` trên Node Nova Compute.
```sh
yum install libguestfs python-libguestfs libguestfs-tools-c -y
```
Cấu hình `nova.conf`
```sh
libvirt_inject_password=true  
libvirt_inject_key=true  
libvirt_inject_partition=-1
```
Restart nova-compute
```sh
service openstack-nova-compute restart
```
2. File injection


- http://kimizhang.com/how-to-inject-filemetassh-keyroot-passworduserdataconfig-drive-to-a-vm-during-nova-boot/
