+++
title = 'Giải quyết vấn đề N + 1 queries trong Rails'
date = 2024-02-24T21:59:06+09:00
featuredImage = 'n_plus_1_problem.png'
draft = false
+++
Rails thường được cho là framework rất dễ học và làm quen, bản thân mình cũng cho là như vậy. Nếu chỉ là làm những trang web cơ bản như bán hàng, blog... thì có lẽ chỉ cần học vài tuần, thêm chút kiến thức về bootstrap, jquery là đã đủ dùng. Nhưng, để có thể nắm vững, tối ưu được hệ thống và phục vụ lượng người dùng lớn thì lại là 1 chuyện khác. Để hiểu sâu về Rails ta cũng cần phải đầu tư thời gian nhiều như những ngôn ngữ và framework khác. Chúng ta không thể mong đợi rằng các vấn đề sẽ tự nhiên biến mất chỉ vì chuyển sang dùng 1 công cụ nhanh hơn. 1 thuật toán chạy chậm trong C++ thì hẳn nhiên nó cũng chạy chậm trong Ruby :pensive:. 

Trong Rails khi nói đến cải thiện hiệu năng thì có rất nhiều khía cạnh cần quan tâm, trong đó đặc biệt phải nói đến là việc truy vấn đến database. Và vấn đề mà anh em chúng ta thường mắc phải (không biết hoặc biết mà bỏ qua), đó là các N + 1 queries .
# N + 1 queries là gì?
Để hiểu rõ về vấn đề này thì mình sẽ dùng ví dụ sau để các bạn dễ hình dung:

```php
class Category < ActiveRecord::Base
  has_many :posts
end

class Post < ActiveRecord::Base
  belongs_to :category
end
```

Đây là ví dụ khá điển hình khi làm 1 trang blog, bạn có 2 class là ```Category``` và ```Post``` có mối quan hệ như trên. Giả sử khi muốn hiển thị tên category của 5 bài đăng gần đây nhất ta thường làm như sau:

```php
# Tại controller
@lasted_posts = Post.order(:published).limit(5)

# Tại View
@lasted_posts.each do |post|
	<%= post.category.name %>
end
```

Nhìn qua thì có vẻ như không có vấn đề gì nhưng khi kiểm tra trên cosole ta sẽ thấy các query được sinh ra như sau:

```php
Post Load (0.5ms)  SELECT  "posts".* FROM "posts" ORDER BY "posts"."published" ASC LIMIT $1  [["LIMIT", 5]]
Category Load (0.3ms)  SELECT  "categories".* FROM "categories" WHERE "categories"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
Category Load (0.2ms)  SELECT  "categories".* FROM "categories" WHERE "categories"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
Category Load (0.2ms)  SELECT  "categories".* FROM "categories" WHERE "categories"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
Category Load (0.2ms)  SELECT  "categories".* FROM "categories" WHERE "categories"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
Category Load (0.2ms)  SELECT  "categories".* FROM "categories" WHERE "categories"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
```

Rails mặc định với cơ chế [lazy-load](https://rubyinrails.com/2014/01/08/what-is-lazy-loading-in-rails/), tức là ứng dụng sẽ chỉ truy vấn tới database khi dữ liệu được yêu cầu như là khi sử dụng các lệnh **all, first, count, each**. Vì vậy khi ta gọi lệnh ```each``` ở đoạn code trên, đầu tiên ứng dụng sẽ query tới 5 bài post gần nhất (1 query). Rồi trong mỗi vòng lặp lại query tới từng category tương ứng của các bài post (5 query). Tổng lại ta có 6 (5 + 1) query, đó là lý do tại sao được gọi là vấn đề n + 1 query. 

Với ví dụ này, số lượng query vẫn còn ít nên có thể các bạn chưa thấy vấn đề gì cả, và trước đây mình cũng như vậy, vì chỉ toàn làm những project nhỏ và đơn giản thôi mà. Nhưng khi đi làm ở công ty, tiếp xúc với hệ thống dữ liệu lớn tới hàng triệu bản ghi mình mới nhận ra sự khác biệt rất lớn. Khi đó nếu không xử lý tốt thay vì (5 + 1) thì số lượng query có thể lên tới (1.000.000 + 1) (nếu không dùng paging, còn nếu có paging thì thường sẽ là 100 + 1). Lúc đó trang web load rất là lâu, và khi nhìn vào cosole thì thực sự là kinh khủng :scream:. Đặc biệt trong trường hợp remote database, mỗi lần truy cập đều tính phí và tốc độ cũng không thể nhanh bằng local database được thì đây là vấn đề thực sự đáng phải quan tâm. Để khắc phục tình trạng này, Rails cung cấp giải pháp rất đơn giản mà hữu ích, đó là ```eager load```. Vậy cụ thể ```eager load``` là? :point_down:

# Eager load trong Rails
Eager load cho phép ta load dữ liệu từ bảng có quan hệ ngay lập tức, sẽ không cần phải truy vấn từng đối tượng. Cụ thể ở đây sẽ load trực tiếp dữ liệu của Category vào trong các Post ngay khi query tới 5 bài đăng gần nhất. 

Có 3 phương pháp sử dụng eager load là ```include```, ```preload``` và ```eager_load```. Vậy 3 phương pháp này có gì khác nhau, ta sẽ thử lần lượt vào đoạn code ở trên nhé  :wink:. Ta chỉ cần sửa dòng lệnh đầu tiên, còn câu lệnh ```each``` thì không cần thay đổi.

* **preload**

```php
# Chỉ truy vấn trên bảng post
@lasted_posts = Post.preload(:category).order(:published).limit(5)
	
# Kết quả tại console
Post Load (0.5ms)  SELECT  "posts".* FROM "posts" ORDER BY "posts"."published" ASC LIMIT $1  [["LIMIT", 5]]
Category Load (0.4ms)  SELECT "categories".* FROM "categories" WHERE "categories"."id" IN ($1, $2)  [["id", 2], ["id", 3]]
	
# Truy vấn cả trên bảng categories
@lasted_posts = Post.preload(:category).where('categories.id = 1').order(:published).limit(5)
	 
# Kết quả tại console
ActiveRecord::StatementInvalid (PG::UndefinedTable: ERROR:  missing FROM-clause entry for table "categories")
```

Dùng ```preload``` sẽ chỉ tạo ra 2 query, 1 để lấy các bài post và 1 để load category tương ứng vào từng bài post. Nhưng nếu ta muốn truy vấn với điều kiện trên bảng quan hệ thì sẽ gây ra lỗi.

* **eager_load**

```php
# Chỉ truy vấn trên bảng post
@lasted_posts = Post.eager_load(:category).order(:published).limit(5)
	
# Kết quả tại console
SQL (4.9ms)  SELECT  "posts"."id" AS t0_r0, "posts"."title" AS t0_r1, "posts"."content" AS t0_r2, "posts"."cover" AS t0_r3, "posts"."category_id" AS t0_r4, "posts"."created_at" AS t0_r5, "posts"."updated_at" AS t0_r6, "posts"."user_id" AS t0_r7, "posts"."description" AS t0_r8, "posts"."slug" AS t0_r9, "posts"."published" AS t0_r10, "posts"."clap_count" AS t0_r11, "categories"."id" AS t1_r0, "categories"."name" AS t1_r1, "categories"."created_at" AS t1_r2, "categories"."updated_at" AS t1_r3, "categories"."slug" AS t1_r4, "categories"."image" AS t1_r5 FROM "posts" LEFT OUTER JOIN "categories" ON "categories"."id" = "posts"."category_id" ORDER BY "posts"."published" ASC LIMIT $1  [["LIMIT", 5]]
	
# Truy vấn cả trên bảng categories
@lasted_posts = Post.eager_load(:category).where('categories.id = 1').order(:published).limit(5)
	 
# Kết quả tại console
SQL (0.7ms)  SELECT  "posts"."id" AS t0_r0, "posts"."title" AS t0_r1, "posts"."content" AS t0_r2, "posts"."cover" AS t0_r3, "posts"."category_id" AS t0_r4, "posts"."created_at" AS t0_r5, "posts"."updated_at" AS t0_r6, "posts"."user_id" AS t0_r7, "posts"."description" AS t0_r8, "posts"."slug" AS t0_r9, "posts"."published" AS t0_r10, "posts"."clap_count" AS t0_r11, "categories"."id" AS t1_r0, "categories"."name" AS t1_r1, "categories"."created_at" AS t1_r2, "categories"."updated_at" AS t1_r3, "categories"."slug" AS t1_r4, "categories"."image" AS t1_r5 FROM "posts" LEFT OUTER JOIN "categories" ON "categories"."id" = "posts"."category_id" WHERE (categories.id = 1) ORDER BY "posts"."published" ASC LIMIT $1  [["LIMIT", 5]]
``` 

Dùng ```eager_load``` sẽ chỉ tạo ra 1 query duy nhất sử dụng **LEFT JOIN**, thời gian truy vấn lâu hơn, nhưng bù lại có thể tìm kiếm dựa trên điều kiện của bảng quan hệ.

* **includes**

```php
# Chỉ truy vấn trên bảng post
@lasted_posts = Post.includes(:category).order(:published).limit(5)
	
# Kết quả tại console
Post Load (0.5ms)  SELECT  "posts".* FROM "posts" ORDER BY "posts"."published" ASC LIMIT $1  [["LIMIT", 5]]
Category Load (0.4ms)  SELECT "categories".* FROM "categories" WHERE "categories"."id" IN ($1, $2)  [["id", 2], ["id", 3]]
	
# Truy vấn cả trên bảng categories
@lasted_posts = Post.includes(:category).where('categories.id = 1').order(:published).limit(5)
	 
# Kết quả tại console
ActiveRecord::StatementInvalid (PG::UndefinedTable: ERROR:  missing FROM-clause entry for table "categories")
	 
# Sửa dụng kết hợp .references
@lasted_posts = Post.includes(:category).where('categories.id = 1').order(:published).limit(5).references(:category)
	 
# Kết quả trên console
SQL (0.7ms)  SELECT  "posts"."id" AS t0_r0, "posts"."title" AS t0_r1, "posts"."content" AS t0_r2, "posts"."cover" AS t0_r3, "posts"."category_id" AS t0_r4, "posts"."created_at" AS t0_r5, "posts"."updated_at" AS t0_r6, "posts"."user_id" AS t0_r7, "posts"."description" AS t0_r8, "posts"."slug" AS t0_r9, "posts"."published" AS t0_r10, "posts"."clap_count" AS t0_r11, "categories"."id" AS t1_r0, "categories"."name" AS t1_r1, "categories"."created_at" AS t1_r2, "categories"."updated_at" AS t1_r3, "categories"."slug" AS t1_r4, "categories"."image" AS t1_r5 FROM "posts" LEFT OUTER JOIN "categories" ON "categories"."id" = "posts"."category_id" WHERE (categories.id = 1) ORDER BY "posts"."published" ASC LIMIT $1  [["LIMIT", 5]]
```

Nếu chỉ dùng ```includes``` thì truy vấn trên bảng post cho kết quả giống với ```preload```. Còn nếu kết hợp với ```references``` thì kết quả sẽ giống với ```eager_load``` khi truy vấn với điều kiện trên bảng quan hệ categories. Qua đó ta có thể thấy được ```includes``` là tiện lợi nhất, bù đắp được chỗ thiếu của cả ```eager_load``` và ```preload```.

# Bullet gem
Mặc dù hiểu được vấn đề và cách giải quyết song đôi khi dù biết là không tốt nhưng vì thời gian gấp gáp mà ta bỏ qua, luôn nghĩ rằng sau này có thời gian rảnh sẽ quay lại tối ưu code. Nhưng khi hệ thống lớn dần, ta cũng quên mất vị trí cần sửa, vậy làm thế nào để phát hiện ra N + 1 query 1 cách nhanh chóng. Việc tìm lại trong từng đoạn code, phân tích xem có vấn đề hay không thì mất rất nhiều thời gian và công sức. Chính vì vậy mà gem [bullet](https://github.com/flyerhzm/bullet) ra đời, không chỉ thông báo khi gặp đoạn code có chứa N + 1 queries, bullet còn phát hiện những nơi mà eager load không được sử dụng, giúp bạn tránh được việc load những dữ liệu không cần thiết.

Để cài đặt các bạn thêm dòng sau vào  **Gemfile**

```php
gem 'bullet', group: 'development'
```

Nhưng như vậy là chưa đủ, **bullet** sẽ chỉ thực hiện những công file mà các bạn yêu cầu trong file **config/environments/development.rb**. Dưới đây là ví dụ config bullet mình hay dùng:

```php
config.after_initialize do
  Bullet.enable = true # Bật bullet.
  Bullet.alert = true # Hiển thị alert box trên browser.
  Bullet.bullet_logger = true # Ghi lại log vào file bullet.log
end
```

Còn nhiều config khác rất hữu ích, các bạn có thể xem tại đây [Bullet Configuration](https://github.com/flyerhzm/bullet#configuration).

# Tổng kết
Vậy là qua bài viết này mình đã phân tích về việc N + 1 queries ảnh hưởng như thế nào tới tốc độ của ứng dụng, đồng thời cũng đưa ra giải pháp cải thiện cho vấn đề này. Hi vọng bài viết hữu ích đối với các bạn. Hãy cùng nhau tạo nên các hệ thống tối ưu hơn nào =))).

P/s: Nếu thông tin trong bài có gì sai hoặc khó hiểu, các bạn cứ thoải mái comment ở dưới giúp mình nhá. Thank you :kissing_heart:.

# Tham khảo
* [https://semaphoreci.com/blog/2017/08/09/faster-rails-eliminating-n-plus-one-queries.html](https://semaphoreci.com/blog/2017/08/09/faster-rails-eliminating-n-plus-one-queries.html)
