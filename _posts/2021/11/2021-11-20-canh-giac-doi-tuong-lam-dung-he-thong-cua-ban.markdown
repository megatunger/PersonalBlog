---
layout: post
title: "Cảnh giác đối tượng lạm dụng hệ thống của bạn"
date: 2021-11-20 07:00:00 +0700
categories: attack, hacker, security, crawler
---


![](looting.gif)

## Crawler

Về luật pháp, việc crawl dữ liệu đã được public là không vi phạm pháp luật. Nhưng tự dưng danh sách sản phẩm của mình lại hiển thị trên trang đối thủ, dữ liệu khách hàng của mình được rao bán công khai trên mạng thì hậu quả là nhãn tiền. Các bác nên kiểm tra các hướng sau:

#### Phía API
- Có API nào có thể get những dữ liệu mà không thuộc của họ không. VD: /users/1 nhưng lại đi get /users/3 chả hạn
- Không sử dụng ID dạng số vì dễ dàng đoán được
- Kiểm tra phân quyền user

#### Phía Server Side Rendering
- Các resources sử dụng pagination cần có limit rate
- Sử dụng Cloudflare để chặn unusual activity, thêm reCaptcha

#### Phía Client
- Làm rối tên class CSS (sử dụng Webpack, Styled Component, Emotion,...)
- Render text thành nhiều span khác nhau
- Dynamic semantic tag (VD lúc hiển thị div lúc thì là section,...)

#### Phía Data
- Sử dụng kĩ thuật Data Footprint để truy được nguồn gốc khi có data leak (VD: thêm dấu cách, kí tự ẩn theo một quy luật vào các trường thông tin)

<br>

## Gian lận referral, affiliate, khuyến mãi

Thu hút người dùng bằng các chương trình tiếp thị liên kết là một phương thức phổ biến hiện nay. Vấn đề thường mắc phải ở đây là không có cơ chế để xác thực user đấy có phải là bot không. Sự thật là trên facebook có những hội nhóm hàng ngày mua bán tool để đăng kí hàng nghìn, hàng triệu nick ảo có chức năng tự liên kết lẫn nhau để chiếm đoạt các giải thưởng. Các bác nên kiểm tra kĩ những vấn đề sau trước khi chạy campaign

![](agentSmith.jpg)

#### Hạn chế register bừa bãi

- Với các form đăng kí, điều không thể thiếu là dịch vụ captcha như reCaptcha.
- Với các chương trình giá trị cao, KYC nên là một điều cần thiết
- Monitoring hành vi register tài khoản liên tục, thời gian đều đặn với các thông tin rất chính xác, chuẩn mực (VD tự dưng các bác thấy DB có một loạt Nguyễn Văn A, Nguyễn Văn B, Nguyễn Văn C liên tiếp nhau)
- Limit rate theo IP, Device

#### Phần thưởng không nên là cách nào đó có thể đổi ra tiền, thẻ nạp

Đầu tiên, các bác cần chắc chắn từ chối trao giải nếu người nhận thưởng và thông tin đăng kí account không giống nhau, và nhớ chụp lại bằng chứng không họ sửa lại thông tin :v

Tiếp theo, cháu thấy có nhiều ứng dụng cho phép rút thu nhập bằng tiền mặt, thì các bác cần đảm bảo chủ tài khoản trùng với tên đăng kí. Có những app còn tệ hơn là cho đổi sang **thẻ nạp**. Đổi sang thẻ nạp là phương thức các khách không mời này thích nhất, vì tính ẩn danh của nó. Thị trường trao đổi thẻ nạp cực kì nhộn nhịp và dễ dàng nên các bác cho đổi ra thẻ nạp là tự dưng thành miếng mồi ngon cho xã hội.

Giải pháp
- Tặng phần thưởng **gán với tài khoản**, chỉ có tài khoản đó sử dụng được
- Nói không với đổi ra thẻ nạp

<br>

## Brute Force OTP

![](2GU.gif)

Thế nào đó có rất nhiều website chú ý chặn brute force với password, nhưng lại không làm với mã OTP. Dẫn đến một vài ví dụ sau:
- Đăng nhập bằng SDT random nào đó, cho chạy tool thử OTP (có từ 1 -> 999999 thôi, nhanh lắm) thế là login được
- Quên mật khẩu, cũng nhập bừa SDT nào đó, rồi đổi mật khẩu của họ
- Tự dưng thiệt hại cả mớ tiền tin nhắn, cuộc gọi OTP
- Khách hàng lạ nào đó tự dưng bị đăng kí dịch vụ của các bác

#### Giải pháp
- Khuyến khích khách hàng sử dụng 2FA
- Limit IP, Device. Đặc biệt chú ý account của admin
- Yêu cầu khách hàng confirm account khi đăng kí
- Lock account nếu vượt quá số lần đăng nhập không thành công
- Login có timeout
