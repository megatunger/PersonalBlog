---
layout: post
title: "Kết hợp Active Storage và Cloudflare CDN chỉ từ 0 đồng"
date: 2021-07-06 07:00:00 +0700
categories: active stoarge, cloudflare, cdn, rails, ruby on rails
---

![](banner-1544x500.png)

#### Đề bài

Nếu bác nào chưa biết về Active Storage, đây là một Gem được tích hợp vào phần core của Ruby on Rails, có nhiệm vụ lo toàn bộ các vấn đề liên quan đến lưu trữ file. Active Storage tạo một địa chỉ cố định dạng `xxx.xx/rails/active_storage/*`, khi truy cập link này tuỳ vào phương thức lưu trữ mà Rails sẽ redirect tiếp đến địa chỉ của file.

![](active_storage.webp)

Các bác có thể dễ thấy nút thắt ở đây chính là mọi request đều phải đi qua rails active storage trước rồi mới được xử lí tiếp. Cháu thường sử dụng AWS S3 để lưu trữ file, như vậy user request một file lên Rails, rails lại request về S3 để generate một link và redirect người dùng đến link đấy. Khi có nhiều resources cần load, việc này trở nên thiếu hiệu quả.

Tuy nhiên, trải nghiệm với Active Storage quá đơn giản, tiện lợi và hoạt động đầy đủ tính năng khiến cháu không muốn đổi sang Paperclip hay CarrierWave. Ngoài ra cái gì đang hoạt động thì tốt nhất không nên động vào =]]

#### Bắt đầu công chuyện

##### Bước 1: Mở Dashboard Cloudflare, vào mục Rules

![](Screen Shot 2021-07-07 at 14.00.46.png)

Các bác tạo Rule mới `Create Page Rule` và config như sau (cái này các bác tự cân đối cho phù hợp nhé)

```
Địa chỉ: <domain>/rails/active_storage/*
Browser Cache TTL: A month
Cache Level: Cache Everything
Edge Cache TTL: 3 month
```

![](Screen Shot 2021-07-07 at 14.04.01.png)

##### Bước 2: Tăng Expire Time hoặc mở public Bucket

Nếu không các bác sẽ bị cache các link đã hết hạn, người dùng sẽ load lỗi. Thêm dòng `public: true` vào trong file `storage.yml`

![](Screen Shot 2021-07-07 at 14.11.13.png)

##### Bước 3: Chỉnh sửa quyền public cho các files đã có trong Bucket

Các bác mở Bucket -> Permission -> Edit

![](Screen Shot 2021-07-07 at 14.16.11.png)

Và bỏ chọn ở mục Block all public access

![](Screen Shot 2021-07-07 at 14.17.14.png)

Sau đó quay lại bucket và mở public cho các file cũ

![](Screen Shot 2021-07-07 at 14.17.39.png)
