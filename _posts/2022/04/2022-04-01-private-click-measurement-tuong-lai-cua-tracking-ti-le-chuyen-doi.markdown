---
layout: post
title: "Private Click Measurement - Tương lai của tracking tỉ lệ chuyển đổi?"
date: 2022-04-01 07:00:00 +0700
categories: webkit, frontend, web
---

Gần đây đội Webkit từ Apple đã chính thức tung ra một tính năng mới trên Safari 15.4+ gọi là Private Click Measurement (PCM). Tính năng này đã được propose lên W3C từ [2019](https://webkit.org/blog/8943/privacy-preserving-ad-click-attribution-for-the-web/), hiện nay đội Chrome/Firefox vẫn [chưa thống nhất](https://developer.chrome.com/docs/privacy-sandbox/attribution-reporting/) với Webkit về chuẩn API nên implementation sẽ có sự khác nhau.

# 🌿 Vấn đề

Như các bác đã biết trong quảng cáo trực tuyến, theo dõi số lượt click vào quảng cáo và đo lường hiệu quả của các chiến dịch khác nhau. Apple có tiếng cứng rắn về quyền riêng tư, nhưng cấm đoán không phải cách giải quyết vấn đề tốt. Nguyên lí để giải quyết mâu thuẫn lợi ích ở đây là:

✅ Danh tính người dùng sẽ được bảo vệ ở quy mô lớn, thông qua đặc tính sau: Sẽ chỉ có 64 event quảng cáo được đo lường song song. Vì ít như vậy đồng nghĩa Dev không thể gán cho mỗi người 1 chiến dịch quảng cáo.

✅ Browser sẽ thực hiện các bước thu thập & gửi báo cáo thay cho người dùng để đảm bảo tính tuân thủ & quyền được từ chối tham gia của người dùng.

✅ Chỉ cho phép chính chủ website được tiếp cận đến thông tin này, browser chỉ gửi những thông tin khi user truy cập chính website đó. T

# 🌿 Cách hoạt động

## Phase 1: Người dùng click vào đường link của bạn

![](store-ad-clicks-1.png)

Giả sử các bác quăng link sản phẩm A của mình lên Facebook chả hạn. Giờ thẻ <a> sẽ có dạng như thế

```
<a href="https://shop.example.com/product.html" // Link đến sản phẩm
   attributionsourceid="[8-bit source ID]"
   attributeon="https://shop.example" // Domain trang shop của các bác>
  Markup
</a>
```

Có 2 thông số quan trọng ở đây:
`attributionsourceid` Tượng trưng cho nguồn Facebook trong ví dụ, nhưng chúng ta sẽ sử dụng ID. Giá trị từ 0 - 255.

`attributeon` Domain nguồn. Phải là dạng domain cấp 1

Các thông tin về event click sẽ được browser tự lưu & xử lí, Dev không có quyền tiếp cận thông tin này.

## Phase 2: Người dùng trigger hành động conversion

![](match-conversions-ad-clicks-1.png)

Giả sử ở trang mua hàng của các bác sẽ đánh dấu hành động "Thêm vào giỏ hàng" được coi là chuyển đổi thành công. Các bác sẽ cho button thêm vào giỏ hàng redirect hoặc gửi lệnh `GET` đến url của *trang giới thiệu* 

`https://social.com/.well-known/private-click-measurement/trigger-attribution/[Trigger Data]/[Độ ưu tiên]`

Trigger Data bắt buộc có độ dài 2 chữ số. Giá trị từ 00 - 15. Có thể hiểu giống như ID chiến dịch quảng cáo

Độ ưu tiên có giá trị từ 00 - 63

Thông tin này sẽ được lưu hoàn toàn ở browser và sau 24-48h (ngẫu nhiên) sẽ đến Phase 3 

## Phase 3: Browser gửi báo cáo đến chủ website

![](send-ad-click-attribution-data-1.png)

Browser sẽ tự động gửi báo cáo sau 24-48h kể từ lúc có trigger conversion. Browser sẽ send request `POST` đến url của *trang giới thiệu* 

`/.well-known/private-click-measurement/report-attribution/`

Body của message sẽ ở dạng JSON như sau
```
{
  "source_engagement_type" : "click",
  "source_site" : "facebook.com",
  "source_id" : [Source ID],
  "attributed_on_site" : "megatunger.com",
  "trigger_data" : [Trigger Data],
  "version": 1
}
```


Nguồn:
- [https://webkit.org/blog/11529/introducing-private-click-measurement-pcm/](https://webkit.org/blog/11529/introducing-private-click-measurement-pcm/)
- [https://webkit.org/blog/8943/privacy-preserving-ad-click-attribution-for-the-web/](https://webkit.org/blog/8943/privacy-preserving-ad-click-attribution-for-the-web/)
- [https://privacycg.github.io/private-click-measurement/](https://privacycg.github.io/private-click-measurement/)