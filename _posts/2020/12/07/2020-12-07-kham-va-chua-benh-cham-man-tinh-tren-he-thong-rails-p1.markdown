---
layout: post
title:  "Khám và chữa bệnh...chậm mãn tính trên hệ thống Rails (P1)"
date:   2020-12-07 07:00:00 +0700
categories: rails, optimize, tối ưu hoá
---

Từ ngày xửa ngày xưa, ông trời sinh ra Ruby nhưng quên bổ sung nước tăng lực cho ngôn ngữ này. Cũng vì vậy, đứa con Ruby on Rails hoạt động với tốc độ khá chill.

Tuy nhiên, người dùng hệ thống của cháu thì không được "chill" cho lắm. Tưởng tượng các bác có 45 phút để làm bài thi mà mất 2 phút quay quay có tức không. Nhân dịp người dùng phàn nàn quá nhiều về hiệu năng, cháu quyết định khám bệnh cho hệ thống này

#### Tình trạng bệnh lý

**Hồ sơ bệnh án:**
- Sử dụng combo Passenger + Nginx
- Ngày thường vắng khách, cuối tuần vỡ trận
- Mặc dù đã scale cấu hình hết cỡ nhưng vẫn chậm, chỉ thấy hết tiền  nhanh hơn

![](Screen Shot 2020-11-29 at 22.22.11.png)

**Ngoài ra, trang quan trọng nhất - trang làm bài tập online có cây DOM siêu to khổng lồ**

![](Screen Shot 2020-12-03 at 18.04.40.png)

**Tốc độ load trang này luôn dao động 30-50s, gây trầm cảm cho người dùng**

![](IMG_3039.JPG)

**hay thỉnh thoảng toang luôn 😢**

#### Hướng điều trị

##### Bệnh 1: Tràn queue

Thiếu gì thì tăng cái đấy đúng không cả nhà. Tính thử một chút thì trang nặng nhất của cháu là khoảng 100 request, một buổi thi là 300 người, vị chi request queue khoảng `100x300=30000` là xả láng rồi ha.

```
server {
  ...
  add_header X-Frame-Options ALLOWALL;
  passenger_enabled on;
  passenger_app_env production;
  passenger_max_request_queue_size 30000;
  ...
}
```

Cháu sử dụng thuộc tính `passenger_max_request_queue_size` cho config nginx

##### Bệnh 2: Scale hệ thống nhưng không thấy nhanh hơn

Bệnh này khi theo dõi một buổi thi cháu mới thấy, mặc dù tốc độ load đề chậm nhưng CPU Usage và RAM không tăng theo. Chắc chắn có nút thắt cổ chai ở đây rồi.

Giờ cháu sẽ tăng số lượng process và thread của cả Web App lẫn Database

***database.yml***
``` yml
production:
  adapter: postgresql
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 30 } %>
  timeout: 5000
  database: ******
```

Ở đây cháu đã tăng pool của DB lên 30 connections.


***nginx conf*** 
``` yml
passenger_max_pool_size <SỐ TỰ NHIÊN>;
server {
  ...
  passenger_max_request_queue_size 30000;
  passenger_min_instances <SỐ TỰ NHIÊN>;
}
```

`passenger_max_pool_size` là số lượng process tối đa mà Passenger sẽ tạo ra.
`passenger_min_instances` là số lượng tối thiểu process sẽ sinh ra.

Việc tăng giá trị 2 cấu hình có mục đích tăng số tiến trình cùng chạy để tận dụng tối đa sức mạnh của CPU. Vì nếu số tiến trình quá thấp, khi gặp các tác vụ blocking I/O hệ thống sẽ phải chờ các tác vụ đó xử lí xong, dẫn đến giảm hiệu quả sử dụng

Công thức passenger_max_pool_size là:
```
passenger_max_pool_size = (RAM * 0.75) / Dung lượng một app
passenger_min_instances = passenger_max_pool_size
```

Để đo thử một app các bác hãy dùng lệnh `passenger-status` và đo xem khi chạy app của các bác sẽ chiếm bao nhiêu RAM. Như của cháu trung bình là `110MB`, bình thường sử dụng cấu hình `t3a.small (2GB RAM)`

`VD: passenger_max_pool_size = (2048*0.75)/110 ~ 14`

***Ở phần tiếp theo, cháu sẽ khám bệnh chuyên sâu ở tầng Rails (Queries & Cache & Non blocking-UI)***
