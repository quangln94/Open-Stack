# Cloudbase

Tương tự như **cloud-init** cho hệ điều hành Linux thì **cloudbase** sử dụng cho hệ điều hành Windows.

## Password cho user for Administrator

Để enable user Administrator, password cần phải được thay đổi bằng cách truy cập thông qua console.

## Cloudbase-init admin user

Thông thường sẽ khuyến nghị việc disable user Administrator và sử dụng **cloudbase-init** để tạo 1 user admin.

**Cloudbase-init** tạo user với tên là **Admin**. User **Admin** được thêm vào group **Administrators**

User **Admin** đượcc gắn 1 password ngẫu hiện trong quá trình cài đặt, có thể được truy xuất bằng cách sau:

## Thiết lập password cho Administrator

Cloudbase có thể chạy powershell scripts trong quá trình tạo instance. Nó cũng có thể được sử dụng để đặt password cho user Administrator. Script có thể được viết trực tiếp vào trường "Customization Sctipt" bến dưới "Configuration" hoặc được tải lên dưới dạng file.

Vì một số lý do, dường như không thể đặt mật khẩu cho user "admin" bằng cách sử dụng phương pháp này, mặc dù đặt mật khẩu cho "Administrator" thì lại đặt được.

Về cơ bản, bất cứ điều gì mà user trong group Administrators có thể thực hiện thông qua powershell đều có thể chạy qua phương thức này. Bất kỳ thứ gì nhập vào trường "Customization Script"

## Tài liệu tham khảo
- https://docs.safespring.com/compute/windows/
