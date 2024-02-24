# Tích hợp CircleCI trong phát triển ứng dụng Ruby on Rails

Trong bài viết trước chúng ta đã [Cùng nhau tìm hiểu về Continuous Integration](/posts/cung_nhau_tim_hieu_ve_continuous_integration/). Vì vậy như đã hứa, hôm nay mình sẽ hướng dẫn các bạn cài đặt và sử dụng 1 trong các công cụ CI phổ biến hiện nay - CircleCI khi phát triển ứng dụng với Ruby on Rails.

# 1. Đăng ký tài khoản
Bước đầu tiên các bạn cần đăng ký tài khoản của CircleCI. Hiện tại công cụ này chỉ hỗ trợ đăng ký bằng tài khoản Github hoặc Bitbucket:

* Truy cập trang đăng ký [https://circleci.com/signup/](https://circleci.com/signup/)
* Chọn đăng ký bằng Github hoặc Bitbucket.
* Điền thông tin username, password và xác nhận bảo mật 2 lớp (nếu có) rồi chọn đăng nhập.
* Sau khi chọn Authorize Application thì Dashboard của CircleCI sẽ hiện ra.

Đến đây các bạn đã tạo tài khoản thành công, nhưng để bắt đầu thì còn phải config trong Rails để CircleCI có thể nhận diện và thực thi công việc theo yêu cầu.
# 2. Config CircleCI trong Rails
Trong project tạo thư mục tên là ```.circleci```, và tạo file ```config.yml``` ( như vậy filepath sẽ là ```.circleci/config.yml``` ). Về nội dung của file config.yml thì tuỳ từng project, từng công việc ta muốn CircleCI thực hiện mà sẽ khác nhau, vì vậy trong bài này mình sẽ lấy file config.yml của ```nddblog``` làm ví dụ :grin:.

```php
# Ruby CircleCI 2.0 configuration file  
#  
# Check https://circleci.com/docs/2.0/language-ruby/ for more details  
#  
version: 2  
jobs:  
  build:  
   docker:  
   # specify the version you desire here  
   - image: circleci/ruby:2.5.3-node-browsers  
       environment:  
         RAILS_ENV: test   
   
    # Specify service dependencies here if necessary  
    # CircleCI maintains a library of pre-built images 
    # documented at https://circleci.com/docs/2.0/circleci-images/  
    - image: circleci/postgres:10.6  
       environment:  
         POSTGRES_USER: dinhduc  
   
    working_directory: ~/repo  
  
    steps:  
    - checkout  
  
    # Download and cache dependencies  
    - restore_cache:  
        keys:  
        - v1-dependencies-{{ checksum "Gemfile.lock" }}  
        # fallback to using the latest cache if no exact match is found  
        - v1-dependencies-  
  
    - run:  
        name: install dependencies  
        command: |  
          bundle install --jobs=4 --retry=3 --path vendor/bundle  
    - save_cache:  
        paths:  
        - ./vendor/bundle  
        key: v1-dependencies-{{ checksum "Gemfile.lock" }}  
  
    # Database setup  
    - run:  
        name: Set up DB  
        command: |  
          bundle exec rake db:create 
          bundle exec rake db:migrate  
  
    # Run rubocop  
    - run:  
        name: Run rubocop  
        command: bundle exec rubocop
  
    # Run tests!  
    - run:  
        name: run tests  
        command: |  
		  mkdir /tmp/test-results 
		  TEST_FILES="$(circleci tests glob 'spec/**/*_spec.rb' | circleci tests split --split-by=timings)"  
          bundle exec rspec --format progress \ --format RspecJunitFormatter \ --out /tmp/test-results/rspec.xml \ --format progress \ $TEST_FILES  
    # collect reports  
    - store_test_results:  
        path: /tmp/test-results  
    
    - store_artifacts:  
        path: /tmp/test-results  
        destination: test-results
```

Mình sẽ giải thích từng phần để các bạn dễ hình dung nhé :wink:.
 
* ```version```: version của CircleCI, hiện nay đã ra tới 2.1 nhưng trong project của mình vẫn là 2 thôi, chưa có thời gian update mà :disappointed_relieved:
* ```jobs```: các job cần phải thực hiện mỗi khi build trên CircleCI.
* ```build```: tên job.
* ```docker```: là executor, nói đơn giản là nơi để thực hiện các job. Có 3 loại executor khác nhau: docker, machine và macos.
* ```image```: tên image sẽ được sử dụng trong docker, và những image này được lấy từ [dockerhub](https://hub.docker.com/). Trong ```nddblog``` mình sử dụng 2 image là ruby - dùng để chạy project và postgres - dùng để lưu trữ dữ liệu.
* ```environment```: các biến môi trường của từng image. ```RAILS_ENV``` cài đặt môi trường rails trên CircleCI là test. ```POSTGRES_USER``` cài đặt username cho database. Lưu ý username ở đây và trong file ```config/database.yml``` phải giống nhau.
* ```working_directory```: thư mục lưu trữ source code.
* ```steps```: các bước thực hiện trong job.
* ```checkout```: lấy code của nhánh đang build từ github hoặc bitbucket về.
* ```restore_cache```: restore các dependencies đã được cache từ lần build trước nếu có.
* ```run```: thực hiện các command line. Trong file mình có 4 lệnh run lần lượt là cài đặt các dependencies, tạo vào migrate database, chạy test rubocop và chạy rspec test. Nếu các bạn cần chạy lệnh nào khác có thể thêm thoải mái.
* ```save_cache```: lưu cache lại các dependencies.
* ```store_test_results```: lưu các kết quả test ra file để hiển thị trên CircleCI.
* ```store_artifacts```: lưu các kết quả của artifacts ra file để hiển thị trên CircleCI (artifacts là các file như file ảnh, logs, coverage reports....).

Trên đây mình đã giải thích những key quan trọng cần có của 1 file config.yml. Dựa vào đó chắc hẳn các bạn đã có thể tạo ra file riêng cho project của mình rồi nhỉ :fist:. Sau khi đã thêm file config.yml vào project thì hãy commit và push file đó lên repo của mình. 

# 3. Chọn project cần build trên CircleCI
Tại Dashboard của CircleCI, các bạn chọn tab ```Add Projects``` , trong list hiện ra tiếp tục chọn Set Up Project tại project mà các bạn vừa mới push file config.yml lên. Sau đó sẽ có các bước hướng dẫn và file mẫu của config.yml nhưng nếu các bạn đã làm theo bước 2 như mình nói ở trên thì không cần làm thêm gì nữa mà chọn nút Start Building rồi ngồi chờ thành quả thôi :yum:. Chúc các bạn thành công và có được kết quả như hình dưới đây nhé :point_down::point_down::point_down:

![CircleCI Success](https://nddblog-prod.s3.amazonaws.com/uploads/image_file/image/5/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88_2018-12-30_23.11.41.png)
# 4. Tổng kết

Qua bài hôm này mình đã hướng dẫn các bạn cách tích hợp CircleCI trong việc phát triển ứng dụng với Ruby on Rails. Phần config có lẽ sẽ hơi khó hiểu nên để hiểu thêm về phần này các bạn có thể đọc thêm tại [Configuring CircleCI](https://circleci.com/docs/2.0/configuration-reference/#section=configuration). Mặc dù có chút khó khăn nhưng khi đã hoàn thành các bạn sẽ nhận ra sự tuyệt vời của CI. Tin mình đi ahihi :wink:

