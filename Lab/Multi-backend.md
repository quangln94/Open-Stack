# Cấu hình Backend cho Openstack

## 1. Cấu hình Multi Backend cho Openstack

### 1.1 Cấu hình 3 backend, chỉnh sửa file `/etc/cinder/cinder.conf` với nội dung sau:
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

### 1.2. Tạo Volume type
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

### 1.3. Tạo Volume
- Tạo Volume với `volume type`
```sh
# Volume này sẽ được tạo trên lvmdriver-1 hoặc lvmdriver-2.
openstack volume create --size 1 --type lvm volume-1

# Volume này sẽ được tạo trên lvmdriver-3
openstack volume create --size 1 --type lvm_gold volume-2
```

## 2. Cấu hình sử dụng driver filter và weighing cho scheduler

### 2.1 Ví dụ 1
- File cấu hình `/etc/cinder/cinder.conf` với nội dung sau:
```sh
[default]
scheduler_default_filters = DriverFilter
enabled_backends = lvm-1, lvm-2

[lvm-1]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = sample_LVM01
filter_function = "volume.size < 10"

[lvm-2]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = sample_LVM02
filter_function = "volume.size >= 10"
```

Volumes với size `< 10 GB` được gửi tới `lvm-1` và volumes với size `>= 10 GB` được gửi tới `lvm-2`.

### 2.2 Ví dụ 2
- File cấu hình `/etc/cinder/cinder.conf` với nội dung sau:
```sh
[default]
scheduler_default_weighers = GoodnessWeigher
enabled_backends = lvm-1, lvm-2

[lvm-1]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = sample_LVM01
goodness_function = "(volume.size < 5) ? 100 : 50"

[lvm-2]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = sample_LVM02
goodness_function = "(volume.size >= 5) ? 100 : 25"
```

## Tài liệu tham khảo
- https://docs.openstack.org/cinder/latest/admin/blockstorage-multi-backend.html
- https://docs.openstack.org/cinder/latest/admin/blockstorage-driver-filter-weighing.html
