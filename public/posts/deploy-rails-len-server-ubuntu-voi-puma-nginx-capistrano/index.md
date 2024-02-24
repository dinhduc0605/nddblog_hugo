# Deploy ứng dụng Rails lên server Ubuntu 18.04 với Puma, Nginx và Capistrano

Xin chào các bạn, có lẽ cũng gần nửa năm rồi mình chưa viết bài mới nhỉ. Hôm nay do thời hạn dùng thử của Amazon free tier đã hết mình mất cả buổi sáng đổi server của blog qua bên Vultr. Do tốn khá nhiều thời gian và công sức nên nhân dịp này mình muốn viết 1 bài để chia sẽ với các bạn cách deploy ứng dụng Rails lên server Ubuntu 18.04 với Puma, Nginx và Capistrano

Trước khi bắt tay vào cài đặt bạn cần có các điều kiện dưới đây:

* Server Ubuntu 18.04. Mình đang sử dụng server của [Vultr](https://www.vultr.com/). Theo mình tìm hiểu thì hiện tại đang là rẻ nhất trong số các nhà cung cấp VPS có uy tín ( 3.5$/tháng ).
* Ứng dụng Rails sẵn sàng để deploy.

# Bước 1: Cài đặt Nginx

 ```php
 deploy@vultr:~$ sudo apt update
 deploy@vultr:~$ sudo apt install nginx
 ```
# Bước 2: Cài đặt database (Postgres)
 
 ```php
 deploy@vultr:~$ sudo apt update
 deploy@vultr:~$ sudo apt install postgresql postgresql-contrib
 ```
 
 Khi deploy Capistrano chỉ chạy lệnh db:migrate, vì vậy các bạn cần tạo sẵn database để deploy được suôn sẻ.
 
 Vào PostgresSQL prompt bằng các lệnh sau:
 
 ```php
 deploy@vultr:~$ sudo -i -u postgres
 deploy@vultr:~$ psql
 ```
 Sau đó ta tạo cơ sở dữ liệu db_name (tên database được định nghĩ trong file database.yml)
 
 ```php
 postgres=$ CREATE DATABASE db_name;
 ```

Để chạy được lệnh db:migrate ta cần có database role, vì vậy ta tạo role như sau (username và password được định nghĩa trong file database.yml) :

```php
postgres=$ CREATE USER username WITH PASSWORD 'password'; 
```
Cấp quyền cho user để có thể thao tác với database

```php
postgres=$ GRANT ALL PRIVILEGES ON DATABASE 'db_name' to username;
```

# Bước 3: Cài đặt RVM

Đầu tiên ta phải cài đặt GPG keys

```php
deploy@vultr:~$ gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```

Sau đó thực hiện các câu lệnh sau để cài đặt RVM

```php
deploy@vultr:~$ sudo apt-get install software-properties-common
deploy@vultr:~$ sudo apt-add-repository -y ppa:rael-gc/rvm
deploy@vultr:~$ sudo apt-get update
deploy@vultr:~$ sudo apt-get install rvm
deploy@vultr:~$ echo 'source "/etc/profile.d/rvm.sh"' >> ~/.bashrc
```

Cài đặt ruby 2.5.3

 ```php
 deploy@vultr:~$ rvm install 2.5.3
 deploy@vultr:~$ rvm use 2.5.3 --default
 ```
 
# Bước 4: Thiết lập SSH keys
 
 Khi deploy ứng dụng lên server ta cần clone code của ứng dụng từ Github (Bitbucket, Gitlab...). Để clone code mà không cần phải nhập mật khẩu (thông qua SSH protocol, không phải HTTP) ta cần tạo SSH key.
 
 ```php
 deploy@vultr:~$ ssh-keygen -t rsa
 ```
 
 Nếu không cần cài đặt gì đặc biết các bạn chỉ việc enter. Sau đó sẽ có 2 file được tạo ra tại thư mục `~/.ssh/`là `id_rsa.pub` và `id_rsa`. Sau đó bạn thêm `id_rsa.pub` vào deployment keys theo hướng dẫn sau [(Cài đặt)](https://docs.github.com/en/developers/overview/managing-deploy-keys#setup-2)
 
Đến đây đã thành công thì sẽ hiển thị như sau

```php
deploy@vultr:~$ ssh -T git@github.com
=> Hi dinhduc0605! You've successfully authenticated, but GitHub does not provide shell access. 
```

Nếu trên máy local của bạn chưa có ssh keys, bạn cũng làm như trên để tạo key mới. Sau đó thêm khoá công khai vào Authorized Keys file trên server.
 
```php
$ cat ~/.ssh/id_rsa.pub | ssh -p your_port_num deploy@your_server_ip 'cat >> ~/.ssh/authorized_keys'
```

# Bước 5: Config Capistrano và Nginx
Đầu tiên ta thêm các gem sau vào group :development trong Gemfile:

```php
group :development do
    gem 'capistrano',         require: false
    gem 'capistrano-rvm',     require: false
    gem 'capistrano-rails',   require: false
    gem 'capistrano-bundler', require: false
    gem 'capistrano3-puma',   require: false
end
```

Sử dụng bundler để cài đặt các gem đó

```php
$ bundle
```

Sau đó thực hiện lệnh sau để config Capistrano:

```php
$ cap install
```

Các file sau tự động được tạo ra:
* `Capfile`: load các task đã được định nghĩa sẵn như: chọn Ruby version, precompile assest, clone git repo...
*  `config/deploy.rb`: cấu hình các task dùng để deploy lên server như truy cập server qua ssh, quản lý các file logs...
* `config`: thư mục chứa file config cho  loại môi trường như staging, production.

Thay thế nội dung file `Capfile` như sau:

```php
# frozen_string_literal: true

# Load DSL and Setup Up Stages
require 'capistrano/setup'
require 'capistrano/deploy'

require 'capistrano/rails'
require 'capistrano/bundler'
require 'capistrano/rvm'

require "capistrano/scm/git"
install_plugin Capistrano::SCM::Git

require 'capistrano/puma'
install_plugin Capistrano::Puma

# Loads custom tasks from `lib/capistrano/tasks' if you have any defined.
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r}
```

Thay thế các thông số sau tương ứng với ứng dụng của mình. Phần còn lại thì giữ nguyên không đổi.

```php
# frozen_string_literal: true

# Change these
server 'your_server_ip', port: your_port_num, roles: [:web, :app, :db], primary: true

set :repo_url,        'git@example.com:username/appname.git'
set :application,     'appname'
set :user,            'deploy'
set :puma_threads,    [4, 16]
set :puma_workers,    0
```

Tiếp theo ta cần cấu hình nginx để lắng nghe các request từ port 80 và chuyển tới Puma socket. Tạo file nginx.conf tại thư mục config (thay thế server_username và appname tương ứng):

```php

upstream puma {
  server unix:///home/deploy/apps/appname/shared/tmp/sockets/appname-puma.sock;
}

server {
  listen 80 default_server deferred;
  # server_name example.com;

  root /home/deploy/apps/appname/current/public;
  access_log /home/deploy/apps/appname/current/log/nginx.access.log;
  error_log /home/deploy/apps/appname/current/log/nginx.error.log info;

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  try_files $uri/index.html $uri @puma;
  location @puma {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://puma;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
}
```

# Bước 6: Deploy ứng dụng
Các bạn tạo commit và push lên nhánh master như sau:

```php
$ git add -A
$ git commit -m "Set up Puma, Nginx & Capistrano"
$ git push origin master
```

Sau đó deploy lần đầu tiên với capistrano:

```php
$ cap production deploy:initial
```

Lệnh này sẽ tự động deploy ứng dụng của bạn lên server, cài đặt các gem cần thiết và khởi động puma server. Quá trình sẽ mất khoảng 5-15 phút.

Nếu mọi thứ suôn sẻ tiếp đến bạn cần symlink file nginx.conf trong ứng dụng Rails tới thư mục `sites-enabled`:

```php
deploy@vultr:~$ sudo rm /etc/nginx/sites-enabled/default
deploy@vultr:~$ sudo ln -nfs "/home/server_username/apps/appname/current/config/nginx.conf" "/etc/nginx/sites-enabled/appname"
```

Khởi động lại Nginx:

```php
deploy@vultr:~$ sudo service nginx restart 
```

Tới đây các bạn đã có thể truy cập website của mình thông qua IP của server rồi.
Với các lần deploy tiếp theo các bạn chỉ cần chạy lệnh sau là đủ (lưu ý nếu bạn thay đổi file config/nginx.conf thì cần phải khởi động lại Nginx):

```php
$ cap production deploy
```

# Tổng kết
Như vậy mình đã hướng dẫn các bạn deploy ứng dụng Rails lên server Ubuntu 18.04 với Puma, Nginx và Capistrano. Hi vọng các bạn thành công!!!
Với các bạn có server dụng lượng bộ nhớ thấp khi deploy có khả năng bị lỗi thì có thể tham khảo bài viết sau:

* Thêm không gian Swap trên Server Ubuntu 18.04 (comming soon)

