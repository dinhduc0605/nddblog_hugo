+++
title = 'Ransack - công cụ tuyệt vời giúp tìm kiếm và sắp xếp dữ liệu đơn giản hơn'
date = 2024-02-23T21:20:44+09:00
draft = true
+++
Kể từ bài viết trước cũng đã phải hơn 2 tháng rồi nhỉ, cứ mỗi lần động vào định viết gì đó mà không nghĩ ra gì là mình lại thôi :joy:. Hôm nay phải cố gắng lắm mới ngồi được vào máy tính gõ ra vài dòng. Sau 1 hồi đắn đo suy nghĩ thì mình quyết định viết về những kiến thức cũng như kinh nghiệm đã học được từ khi đi làm đến giờ :sunglasses:.
 
Trước đây mỗi lần cần tìm kiếm hay sắp xếp dữ liệu mình đều tự viết các câu query, tuy rằng rails cũng có sẵn những method như ```where```, ```find```, ```order``` ... nhưng nhiều lúc vẫn khá là vất vả, nhất là khi phải query qua nhiều thuộc tính. Gần đây mới biết thêm 1 gem rất thú vị là Ransack - giúp giải quyết vấn đề tìm kiếm và sắp xếp 1 cách nhanh chóng và hiệu quả :heart_eyes:. Trong bài này mình sẽ giới thiệu sơ qua cách cài đặt cũng như cách sử dụng gem này.
# Cài đặt

 Ransack tương thích với Rails 5.0, 5.1 và 5.2 và phiên bản Ruby từ sau 2.2. Nếu bạn sử dụng Rails < 5.0 thì hãy dùng ruby 1.8. Nếu như với ruby 1.8 hay JRuby mà gặp phải vấn đề, hãy sử dụng các phiên bản trước của Ransack (từ 1.3.0 trở xuống).

Để cài đặt bạn thêm dòng sau vào Gemfile

```ruby
gem 'ransack'
```
Nếu các bạn muốn sử dụng phiên bản mới nhất thì hãy sử dụng nhánh **master**

```ruby
gem 'ransack', github: 'activerecord-hackery/ransack'
```
# Cách sử dụng
**Trong controller**

Method ```ransack``` dùng để tìm kiếm dữ trong database dựa vào các điều kiện được truyền từ ```params[:q]```, sau đó kết quả được lấy ra thông qua method ```result```.

```ruby
def index
  @q = Person.ransack(params[:q])
  @people = @q.result(distinct: true)
end
```

Khi dùng với pagination như ```kaminari``` thì không thể dùng ```distinct: true```, nên hãy sử dụng ```.to_a.uniq```:

```ruby
@people = @q.result.includes(:articles).page(params[:page]).to_a.uniq
```

**Trong view**

2 helper method chính là ```search_form_for``` và ```sort_link```, được định nghĩa trong [Ransack::Helpers::FormHelper](https://github.com/activerecord-hackery/ransack/blob/master/lib/ransack/helpers/form_helper.rb).

**```search_form_for``` thay thế cho ```form_for``` để tạo search form trong view với tham số đầu mặc định là q ( params[:q] )**

```php
<%= search_form_for @q do |f| %>

  # Tìm các phần tử có thuộc tính name chứa...
  <%= f.label :name_cont %>
  <%= f.search_field :name_cont %>

  # Tìm các phần tử nếu trong các articles liên kết tồn tại article có title chứa...
  <%= f.label :articles_title_start %>
  <%= f.search_field :articles_title_start %>

  # Các thuộc tính có thể được nối liền và được tìm kiếm với 1 giá trị duy nhất.
  <%= f.label :name_or_description_or_email_or_articles_title_cont %>
  <%= f.search_field :name_or_description_or_email_or_articles_title_cont %>

  <%= f.submit %>
<% end %>
```

Tham số của ```f.search_field phải``` được viết dưới dạng như sau: ```attribute_name[_or_attribute_name]..._predicate```. Trong đó ```attribute_name``` là tên trường ( thuộc tính ) của model, ```_or_``` là kết hợp thêm dữ liệu của trường ( thuộc tính ) khác, còn ```_predicate``` là ```cont``` ( chứa ), ```start``` ( bắt đầu với )... Như ví dụ ở trên ```name_or_description_or_email_or_articles_title_cont``` tức là tìm các Person có name hoặc description hoặc email hoặc ( tồn tại article trong các articles liên kết có title ) chứa chuỗi được nhập vào form.

Để truyền format ta làm như sau:

```php
<%= search_form_for(@q, format: :pdf) do |f| %>

<%= search_form_for(@q, format: :json) do |f| %>
```

**```sort_link``` giúp header của các table có thế sắp xếp theo thứ tự tùy theo từng loại dữ liệu ( text, date,... )**

```php
<%= sort_link(@q, :name) %>
```

Có thể truyền thêm các tham số khác như tên header, kiểu sắp xếp mặc định:

```php
<%= sort_link(@q, :name, 'Last Name', default_order: :desc) %>
```

Sắp xếp với nhiều trường khác nhau:

```php
<%= sort_link(@q, :last_name, [:last_name, 'first_name asc'], 'Last Name') %>
```

Như ví dụ trên, đầu tiên sẽ xếp theo ```last_name``` rồi đến ```first_name```.

Trong trường hợp bạn muốn sắp xếp theo các giá trị phức tạp, như kết quả của 1 lệnh SQL bạn có thể dùng scope. Trong model bạn định nghĩa scope với các thuộc tính ảo mà bạn muốn dùng để sắp xếp:

```ruby
class Person < ActiveRecord::Base
  scope :sort_by_reverse_name_asc, lambda { order("REVERSE(name) ASC") }
  scope :sort_by_reverse_name_desc, lambda { order("REVERSE(name) DESC") }
...
```

sau đó sử dụng trong view như sau:

```php
<%= sort_link(@q, :reverse_name) %>
```

```sort_link``` khi render trên view sẽ được hiển thị với 1 mũi tên bên cạnh, để tùy chỉnh lại mũi tên này ( màu sắc, kích thước... ) bạn có thể tùy chỉnh trong  ````config/initializers/ransack.rb````.

```ruby
Ransack.configure do |c|
  c.custom_arrows = {
    up_arrow: '<i class="custom-up-arrow-icon"></i>',
    down_arrow: 'U+02193',
    default_arrow: '<i class="default-arrow-icon"></i>'
  }
end
```
Như vậy là ở trên mình đã giới thiệu sơ qua cách sử dụng gem Ransack, đây chỉ là những ví dụ cũng như cách sử dụng cơ bản nhất. Để hiểu thêm về gem này các bạn có thể tham khảo tại [Ransack GitHub](https://github.com/activerecord-hackery/ransack).

Định viết thêm phần cuối kinh nghiệm khi sử dụng Ransack mà tại dùng cũng lâu rồi không nhớ ra cái nào hay để viết, đành tạm bỏ trống  sau này có thì thêm vào vậy :pensive:.

# Kinh nghiệm khi dùng Ransack
* Comming soon.
