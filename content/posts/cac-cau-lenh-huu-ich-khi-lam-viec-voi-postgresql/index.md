+++
title = 'Các câu lệnh hữu ích khi làm việc với PostgreSQL'
date = 2019-05-02
featuredImage = 'postgresql.png'
+++
Dạo này ở công ty mình hay được giao task lấy dữ liệu trực tiếp từ database theo yêu cầu của bên sale. Và để làm được thì tất nhiên là phải viết raw query rồi. Thực sự thì trước mình cũng chỉ mới nắm được các kiến thức cơ bản của SQL thôi chứ cũng chưa có nhiều cơ hội thực hành nên vẫn gà mờ lắm :sob:. Mới đầu để viết đc query thoả mãn các yêu cầu thì hầu như mình phải google liên tục, động vào cái gì cũng lơ mơ, rất là khó chịu và mất thời gian. Nhưng dần dần mình cũng quen hơn, nhớ được nhiều câu lệnh hơn, và cảm thấy việc viết SQL khá là thú vị :relieved:. Mình nhận ra rằng học và nhớ được lý thuyết là 1 chuyện, còn để áp dụng nó vào thực tế lại 1 chuyện khác không hề đơn giản. Vì vậy trong bài viết này mình sẽ không đề cập đến các kiến thức về SQL mà sẽ tập trung vào các trường hợp cụ thể. 

**Lưu ý là các câu lệnh dưới đây đều được viết bằng PostgreSQL nhé, nên có thể sẽ không đúng đối với MySQL, SQLite...**

Mình sẽ thực hiện truy vấn đối với các bảng như trong hình sau:

![](https://nddblog-prod.s3.amazonaws.com/uploads/image_file/image/11/Screen_Shot_2019-05-02_at_11.32.25.png)

# Các trường hợp
### 1. Tìm các bài viết có chứa 1 trong nhiều từ khoá, ví dụ `A` `B` `C`

Thông thường cách đơn giản nhất là sử dụng `LIKE` nhiều lần với từng giá trị

```php
select * from posts
where content like '%A%' or content like '%B%' or content like '%C%'
```

Nhưng như vậy ta phải lặp lại `content LIKE` nhiều lần, rất là bất tiện. Những lúc như vậy ta có thể sử dụng từ khoá `SIMILAR TO` như sau

```php
select * from posts
where content similar to '%(A|B|C)%';
```

### 2. Trích xuất tên tất cả các user và tên bài viết gần nhất của họ

Có 2 cách như sau

```php
select u.name, p.title from users u
join posts p on p.id = (select max(p2.id)
                        from posts p2
                        where p2.user_id = u.id
                        group by p2.user_id)
```

```php
select u.name, p.title from users u
join posts p on p.user_id = u.id
left join posts p2 on p2.user_id = u.id and p.id < p2.id
where p2.id isnull
```

### 3. Kết hợp các cột với nhau thành 1 cột với `||`
Khi bạn muốn trích xuất dữ liệu gộp từ nhiều cột hay sử dụng toán tử `||`. 

```php
select 'Bài viết ' || p.title || ' từ ngày ' || p.created_at::date
from posts
```
Với câu lệnh trên ta sẽ lấy được những kết quả dạng như sau `Bài viết Các câu lệnh hữu ích khi làm việc với PostgreSQL từ ngày 2019-05-02`

### 4. Trích xuất các bài viết theo thời gian cùng múi giờ

Các database thường được lưu dưới múi giờ là 0, trong khi ở Nhật có múi giờ là +9. Vì vậy khi có các điều kiện liên quan đến thời gian ta cần chú ý cộng thêm 9 giờ để được kết quả chính xác. Trong ví dụ này ta sẽ lấy ra các bài viết được viết từ này 01/03/2019 đến ngày 01/04/2019

```php
select id, title
from posts
where (created_at + interval '9 hours')::date between '2019-03-01' and '2019-04-01'
```

Ta sử dụng `interval` để cộng thêm 9 giờ vào `created_at` rồi dùng `::date` để ép kiểu từ `datetime` về `date` sau đó so sánh như bình thường.

### 5. Kiểm tra 1 hay nhiều giá trị có nằm trong trường định dạng array hay không

Giả sửa trong bảng Post có trường like_user_ids dạng array, dùng để lưu id của những người đã like bài viết chẳng hạn. Để tìm những bài viết mà những user có id = 1, 2, 3... đã like ta sử dụng toán tử `@>` như sau:

```php
select id, title
from posts
where like_user_ids @> Array[1, 2, 3]
```

### 6. Lấy tổng số lượng trong theo từng khoảng thời gian

Trong trường hợp ta muốn lấy số bài được viết theo từng ngày thì nếu như thời gian ở định dạng `date` ta chỉ việc đơn giản là ` group_by(date)` là xong, nhưng nếu là định dạng `timestamp` thì ta không thể group_by được vì khi đó thời gian được tính đến bằng giây. Vì vậy ta sẽ dùng `to_char` để đổi thời gian về dạng tháng rồi `group_by` theo dữ liệu đó.

```php
select to_char(created_at, 'YYYY年MM月') monthly, count(id) count
from posts
where created_at between '2019-01-01' and '2019-05-31'
group by monthly;
```
# Tổng kết
Trên đây là 1 số ứng dụng trong các trường hợp cụ thể, hi vọng sẽ giúp ích cho các bạn trong những trường hợp tương tự. Mình sẽ cập nhật thêm các ví dụ khác dần dần, các bạn nhớ theo dõi bài viết này thường xuyên nhé :kissing_heart:
