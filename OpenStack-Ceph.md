# Cầu hình OpenStack bản Train với Backend là Ceph tích hợp với Ceph Nautilus
## Mô hình

|Server|controller1|compute1|ceph01|ceph02|
|------|-----------|--------|------|------|
|IP|192.168.20.11|192.168.20.31|192.68.20.51|192.168.20.52|

## 1. Thực hiện trên Node Ceph01
Tạo pool trên Ceph

Mặc định Ceph block devices sử dụng pool `rbd`. Ta tạo pool cho `Cinder`  và `Glance`.
```sh
ceph osd pool create volumes 128
ceph osd pool create images 128
ceph osd pool create backups 128
ceph osd pool create vms 128
```
## Cấu hình OpenStack Ceph Clients
Mỗi Node chạy `glance-api`, `cinder-volume`, `nova-compute` và `cinder-backup` cần có Ceph clients. Mỗi Node đó sẽ cần file `ceph.conf`. Ở đây Mô hình Lab gồm 2 Node: Controller: glance-api, cinder-volume, cinder-backup. Compute: nova-compute. Nên copy lên 2 server:
```sh
ssh 192.168.20.11 sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
ssh 192.168.20.31 sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
```
## Cài đặt CEPH CLIENT PACKAGES
- Trên Node chạy `glance-api` cần Python bindings cho `librbd`:
```sh
yum install python-rbd
```
- Trên Node chạy `nova-compute`, `cinder-backup` và `cinder-volume` sử dụng cả Python bindings vf client command line tools:
```sh
yum install ceph-common
```
## SETUP CEPH CLIENT AUTHENTICATION
Nếu enable [cephx authentication](https://docs.ceph.com/docs/master/rados/configuration/auth-config-ref/#enabling-disabling-cephx), tạo 1 user cho `Nova, `Cinder`, `Glance` sử dụng command sau:
```sh
ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images' mgr 'profile rbd pool=images'
ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd-read-only pool=images' mgr 'profile rbd pool=volumes, profile rbd pool=vms'
ceph auth get-or-create client.cinder-backup mon 'profile rbd' osd 'profile rbd pool=backups' mgr 'profile rbd pool=backups'
```
Thêm `keyrings` cho `client.cinder`, `client.glance`, `client.cinder-backup` đến các Node thích hợp và thay đổi quyền:
```sh
ceph auth get-or-create client.glance | ssh 192.168.20.11 sudo tee /etc/ceph/ceph.client.glance.keyring
ssh 192.168.20.11 sudo chown glance:glance /etc/ceph/ceph.client.glance.keyring
ceph auth get-or-create client.cinder | ssh 192.168.20.11 sudo tee /etc/ceph/ceph.client.cinder.keyring
ssh 192.168.20.11 sudo chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup | ssh 192.168.20.11 sudo tee /etc/ceph/ceph.client.cinder-backup.keyring
ssh 192.168.20.11 sudo chown cinder:cinder /etc/ceph/ceph.client.cinder-backup.keyring
```
Trên Node chạy `nova-compute` cần file `keyring` cho `nova-compute`:
```sh
ceph auth get-or-create client.cinder | ssh 192.168.20.31 sudo tee /etc/ceph/ceph.client.cinder.keyring
```
Cần lưu trữ secret key của `client.cinder` user trong `libvirt`. Tiến trình `libvirt` cần nó để truy cập Cluster khi attaching 1 block device từ `Cinder`.

Tạo 1 temporary copy của secret key trên Node chạy `nova-compute`:
```sh
ceph auth get-key client.cinder | ssh 192.168.20.31 tee client.cinder.key
```
Trên Node compute, thêm secret key tới `libvirt` và xóa temporary copy của key:
```sh
$ uuidgen
457eb676-33da-42ec-9a8c-9293d545c337

$ cat > secret.xml <<EOF
<secret ephemeral='no' private='no'>
  <uuid>457eb676-33da-42ec-9a8c-9293d545c337</uuid>
  <usage type='ceph'>
    <name>client.cinder secret</name>
  </usage>
</secret>
EOF
$ sudo virsh secret-define --file secret.xml
Secret 457eb676-33da-42ec-9a8c-9293d545c337 created
$ sudo virsh secret-set-value --secret 457eb676-33da-42ec-9a8c-9293d545c337 --base64 $(cat client.cinder.key) && rm client.cinder.key secret.xml
```
Lưu `uuid` của secret cho cấu hình `nova-compute` sau này.

## 2. Cấu hình OPENSTACK để sử dụng CEPH

**Thực hiện trên Node `controller1`**

- Cấu hình GLANCE

Glance cần sử dụng nhiều backends để lưu images. Để sử dụng Ceph block devices mặc định, cấu hình GLANCE như sau:
```sh
crudini --set /etc/glance/glance-api.conf glance_store stores rbd
crudini --set /etc/glance/glance-api.conf glance_store default_store rbd
crudini --set /etc/glance/glance-api.conf glance_store rbd_store_pool images
crudini --set /etc/glance/glance-api.conf glance_store rbd_store_user glance
crudini --set /etc/glance/glance-api.conf glance_store rbd_store_ceph_conf /etc/ceph/ceph.conf
crudini --set /etc/glance/glance-api.conf glance_store rbd_store_chunk_size 8
```
ENABLE COPY-ON-WRITE CLONING OF IMAGES

Lưu ý: Việc này sẽ exposes backend location thông qua `Glance’s API`, do đó endpoint với tùy chọn này được bật không nên truy cập công khai. Nếu muốn enable copy-on-write cloning of images thực hiện như sau:
```sh
crudini --set /etc/glance/glance-api.conf DEFAULT show_image_direct_url True
```
- Cấu hình CINDER
OpenStack requires 1 driver để tương tác với Ceph block devices. Bạn cần chỉ định pool name cho block device. Trên Node OpenStack sử file `/etc/cinder/cinder.conf` với nội dung sau:
```sh
crudini --set /etc/cinder/cinder.conf DEFAULT enabled_backends ceph
crudini --set /etc/cinder/cinder.conf DEFAULT glance_api_version 2
crudini --set /etc/cinder/cinder.conf ceph volume_driver cinder.volume.drivers.rbd.RBDDriver
crudini --set /etc/cinder/cinder.conf ceph volume_backend_name ceph
crudini --set /etc/cinder/cinder.conf ceph rbd_pool volumes
crudini --set /etc/cinder/cinder.conf ceph rbd_ceph_conf /etc/ceph/ceph.conf
crudini --set /etc/cinder/cinder.conf ceph rbd_flatten_volume_from_snapshot false
crudini --set /etc/cinder/cinder.conf ceph rbd_max_clone_depth 5
crudini --set /etc/cinder/cinder.conf ceph rbd_store_chunk_size 4
crudini --set /etc/cinder/cinder.conf ceph rados_connect_timeout -1
```
Nếu sử dụng `cephx authentication` thì phải cấu hình `user` và `uuid` của secret bnj thêm vào `libvirt` ở trên:

Lưu ý thay đổi `uuid` cho chính xác với Lab của bạn.
```sh
crudini --set /etc/cinder/cinder.conf ceph rbd_user cinder
crudini --set /etc/cinder/cinder.conf ceph rbd_secret_uuid 457eb676-33da-42ec-9a8c-9293d545c337
```
Chú ý rằng nếu cấu hình multiple cinder backends, `glance_api_version = 2` phải có trong [DEFAULT] section.
```sh
crudini --set /etc/cinder/cinder.conf DEFAULT glance_api_version 2
```
- Cấu hình CINDER BACKUP
OpenStack Cinder Backup requires 1 specific daemon nên đừng quên cài đặt nó. Trên Node `Cinder Backup`, thêm vào file `/etc/cinder/cinder.conf` như sau:
```sh
crudini --set /etc/cinder/cinder.conf DEFAULT backup_driver cinder.backup.drivers.ceph.CephBackupDriver
crudini --set /etc/cinder/cinder.conf DEFAULT backup_ceph_conf /etc/ceph/ceph.conf
crudini --set /etc/cinder/cinder.conf DEFAULT backup_ceph_user cinder-backup
crudini --set /etc/cinder/cinder.conf DEFAULT backup_ceph_chunk_size 134217728
crudini --set /etc/cinder/cinder.conf DEFAULT backup_ceph_pool backups
crudini --set /etc/cinder/cinder.conf DEFAULT backup_ceph_stripe_unit 0
crudini --set /etc/cinder/cinder.conf DEFAULT backup_ceph_stripe_count 0
crudini --set /etc/cinder/cinder.conf DEFAULT restore_discard_excess_bytes true
```
- Cấu hình NOVA để ATTACH CEPH RBD BLOCK DEVICE
Để attach Cinder devices (either normal block hoặc issuing a boot from volume), bạn phải sửa Nova (và libvirt) với `user` và `UUID` nào sẽ refer khi attaching device. libvirt sẽ refer tới user này khi connecting và authenticating với Ceph cluster.
```sh
crudini --set /etc/nova/nova.conf libvirt rbd_user cinder
crudini --set /etc/nova/nova.conf libvirt rbd_secret_uuid 457eb676-33da-42ec-9a8c-9293d545c337
```
- Cấu hình NOVA
Để boot tất cả VM trực tiếp từ Ceph, bạn phải cấu hình ephemeral backend cho Nova.

Nó được recommended để enable RBD cache trong file cấu hình Ceph (enabled by default since Giant). Hơn nữa, enabling admin socket mang đến rất nhiểu lọi ích trong việc troubleshooting. Có 1 socket trên VM sử dụng 1 Ceph block device sẽ giúp điều tra performance và các hoạt động không chính xác.

This socket can be accessed like this:
```sh
ceph daemon /var/run/ceph/ceph-client.cinder.19195.32310016.asok help
```
Trên mỗi Node compute sửa file cấu hình Ceph như sau:
```sh
$ vim /etc/ceph/ceph.config
[client]
    rbd cache = true
    rbd cache writethrough until flush = true
    admin socket = /var/run/ceph/guests/$cluster-$type.$id.$pid.$cctid.asok
    log file = /var/log/qemu/qemu-guest-$pid.log
    rbd concurrent management ops = 20
```
Phân quyền cho thư mục:
```sh
mkdir -p /var/run/ceph/guests/ /var/log/qemu/
chown qemu:libvirtd /var/run/ceph/guests /var/log/qemu/
```
Lưu ý rằng user `qemu` và group `libvirtd` có thể thay đổi tùy vào hệ thống của bạn. The provided example works for RedHat based systems.

- RESTART OPENSTACK

Để activate Ceph block device driver và load block device pool name tới configuration, bạn phải restart OpenStack.
```sh
service openstack-glance-api restart
service openstack-nova-compute restart
service openstack-cinder-volume restart
service openstack-cinder-backup restart
```
- BOOTING FROM A BLOCK DEVICE

Bạn có thể tạo 1 volume từ 1 image sử dụng Cinder command line tool:
```sh
cinder create --image-id {id of image} --display-name {name of volume} {size of volume}
```
Bạn có thể sử dụng `qemu-img` để convert từ 1 format tới 1 cái khác, ví dụ:
```sh
qemu-img convert -f {source-format} -O {output-format} {source-filename} {output-filename}
qemu-img convert -f qcow2 -O raw precise-cloudimg.img precise-cloudimg.raw
```
When Glance và Cinder cùng sử dụng Ceph block devices, image là 1 copy-on-write clone, nên nó có thể tạo nhanh 1 volume mới. Trong OpenStack dashboard, bạn có thể boot từ volume đó như sau:
- Launch a new instance.
- Choose the image associated to the copy-on-write clone.
- Select "boot from volume".
- Select the volume you created.

## 4. Kiểm tra hoạt động

### 4.1. Kiểm tra việc tích hợp Glance và Ceph
Trên OpenStack controller, download image cirros
```sh
cd
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```
Upload image lên Glance
```sh
openstack image create "cirros-ceph" --file cirros-0.3.4-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```
Trên `ceph01`, kiểm tra RBD-image của Image vừa tạo
```sh
[root@ceph01 ~]# rbd -p images ls
09566219-9b57-4bf2-8385-261654c6edc1
15b841aa-2f29-4fab-a9da-a0355f0c6dc3
```
### 4.2. Kiểm tra việc tích hợp Cinder và Ceph
Trên OpenStack, tạo 1 volume thuộc backend ceph
```sh
openstack volume create --size 10 volume-test                    
```
Trên `ceph01`, kiểm tra RBD-image của Image vừa tạo
```sh
[root@ceph01 ~]# rbd -p volumes ls
volume-56271869-bd23-4346-98ff-a5d45796d23d
volume-6948486a-2ac2-41f8-835f-f7d4514faf5e
volume-a92542fc-17de-4f9f-b81b-4e8b371e5196
volume-cf8c0435-f8d1-42b4-ab47-66ca41de8015
volume-e559a2c6-1eb6-4a82-a745-fcf772bedbee
volume-f3a547a5-06f0-460c-8df6-7a2315d9d148
```
### 4.3. Kiểm tra việc tích hợp Nova và Ceph
Trên OpenStack, tạo một máy ảo boot từ image `cirros-0.3.4-x86_64-disk.img`

Trên `ceph01`, kiểm tra RBD-image của Image vừa tạo
```sh
[root@ceph01 ~]# rbd -p vms ls
edf26c89-a0b3-45a9-9276-33b47907e23d_disk
```
## Tài liệu tham khảo
- https://docs.ceph.com/docs/master/rbd/rbd-openstack/
- https://gist.github.com/vanduc95/97c4110338e0319a11d4b8ab36c2134a
- https://github.com/hocchudong/Ghichep-Storage/blob/master/ChienND/Ceph/Configure%20Block%20Ceph%20with%20OpenStack.md
