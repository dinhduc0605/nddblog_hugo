# Cùng nhau tìm hiểu về Continuous Integration

Chắc hẳn nhiều bạn cũng đã từng nghe đến cụm từ Continuous Integration ( gọi tắt là CI ), vậy các bạn có từng thắc mắc CI nghĩa là gì, và tại sao lại cần đến nó không? Nếu có hãy cùng mình tìm hiểu thông qua bài viết này nhé :yum:.

# 1. Continuous Integration là?
Continuous Integration là cách thức phát triển phần mềm DevOps, trong đó các lập trình viên thường xuyên merge ( hay commit ) code của họ vào các central repo ( như github, gitlab... ), sau đó những đoạn code sẽ được build và test 1 cách tự động. CI thường là nói đến giai đoạn build và tích hợp đoạn code mới vào quá trình release của phần mềm. 

# 2. Tại sao lại cần Continuous Integration?
Thông thường, các lập trình viên thường làm việc độc lập trong 1 thời gian dài và chỉ merge code vào nhánh master khi họ đã hoàn thành công việc. Chưa kể đến sau khi merge còn cần phải chờ review, có khi phải đợi đến vài ngày hay cả tuần. Điều này dẫn đến việc merge code gặp khó khăn, tiêu tốn nhiều thời gian, và đồng thời cũng tạo ra nhiều bugs, conflicts... khi mà code không được cập nhật trong thời gian dài. 

CI giúp tìm ra bug ngay từ sớm trong quá trình phát triển, giúp việc sửa chữa tốn ít công sức hơn, từ đó cải thiện chất lượng phần mềm, giảm thời gian review code, đồng thời rút ngắn thời gian release các chức năng mới của sản phẩm. Việc test tự động được chạy với mỗi lần commit hay pull, đảm bảo nhánh master luôn trong trạng thái ổn định, không bug.

# 3. Cách thức hoạt động của Continuous Integration
Với continuos integration, lập trình viên thường sử dụng git để commit hay merge code của họ vào các central repo như GitHub, GitLab...

![](https://cdn-images-1.medium.com/max/800/0*Ibsu7Nvvd9gyhHxO.png)

Trên hình là 1 workflow phổ biến của CI. Thường thì mình sẽ thêm 1 bước là chạy test tại local trước, kiểm tra thật kỹ code của mình, khi xác định không có lỗi thì mới push lên. Làm như vậy sẽ giảm số lần test trên server, vì CI được dùng chung cho cả team, tài nguyên có hạn, nếu quá nhiều build cùng lúc thì sẽ có người phải chờ, và tất nhiên là chẳng ai muốn phải chờ rồi đúng không :joy:. Hơn nữa việc đó cũng cho thấy bản thân là người cẩn thận, sẽ được đồng nghiệp đánh giá cao hơn :stuck_out_tongue_winking_eye:. Sau khi push code lên, thông thường sẽ là gửi pull request vào nhánh master. Lúc này trên server sẽ tự động build và chạy test. Nếu như có lỗi xảy ra thì sẽ được thông báo cho các thành viên trong project, người chịu trách nhiệm cho pull đó ngay lập tức sửa code rồi push lên lần nữa, cứ lặp lại như vậy cho tới khi tất cả các test đều thành công. Tất nhiên đây chỉ là thông qua các test case đã được viết trước, còn về logic của đoạn code thì vẫn cần có người review, nếu ok thì lúc đó mới được merge vào nhánh master.

# 4. Một số tool Continuos Integration phổ biến
* **Jenkins**: Là công cụ mã nguồn mở được viết bằng java do Kohsuke Kawaguchi ( nhân viên của Sun Microsystem thời điểm đó ) viết ra. Jenkins nổi bật trong các công cụ CI ngoài yếu tố miễn phí thì còn nhờ tính linh hoạt, nhiều plugin có thể được tích hợp 1 cách dễ dàng. 

* **TeamCity**: Là công cụ CI server mạnh mẽ, được phát triển bởi công ty JetBrains, cái tên có lẽ không xa lạ gì với dân dev chúng ta. Công ty này có rất nhiều sản phẩm nổi tiếng như WebStorm, RubyMine, ReSharper.... TeamCity cung cấp đầy đủ tính năng cho phiên bản miễn phí nhưng giới hạn cho 100 lần build cấu hình ( tuỳ chỉnh config của CI ) và 3 build agents ( cho phép chạy song song 3 build cùng lúc ). Công cụ này chạy trên nhiều nền tảng và hỗ trợ nhiều công cụ cũng như framework khác nhau. Đồng thời cũng có lượng lớn plugins được phát triển bới JetBrains cũng như các bên thứ 3.

* **Travis CI**: Là 1 trong những giải pháp máy chủ hosted CI lâu đời nhất và được mọi người tin dùng. Mặc dù thường được biết đến là giải pháp máy chủ lưu trữ ( hosted ), nhưng nó cũng cung cấp phiên bản dành cho các doanh nghiệp. Travis CI miễn phí cho tất cả các phần mềm mã nguồn mở được lưu trữ trên GitHub. Nhưng ngược lại nếu không phải thì chỉ được 100 build đầu tiên. Có 1 vài mức giá khác nhau có thể lựa chọn, khác biệt chủ yếu là số lượng build có thể chạy cùng lúc.

* **Circle CI**: công cụ được phát triển bới công ty cùng tên, cung cấp dịch vụ cloud CI. Hiện nay Circle CI mới chỉ hộ trợ GitHub và 1 vài ngôn ngữ bao gồm Java, Ruby/Rails, Python, Node.js, PHP, Haskell, và Scala. Nhưng khác biệt chính của nó so với các công cụ khác là cách cung cấp dịch vụ. Giá tiền sẽ được tính theo các container, mỗi container có giá giống nhau và cố định. Container đầu tiên là miễn phí, không giới hạn số lượng project. Có 5 cấp độ build song song (1x, 4x, 8x, 12x và 16x). Hoặc cũng có thể chạy 4 builds trên 16 containers với tốc độ 4x. 

* ...

# 5. Link tham khảo
* [https://aws.amazon.com/devops/continuous-integration/](https://aws.amazon.com/devops/continuous-integration/)
* [https://docs.microsoft.com/en-us/azure/devops/learn/what-is-continuous-integration](https://docs.microsoft.com/en-us/azure/devops/learn/what-is-continuous-integration)
* [https://code-maze.com/top-8-continuous-integration-tools/](https://code-maze.com/top-8-continuous-integration-tools/)

# 6. Tổng kết
Vậy là qua bài viết này mình đã giới thiệu sơ qua về Continuous Integration,  cách thức phát triển phần mềm vô cùng hiệu quá. Đồng thời cũng đã liệt kê 1 số công cụ phổ biến hiện nay như Jenkins, Travis CI, Circle CI... Trong bài viết sau mình sẽ hướng dẫn các bạn cách config để sử dụng Circle CI trong Rails project. Mong mọi người đón đọc nhé  :grin:.

