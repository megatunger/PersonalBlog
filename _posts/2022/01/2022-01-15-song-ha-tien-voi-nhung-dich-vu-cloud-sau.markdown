---
layout: post
title: "Sống hà tiện với những dịch vụ cloud sau"
date: 2022-01-15 07:00:00 +0700
categories: aws, bunny, vultr, hosting, vultr, cloudflare
---

Lượng Free Credits mà các dịch vụ cloud nổi tiếng như AWS, Google Cloud, Azure cung cấp trông có vẻ hào phóng cho người dùng, nhưng đảm bảo các bác dựng xong trang web xinh xinh cũng là lúc...hết tiền. Và dần dần bọn họ sẽ vặt lông các bác bằng các hoá đơn chi phí cloud trên trời. Đơn cử ví dụ AWS S3 chi phí dev hàng tháng có khi chỉ 2-10$, nhưng thực tế giá bandwidth của nó tận 50$/1TB, có nghĩa là làm trang web lưu trữ chia sẻ asset mà vài triệu lượt truy cập mỗi tháng thì chỉ còn cái nịt. 

Vì vậy để tránh bị Sales AWS thuốc các bác nên biết các giải pháp Replica sau (chất lượng 99% nha):

## Lưu trữ: Backblaze:

![](Screen%20Shot%202022-01-15%20at%2019.53.40.png)

Giá [Backblaze](https://www.backblaze.com/b2/cloud-storage-pricing.html) chấp hết các anh tài, giá bằng 1/3. Tính năng Rep 1:1 nên yên tâm luôn nha. Phần hay nhất ở đây là Backblaze có hợp tác với Cloudflare, gọi là [Bandwidth Alliance](https://www.cloudflare.com/en-ca/bandwidth-alliance/backblaze/) nên nếu các bác dùng Cloudflare các bác sẽ không bị tính traffic download

## Cache hết mọi thứ: Cloudflare

Cloudflare sẽ đẩy asset của các bác về server edge của họ, để khi user download một file, nó sẽ hit vào server của họ, vừa tăng tốc độ tải, vừa giảm được chi phí server của các bác. Bản free mặc định đã có cái này nhưng các bác chỉ được config 3 page rules, và thường sẽ không đủ để config hết cho các dịch vụ các bác. 

Các bác có thể nâng cấp gói Cloudflare Pro (20$/tháng/domain), config khéo một tí các bác có thể hit cache lên tới 85%

## Deploy Website: Cloudflare & Vercel

Nếu website các bác tĩnh hoàn toàn, hãy tham khảo [Cloudflare Pages](https://pages.cloudflare.com), mọi Static Website ở đây BUILD & DEPLOY đều không mất phí và UNLIMITED BANDWIDTH (chắc không có ai chơi lớn thế này đâu). 

![](Screen%20Shot%202022-01-15%20at%2020.43.49.png)

Nếu website các bác cần có serverless đứng sau, hãy tham khảo Vercel, giá của họ là 20$/tháng cho 1TB Bandwidth và Unlimited Function Requests

![](Screen%20Shot%202022-01-15%20at%2020.42.51.png)

Nếu các bác cảm thấy mình nhiều tiền quá và nghiện AWS thì dịch vụ tương đương này sẽ là AWS Amplify (không phải S3 đâu, các bác quên tính chi phí build rồi). Giá mà tính ra chắc phải gấp vài chục lần những cái tên còn lại :))

![](Screen%20Shot%202022-01-15%20at%2020.45.05.png)
<sub><sup>Báo giá sương sương giấu tên</sup></sub>

## Server: Vultr

Quên EC2 đi, [Vultr](https://www.vultr.com/products/cloud-compute/) mới là chân ái. Cháu có kha khá project dùng Vultr rồi, thấy chất lượng ổn, chưa bị downtime lần nào, tốc độ mạng tương đương ap-southeast-1

![](Screen%20Shot%202022-01-15%20at%2020.36.56.png)

Giá trên là giá Vultr, x2 lên các bác có giá AWS EC2 :)

## Lưu ảnh: [Bunny.net](https://bunny.net)

Chắc các bác sẽ hỏi:
- Tại sao không lưu hết vào Backblaze? Vì đây là CDN, tốc độ khi access các file nhỏ sẽ nhanh hơn một dịch vụ mang tính "backup"
- Tại sao không dùng Cloudflare? Vì Bunny có thêm dịch vụ tối ưu ảnh xịn xò

![](Screen%20Shot%202022-01-15%20at%2020.38.29.png)