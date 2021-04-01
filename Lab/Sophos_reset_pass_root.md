# Hướng dẫn thực hiện reset root pass cho máy chủ Sophos

## 1. Thực hiện như sau

- Khởi động lại máy chủ Sophos -> `ESC` vào `GRUB boot loader`

<img src=https://i.imgur.com/gilWSg0.png>

- Màn hình hiện như sau

<img src=https://i.imgur.com/b0vxPiR.png>

- Ấn phím `e` để tiếp tục, màn hình hiện ra như sau:

<img src=https://i.imgur.com/GPq545e.png>

- Tiếp tục ấm phím `e` mà hình hiện ra như sau:

<img src=https://i.imgur.com/9E3KNdb.png>

- Viết vào lệnh sau và ấn `Enter`

```sh
$ init=/bin/bash
```

<img src=https://i.imgur.com/iMJXSwI.png>

Màn hình hiện ra như sau

<img src=https://i.imgur.com/OA9aKHR.png>

- Tiếp tục ấn phím `b`, màn hình hiện ra như sau:

<img src=https://i.imgur.com/8E9q7KF.png>

- Thực hiện thay đổi password cho user root

```sh
passwd root
```

<img src=https://i.imgur.com/jt8JHXK.png>

- Sau khi thay đổi pass thành công, thực hiện lệnh sau để reboot lại máy chủ Sophos

```sh
/etc/init.d/rc6.d/S10reboot
```
***Thực hiện login với password***

## Tài liệu tham khao
- https://support.sophos.com/support/s/article/KB-000034260?language=en_US
