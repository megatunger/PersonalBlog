---
layout: post
title:  "Khám và chữa bệnh...chậm mãn tính trên hệ thống Rails (P2)"
date:   2020-12-07 07:00:00 +0700
categories: rails, optimize, tối ưu hoá
---
Tiếp nối buổi khám cho một bệnh nhân Rails bị chậm mãn tính, ở bài trước cháu đã tối ưu chủ yếu liên quan cấu hình nginx, passenger. Ở bài này cháu sẽ tiếp tục khám các bệnh ở tầng Rails app.

##### Bệnh 3: N+1 queries

*Trộng chán không các bác*

![](Screen Shot 2020-12-04 at 17.46.28.PNG)

Sửa cái này thật ra không khó, mẹo của cháu là các bác điều tra xem chỗ nào các bác dùng quan hệ `has_many`, vòng lặp `.each` thì ngay lập tức `.includes` bảng các bác định join.

Ngoài ra hãy chú ý các câu lệnh `.where` mà **nằm trong vòng lặp**.

![](Screen Shot 2020-12-04 at 17.51.09.PNG)

Hãy thử refactor lại logic xem có đẩy được ra ngoài không. Tối ưu cái này là các bác đã giảm thời gian render khủng khiếp rồi

##### Bệnh 4: Cache đâu?

Nội dung câu hỏi và câu trả lời đều là những đoạn HTML không đổi và phục vụ liên tục cho người dùng. Vì vậy cháu quyết định cache các đoạn dữ liệu này

Sử dụng cache ở rails khá đơn giản. Các bác chỉ việc wrap nó vào trong đoạn `cache` như này

```ruby
<% cache(answer) do %>
    <%= answer.content.html_safe %>
<% end %>
```

Như vậy nếu cùng object answer, rails sẽ ưu tiên đọc từ cache. Có lưu ý là các bác nhớ config `cache_store` trong `development.rb` và `production.rb`

Chơi lớn thì sử dụng `memcached` để dạt hiệu quả

***Before 2 bệnh trên***
![](Screen Shot 2020-12-04 at 17.54.48.PNG)

***After @megatunger khám bệnh (+500% mana)***
![](Screen Shot 2020-12-04 at 17.54.11.PNG)

##### Bệnh 6: Thời gian load lần đầu tiên quá lâu

Cháu nhận thấy khi phân tích network có rất nhiều thành phần bị phụ thuộc dẫn đến các khoảng thời gian chết. Đặc biệt ngay từ đầu đã phải load một cái DOM gần 600KB đã gây khó chịu cho người dùng rồi.

Giải pháp ở đây là sử dụng `gem 'render_async'`

Gem này sẽ biến các đoạn `render :partial` của các bác thành một loạt đoạn Javascript load theo kiểu Ajax như thế này 

![](Screen Shot 2020-12-07 at 17.01.38.png)
![](Screen Shot 2020-12-07 at 17.01.51.png)

Nhờ vậy request network khi đó sẽ hiệu quả hơn (tận dụng tối đa resources hệ thống)

![](Screen Shot 2020-12-07 at 17.08.55.png)

Cách implement thì hơi loằng ngoằng tí, và nó cũng phải custom lại theo logic code nên bác nào muốn tìm hiểu thì lên docs của gem đọc nhá.

### Hiệu quả điều trị

*Response time từ ~5000ms xuống còn*

![](Screen Shot 2020-12-07 at 17.08.51.png)

***961ms!***

Cháu khá ấn tượng với kết quả đạt được và đã rút ra nhiều mẹo hay hay :D. Hẹn gặp các bác ở những bài sau!

