# Hướng dẫn thay đổi IP máy chủ Sophos

## 1. Đăng nhập vào máy chủ Sophos
## 2. Thực hiện các thao tác sau
```sh
$ cc
$ RAW
$ lock_override
$ OBJS
$ itfparams
$ primary
```

<img src=https://i.imgur.com/YKXoJvS.png>

- Thực hiện tiếp như sau:
```sh
# Ấn nut <TAB> 2 lần

<TAB><TAB>
```

<img src=https://i.imgur.com/sX9sHyB.png>

- Thực hiện các lênh sau để thay đổi IP

```sh
$ address='192.168.1.2'
$ default_gateway_address='192.168.1.1'
```

Thay đổi các tham số khác nếu cần thiết

<img src=https://i.imgur.com/ellNRzX.png>

- Thực hiện lưu cấu hình

```sh
$ w
$ quit
```

***IP WebAdmin đã được thay đổi, có thể truy cập vào WebAdmin thông qua IP mới.***

## Tài liệu tham khảo
- https://community.sophos.com/utm-firewall/f/hardware-installation-up2date-licensing/87012/webadmin-access-through-eth1-which-needs-to-be-added-via-command-line
- https://rattkin.info/archives/1749
