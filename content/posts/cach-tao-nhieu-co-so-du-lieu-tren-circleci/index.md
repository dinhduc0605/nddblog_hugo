+++
title = 'Cách tạo nhiều cơ sở dữ liệu trên CircleCI'
date = 2024-02-24T22:22:42+09:00
draft = false
+++
Thông thường các hệ thống  chỉ có cơ sở dữ liệu  (CSDL) của chính nó, nhưng với các hệ thống phức tạp hơn, liên kết với các hệ thống khác thì nhiều khi sẽ phải truy cập trực tiếp vào các CSDL bên ngoài để lấy thông tin. Trên CircleCI, để tạo CSDL của chính hệ thống ta chỉ cần gọi `bundle exec rake db:create` và `bundle exec rake db:migrate` (ở đây mình giả sử các bạn sử dụng Rails)  là tất cả các bảng sẽ tự động được tạo ra.  Nhưng còn CSDL của hệ thống bên ngoài thì làm như thế nào ? Hôm nay mình sẽ giới thiệu với các bạn cách xử lý trong tình huống này.

# Bước 1: Cài đặt postgres trên CircleCI
Cài đặt postgres ta cần chạy các lệnh sau

```php
sudo curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
sudo apt-get update && sudo apt-get install -y postgresql-client
```

Chú ý trước mỗi câu lệnh đều cần thêm sudo vì để thực hiện các lệnh này trên CircleCI ta cần quyển root 	

# Bước 2: Tạo CSDL thứ 2
Khi đã có postgres ta có thể sử dụng câu lệnh SQL để tạo CSDL như sau

```php
psql -h localhost -U #{user_name} -c "CREATE DATABASE #{database_name};" 
``` 

`user_name` là giá trị của  `POSTGRES_USER` ta khai báo tại image postgres (giả sử rằng bạn config image postgres trong file config.yml của circleci). Để hiểu hơn về cách viết file config.yml các bạn có thể đọc bài [Tích hợp CircleCI trong phát triển ứng dụng Ruby on Rails](https://nddblog.com/posts/tich-hop-circleci-trong-phat-trien-ung-dung-ruby-on-rails) mình đã viết khá lâu trước đây.

# Bước 3: Tạo các bảng cần thiết trong CSDL
Trong test case các bạn cần đến bảng nào thì tạo thêm bảng đó. Ví dụ mình cần bảng User thì sẽ viết lệnh SQL như sau.  Câu lệnh khá là dài vì vậy mình cho vào file users.sql rồi sử dụng tag -f của psql.

 ```php
 // users.sql
 
 CREATE TABLE public.users (
	 id            serial not null primary key,
	 name          string,
	 email         string,
	 phone         string
 )
 ```
 
 sử dụng psql để tạo bảng users
 
 ```php
 psql -h localhost -U #{user_name} -d #{database_name} -f users.sql
 ```
 
 # Bước 4: Tổng hợp các bước trên
 Từ các bước trên ta sẽ thêm lệnh run vào config.yml
 
```php
- run:
name: Setup Secondary Database
command: |
	sudo curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
	sudo apt-get update && sudo apt-get install -y postgresql-client
	psql -h localhost -U #{user_name} -c "CREATE DATABASE #{database_name};"
	psql -h localhost -U #{user_name} -d #{database_name} -f db/sql/users.sql
```

# Kết quả
Tạo thành công CSDL và các bảng trên CircleCI 

 ![](https://nddblog-prod.s3.amazonaws.com/uploads/image_file/image/26/Screen_Shot_2020-04-23_at_0.08.55.png)

Như vậy mình đã hướng dẫn các bạn tạo thêm CSDL thứ 2 ngoài CSDL của hệ thống chính trên CircleCI. Chúc các bạn thành công!
