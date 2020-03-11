# Thực hiện test Cinder QoS 
## Step 1: Tạo cinder-qos
```sh
$ openstack volume qos create --consumer "front-end" --property "read_iops_sec=20000" --property "write_iops_sec=10000" high-iops
hoặc
$ cinder qos-create high-iops consumer="front-end" read_iops_sec=20000 write_iops_sec=10000
```
## Step 2: Gắn QoS vừa tạo cho `volume type`
```sh
$ openstack volume qos associate QOS_ID VOLUME_TYPE_ID
hoặc
$ cinder qos-associate QOS_ID VOLUME_TYPE_ID
```
Trong đó: 
- `QOS_ID:` ID của Qos vừa tạo ở trên hoặc dùng command sau kiểm tra lại: `openstack volume qos list`
- `VOLUME_TYPE_ID:` ID của `volume type` dùng commnad sau để kiểm tra: `openstack volume type list`

## Step 3: Gắn volume vào máy VM
Thực hiện trên `Horizon` hoặc sử dụng command
## Step 4: Kiểm tra QoS đã được áp dụng cho Volume trên VM chưa
**Thực hiện trên Node Compute**
- Liệt kê các VM đang chạy bằng lệnh sau:
```sh
[root@compute1 ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 1     instance-00000001              running
```
- Sử dụng lệnh `virsh dumpxml` để `dump` ra file `.xml` như sau:
```sh
$ virsh dumpxml instance-00000001 > /tmp/instance-00000001.xml
```
- Kiểm tra các thông số trong file vừa dump `instance-00000001.xml`:
```sh
cat /tmp/instance-00000001.xml
......
      <source protocol='rbd' name='volumes/volume-e559a2c6-1eb6-4a82-a745-fcf772bedbee'>
        <host name='192.168.20.51' port='6789'/>
        <host name='192.168.20.52' port='6789'/>
      </source>
      <target dev='vdb' bus='virtio'/>
      <iotune>
        <read_iops_sec>20000</read_iops_sec>
        <write_iops_sec>10000</write_iops_sec>
......
```
=> Thông số: `read_iops_sec` và `write_iops_sec` đã được áp dụng
## Một số command hay dùng:
- Bỏ `qos` khỏi `volume type`
```sh
openstack volume qos disassociate --volume-type <volume-type> | --all <qos-spec>
```
- volume qos delete
```sh
openstack volume qos delete [--force] <qos-spec> [<qos-spec> ...]
```
- volume qos set
```sh
openstack volume qos set [--property <key=value> [...] ] <qos-spec>
```
- volume qos show
```sh
openstack volume qos show <qos-spec>
```
- volume qos unset
```sh
openstack volume qos unset [--property <key> [...] ] <qos-spec>
```

## Tài liệu tham khảo
- https://docs.openstack.org/cinder/latest/admin/blockstorage-capacity-based-qos.html
- https://docs.openstack.org/cinder/latest/admin/blockstorage-basic-volume-qos.html
- https://docs.openstack.org/python-openstackclient/train/cli/command-objects/volume-qos.html
