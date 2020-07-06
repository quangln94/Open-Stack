# Cấu hình Multi Backend cho Openstack

## 1. Cấu hình 3 backend, chỉnh sửa file /etc/cinder/cinder.conf` với nội dung sau:
```sh
enabled_backends=lvmdriver-1,lvmdriver-2,lvmdriver-3
[lvmdriver-1]
volume_group=cinder-volumes-1
volume_driver=cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name=LVM
[lvmdriver-2]
volume_group=cinder-volumes-2
volume_driver=cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name=LVM
[lvmdriver-3]
volume_group=cinder-volumes-3
volume_driver=cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name=LVM_b
```

Trong cấu hình trên, `lvmdriver-1` và `lvmdriver-2` có cùng `volume_backend_name`. Nếu 1 volume tạo requests đến `LVM`, scheduler sử dụng capacity filter scheduler để chọn driver phù hợp nhất là `lvmdriver-1` hoặc `lvmdriver-2`. capacity filter scheduler mặc định được enabled.

## 2. Tạo Volume type
- Tạo `volume type`
```sh
openstack volume type create lvm
openstack volume type create lvm_gold
```
- Link `volume type` đến backend:
```sh
openstack volume type set lvm  --property volume_backend_name=LVM
openstack volume type set lvm_gold  --property volume_backend_name=LVM_b
```

## 3. Tạo Volume
- Tạo Volume với `volume type`
```sh
# Volume này sẽ được tạo trên `lvmdriver-1` hoặc `lvmdriver-2`.
openstack volume create --size 1 --type lvm volume-1
openstack volume create --size 1 --type lvm_gold volume-2
```
## Tài liệu tham khảo
- https://docs.openstack.org/cinder/latest/admin/blockstorage-multi-backend.html
