---
layout: post
title: "Điều Flutter luôn giữ kín trong tim"
date: 2021-02-01 07:00:00 +0700
categories: flutter, mobile app
---

Chúng ta của hiện tại, em dành cả thanh xuân cho React Native. Anh dành cả thanh xuân để debug. Chúng ta dành cả thanh xuân cho nhau mà không hề nghĩ suy. Gặp nhau là duyên phận, xa nhau cũng là duyên phận. Chẳng ai biết tương lai sau này. Dù sau này có nhau hay không thể bên nhau cũng đừng quên rằng chúng ta đã dành tất cả điều tuyệt vời nhất trên npm.js. Thương React Native.

## There's an Widget for that.

Ở thập kỉ trước, "There's an app for that" là khẩu hiệu của nhà nhà, của mọi công ty đều muốn xây dựng app. Với Flutter, cháu xin cải biên khẩu hiệu trên thành "There's an widget for that". Đủ để các bác biết là Flutter nó xây dựng sẵn một bộ Components/Widget đa dạng thế nào rồi đó. Giờ cháu sẽ giới thiệu các bác một vài Widget có một không hai:

#### Hero

Widget làm animation mà cháu thích nhất. Widget này sẽ tự biến đổi, di chuyển component của các bác từ màn hình này sang màn hình khác một cách liền mạch nhất có thể.

![](hero_1.gif) | ![](hero_2.gif)

Cách sử dụng thì không thể đơn giản hơn

```dart
Hero(
    tag: <ID phân biệt element đó>,
    child: <Phần tử cần áp dụng>
)
```

[Documentation](https://flutter.dev/docs/development/ui/animations/hero-animations#basic-structure-of-a-hero-animation)

#### ListWheelScrollView

Một danh sách cuộn lên cuộn xuống tương đối nhàm chán? Giờ thêm tí hiệu ứng 3D cho zui nha.

{% youtube "http://www.youtube.com/watch?v=dUhmWAz4C7Y" %}

#### PhysicalModel

Tạo cảm giác chiều sâu bằng hiệu ứng đổ bóng (Shadow), quan trọng nhất: dùng được cho mọi Widget (bình thường sẽ phải tìm cách wrap vào Card)

{% youtube "http://www.youtube.com/watch?v=XgUOSS30OQk" %}

#### InteractiveViewer

Tạo một khung ảnh zoom ra zoom vào dễ hơn cả code Hello World

{% youtube "http://www.youtube.com/watch?v=zrn7V3bMJvg" %}

#### AnimatedIcon

Đọc phát biết làm gì luôn.

{% youtube "http://www.youtube.com/watch?v=pJcbh8pbvJs" %}

#### IndexedStack

Giữ nguyên state khi chuyển Widget (hơi khó hiểu lúc đầu nhưng các bác hãy nghĩ đến ví dụ là làm một cái BottomNavigationBar thì khi chuyển tab thì trạng thái phải giữ nguyên đúng không)

{% youtube "http://www.youtube.com/watch?v=_O0PPD1Xfbk" %}

...tạm thế, code gần 1 năm rồi cháu vẫn chưa thuộc hết đống Widget đâu =))

## Flutter Inspector

Có một bệnh ôn dịch của các framework "react" là **RE-RENDER**. Bệnh này do các Dev lười (như cháu) không set key, `useEffect` vô tội vạ, thay đổi 1 props nhưng cho render lại hết cả những thứ không liên quan. Tưởng tượng giống như kiểu có 1 F0 nhiễm Covid nhưng cho xét nghiệm đến tận...F5 vậy. Rất lãng phí tài nguyên.

Flutter không thoát được đâu, các bác mà không code theo kiến trúc cụ thể như `BLOC` thì xác định app toang dần đều nha =)). Tuy vậy, chúng ta vẫn may mắn vì có bộ Inspector/Profiler ngon hơn cả Ngọc Trinh.

![](Screen Shot 2021-02-03 at 01.37.41.png)

Chức năng đỉnh nhất: Track widget rebuilds. Nó giúp track cái widget bác đang dùng có những thành phần nào đang rerender, render bao nhiêu lần. Nếu có một thành phần đang render nó cũng hiện luôn một cái vòng xoay xoay bên cạnh giúp các bác biết ngay thằng nào đang có dấu hiệu không bình thường

## Và những điều giấu kín trong tim

#### Issues

Thật sự cháu rất trầm cảm khi thấy số lượng issues tăng nhanh như giá cổ phiếu vậy. 6 tháng tăng độ 3k issues :)

![](Screen Shot 2021-02-03 at 00.38.01.png)

#### Không có CodePush

Vì nó compile ra native code, các bác định làm thủ thuật replace code trong lúc runtime kiểu gì:))

#### Không chơi thân với hàng xóm Apple lắm

- Bộ CupertinoDesign (mang design iOS lên App): Được Google coi như công dân hạng 2. Dùng bộ này xong App của các bác sẽ trông giống như iPhone chạy Android

![](IMG_3659.PNG)
**_Không chút giả trân luôn_**

- App nặng hơn tầm 50% & nóng máy hơn so với Android
- Build iOS cực chậm, lắp Firebase vào thì các bác nhớ lắp [cái này](https://github.com/invertase/firestore-ios-sdk-frameworks) vào nha. Không là build 20 phút cho một cái app Hello World thì hông ai độ được đâu:))

#### Bộ Web không ngon lắm

Cháu dùng rồi, thấy bộ Web còn nhiều lỗi lắm:

- Setup xong code một hồi thì một loạt lỗi tâm linh xuất hiện, không build được mobile app nữa. Tứk
- Chưa có Hot Reload. Sửa một cái Button thôi mà chờ nó load lại cả app.
- Chậm, chậm vl. NextJS, React, Angular,.. chắc cười khinh bỉ
- Vấn đề cốt lõi: Giao diện Desktop/App nó khác nhau một trời một vực nên các bác hãy nghĩ là nó đang giải quyết bài toán PWA cho Mobile nhá, không phải là mang App lên Desktop dùng đâu:))) Âm điểm UX luôn đó.
