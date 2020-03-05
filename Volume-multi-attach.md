# Volume multi attach
## 1. Tạo Shared Volume
Tạo volume type:
```sh
$ cinder type-create multiattach
$ cinder type-key multiattach set multiattach="<is> True"
```
Lưu ý: Tạo volume type mặc định chỉ được thực hiện bởi admin, bạn có thể thay đổi trong file cấu hình `policy.json` nếu cần. Thực hiện như sau:
```sh
echo "'volume:multiattach': 'rule:admin_api'" > /etc/cinder/policy.json
echo "'volume:multiattach': 'rule:admin_or_owner'" > /etc/cinder/policy.json
```
Đê tạo volume bạn cần sử dụng volume type vừa tạo trước đó như sau:
```sh
$ cinder create <volume_size> --name <volume_name> --volume-type <volume_type_uuid>
$ cinder create 10 --name shared-volume --volume-type b47458d2-7914-4757-8945-ba7d7b32cd26
```
## 2. Kiểm tra
Thực hiện tạo 2 VM sau đó attach 1 volume `shared-volume` này vào 2 VM

## Tài liệu tham khảo:
- https://docs.openstack.org/cinder/latest/admin/blockstorage-volume-multiattach.html
- https://superuser.openstack.org/articles/meet-volume-multi-attach-great-new-feature-openstack-queens/
