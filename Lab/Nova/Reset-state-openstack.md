## 1. Để có thể delete Instance đang ở trạng thái `deleting` có thể thực hiện như sau:
Reset state sẽ đưa state của instance thành `error`. Sau đó có thể thực hiện delete delete instance:
```sh
$ nova reset-state VM-ID
$ nova delete VM-ID
```
## 2. VM bị treo staus khi Migrate từ Node này sang Node khác thực hiện như sau:
Có thể đưa Instance về trạng thái `active` thay vì `error` như sau:
```sh
$ nova reset-state --active VM-ID
```
Thực hiện reboot server sau đó thực hiện Migrate lại
