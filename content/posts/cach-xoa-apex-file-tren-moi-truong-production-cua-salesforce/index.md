+++
title = 'Cách xoá Apex file trên môi trường production của Salesforce'
date = 2024-03-16T01:15:53+09:00
tags = ["Salesforce", "Apex Class"]
+++
Bạn đã từng phải đối mặt với việc cần xoá một file Apex Class trên môi trường Production của Salesforce mà không biết cách thực hiện? Trong bài viết này, mình sẽ chia sẻ về cách thức xoá một Apex Class một cách an toàn và hiệu quả bằng cách sử dụng công cụ Workbench 💪

## Workbench là gì?
Workbench là một công cụ mạnh mẽ giúp bạn thực hiện các thao tác quản lý dữ liệu và metadata trên các môi trường Salesforce một cách dễ dàng thông qua Force.com APIs. Trong trường hợp này, chúng ta sẽ sử dụng Workbench để xoá một file Apex Class trên môi trường Production.

{{< admonition type=warning title="Chú ý" open=true >}}
Workbench không phải sản phẩm được phát triển bới Salesforce
{{< /admonition >}}

## Các bước để xoá Apex file trên môi trường production
### Bước 1: Chuẩn bị file destructiveChanges.xml và package.xml
Tạo 1 thư mục tên `package` rồi mở trình soạn văn bản của bạn, tạo một file mới với tên `destructiveChanges.xml` trong thư mục đó

Sử dụng cú pháp XML để chỉ định các thành phần cần xoá.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>Your_Apex_Class_Name</members>
        <name>ApexClass</name>
    </types>
    <version>58.0</version>
</Package>
```

{{< admonition type=warning title="Chú ý" open=true >}}
Đảm bảo thay thế Your_Apex_Class_Name bằng tên của file Apex Class bạn muốn xoá.
{{< /admonition >}}

Tiếp đến ta tạo file mới với tên `package.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
  <version>58.0</version> <!-- Use the appropriate API version -->
</Package>
```

### Bước 2: Đóng gói tất cả thành file Zip
Đóng gói thư mục `package` thành file `package.zip`. Các hệ điều hành hiện nay đều hỗ trợ việc đóng gói này rất đơn giản, nên mình sẽ không nói chi tiết ở đây.

Nhưng chú ý với những bạn dùng Macbook, nếu làm theo cách thông thường, chọn folder -> Compress thì ở step sau, khi deploy file zip bằng Workbench sẽ có lỗi như sau

> No Package.xml Found

Vì vậy các bạn cần sử dụng command line để thực hiện việc đóng gói này

```bash
cd package
zip -r -X package.zip *
```

### Bước 3: Truy cập vào Workbench
Truy cập trang chính thức của [Workbench](https://workbench.developerforce.com/). Chọn môi trường Production từ menu dropdown nếu chưa được chọn mặc định rồi đăng nhập vào tài khoản Salesforce của bạn.

### Bước 4: Deploy file package.zip
Trong menu trên cùng, chọn "Migration" và sau đó chọn "Deploy".
Chọn tập tin ZIP mà bạn đã tạo ở Bước 2. Chọn các option như trong hình bên dưới.

{{< image src="deploy_zip_file.png" caption="Deploy package.zip" >}}

Chọn "Next" và bấm nút "Deploy".
Nếu quá trình deploy không gặp vấn đề thì chúc mừng bạn, file apex class đã được xoá thành công 🙌

## Kết Luận
Mặc dù ở đây mình hướng dẫn là xoá file apex class, nhưng các bạn cũng có thể sử dụng để xoá các metadata khác, ví dụ như trigger.
Sử dụng tập tin ZIP chứa package.xml và destructiveChanges.xml là một cách an toàn và hiệu quả để xoá các thành phần Metadata trên môi trường Production của Salesforce.
Nhưng hãy nhớ kiểm tra kỹ lưỡng trước khi thực hiện bất kỳ thay đổi nào trên môi trường Production nhé.
Chúc bạn thành công!!!
