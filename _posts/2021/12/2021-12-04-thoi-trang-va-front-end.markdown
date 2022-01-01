---
layout: post
title: "Thế giới Web và tính xoay vòng -  Phần 1: Cuộc chiến Server Side vs Client Side"
date: 2021-12-04 07:00:00 +0700
categories: react, frontend, nextjs, ssr, csr
---

# Phần 1: Cuộc chiến Server Side vs Client Side

![](SSR-vs-CSR-2.jpg)

## 2000-2010: Server Side Rendering (SSR)

Trong thời kì đầu, các website còn đơn giản nên các nhà phát triển sử dụng render template của PHP, .NET, Ruby on Rails để vừa "nhồi" code logic, vừa render được ra HTML. Cách làm này thật ra khá...thuận tiện cho người Dev vì kiểm soát logic rất dễ, code nhanh.

![](41062018-b120a09e-6991-11e8-8d36-3f588688105f.png)

Tưởng tượng các bác được giao features hiển thị danh sách Users. Lúc đó, các bác chỉ việc lưu mảng các object ở Controller, đến bước View bác cho một vòng lặp với mỗi object thì tạo ra 1 phần tử <li> chả hạn. Mọi thứ nó tuần tự theo đúng flow các bác nghĩ nên thật ra code có khi còn nhanh hơn mấy đồ chơi hiện đại

Mọi thứ có vẻ OK trong 1 thập kỉ cho đến khi người ta bắt đầu nghĩ đến việc chuyển các app Desktop lên nền tảng Web. Chả hạn bây giờ, tôi muốn sử dụng Word, Excel mà không cần phải tải một bộ cài vài trăm MB. Các dịch vụ như thế này gây áp lực cực lớn lên hệ thống server, và rất khó để scale theo chiều ngang vì mọi thứ đều là stateful 

## 2010s: Client Side Rendering (CSR)

Developers bắt đầu biết rằng, việc tải cả một đoạn HTML to bổ bố dù tối ưu thế nào thì cũng chậm, chi bằng có cách nào đó để phía Client "san sẻ công việc". Họ mới nghĩ ra cách khác, Server sẽ chỉ gửi đoạn JSON chứa data cần render, Client sẽ đọc data đó và render ra phần tử HTML.

![](innovation.gif)

Vào thời kì đầu, bộ môn jQuery và AJAX vẫn gánh vác cả thế giới Web nhưng điểm yếu chết người của nó là: *Vô tổ chức* . Việc truy cập các phần tử DOM thoải mái ở mọi chỗ bằng dấu `$` đã làm việc phát triển một Web App trở thành cơn ác mộng. Tưởng tượng các bác đang có một phần tử hiển thị trên màn hình xong tự dưng một hàm của đứa intern nào đó remove tất cả phần tử theo class và vô tình dính luôn cái của các bác.

![](chaos.gif)

Về lí thuyết, trước khi có React các bác đã có thể làm được tất cả mọi thứ rồi. Nhưng cái thời đấy thiếu là một cách tư duy để phát triển dự án Web bền vững. Đó là lúc sự xuất hiện của **Angular, Ember.js, Vue.js React**.

- React hướng mọi người theo tư duy "Component", mỗi phần tử HTML giờ sẽ bao hàm cả những trường hợp có thể xảy ra (side-effect), sự kiện (events) và người ta thiết kế web theo cách ghép các khối nhỏ vào nhau.

- Các framework này đều tận dụng lợi thế của Virtual DOM để kiểm soát & làm quá trình render hiệu quả hơn

Kết hợp với WebAssembly chúng ta có một thập kỉ bùng nổ các web app, mọi ứng dụng desktop đều có thể biến thành một ứng dụng web.

![](lifechanging.gif)

Tuy nhiên, một bộ phận nhỏ lại tiếp tục gặp vấn đề...

## 2020s: Server Side Rendering trở lại

Khi các trang thiên về nội dung như tin tức, ecommerce,...sử dụng CSR họ không thể SEO được. Tất cả những gì Crawler nhìn thấy là một file index.js, chấm hết. Crawler không phải là một trình duyệt nên họ không thể kì vọng nó sẽ bật trang web lên rồi chờ nó đọc file JS biến thành HTML.

![](noseo.gif)

Một giải pháp đau não cho vấn đề tiền đình này xuất hiện:
- Ở phía server sẽ đọc file js -> Render ra HTML
- File HTML này kèm theo JS được trả về Client cho lần đầu tiên mở trang Web
- Client render phần HTML & CSS như bình thường
- Sau khi render xong bắt đầu quá trình nối các phần tử trong DOM thật vào Virtual DOM của React để biến thành một App React (dehydrate)

Đó chính là Next.js. Như các bác thấy, giải pháp này lai giữa SSR và CSR, và có thể nói Next.js là giải pháp phát triển app React hoàn thiện nhất trong thời điểm hiện tại.

![](nextjs.png.webp)

*(Còn tiếp) Phần 2: Cuộc chiến Real DOM vs Virtual DOM*