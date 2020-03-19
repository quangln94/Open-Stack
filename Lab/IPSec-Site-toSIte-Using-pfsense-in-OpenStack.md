# Cấu hình VPN Site to site sử dụng Pfsense
## Mô hình Lab
<img src=https://i.imgur.com/162lQfp.png>

## 1. Thực hiện cài đặt Pfsense trên VM. Thực hiện trên cả 2 site
## 2. Cấu hình Pfsense trên Site 1 và Site 2
**Step 1: Đăng nhập vào Web interface theo IP Lan hoặc IP Wan**
http://192.168.1.2 hoặc http://123.4.5.1

**Step 2: Cấu hình IPsec VPN**

Chọn VPN -> IPsec
<img src=https://i.imgur.com/AfltVeK.png>

**Step 3: Thực hiện Add Phase 1, thực hiện trên cả 2 Pfsense**

Chọn Add P1
<img src=https://i.imgur.com/8f2D7Vl.png>
**Step 4: Cấu hình thông tin IPsec**

Điền thông tin ***Remote Gateway*** của Site 2
<img src=https://i.imgur.com/iUfrNgP.png>
Điền thông tin ***Remote Gateway*** của Site 1
<img src=https://i.imgur.com/a2YYKCw.png>

**Step 5: Cấu hình Phase 1**

Sử dụng Pre-Shared Key có sẵn hoặc tự tạo bằng cách ***Generate new Pre-Shared Key*** sẽ tạo ra chuỗi ***Pre-Shared Key***. Chuỗi ***Pre-Shared Key*** này giống nhau trên cả 2 thiết bị Pfsense hoặc thiết bị cấu hình VPN Site to site

Copy ***Generate new Pre-Shared Key*** lên cả 2 thiết bị Pfsense

***Chú ý: Thuật toán mã hóa cũng phải giống nhau trên cả 2 Site***

<img src=https://i.imgur.com/ONUsDkn.png>

Sau khi cấu hình xong chọn ***Save*** ở dưới cùng của Page

**Step 6: Sau khi cấu hình Phase 1, tiếp tục cấu hình Phase 2**

Chon ***Show Phase 2 Entries*** -> ***Add P2***. Thực hiện trên cả 2 Pfsense
<img src=https://i.imgur.com/PsoKxU0.png>
<img src=https://i.imgur.com/ZY8ftcv.png>

Thực hiện trên Site 1
<img src=https://i.imgur.com/xMCZcLF.png>

Trong đó:
- ***Local Network:*** Chọn Private Lan. Nếu hệ thống có nhiều dải Private Lan thì chú ý chọn đúng dải
- ***Remote Network:*** Chọn dải Private Lan cần Remote đến. Ở đây là 172.16.0.0/24

Thực hiện trên Site 2
<img src=https://i.imgur.com/TQ6LxNA.png>

Trong đó:
- ***Local Network:*** Chọn Private Lan. Nếu hệ thống có nhiều dải Private Lan thì chú ý chọn đúng dải
- ***Remote Network:*** Chọn dải Private Lan cần Remote đến. Ở đây là 192.168.1.0/24

Thực hiện giống nhau trên cả 2 Site
<img src=https://i.imgur.com/vXjEIoj.png>

Sau đó chọn ***Save*** -> ***Apply Changes***

**Step 7: Cấu hình Firewall**

Thực hiện trên cả 2 Site

Chọn ***Firewall*** -> ***Rules***
<img src=https://i.imgur.com/RzdlU87.png>
Chọn sang tab ***IPsec*** -> ***Add***
<img src=https://i.imgur.com/lsSqu71.png>

Thực hiện Edit Reles như sau:
<img src=https://i.imgur.com/Ke67gCZ.png>

Trong đó: 
- Source: Ở đây để Any hoặc chọn dải IP Remote đến: 172.16.0.0/24 với Site 1 hoặc 192.168.1.0/24 với Site 2.

Sau đó chọn ***Save*** -> ***Apply Changes***
