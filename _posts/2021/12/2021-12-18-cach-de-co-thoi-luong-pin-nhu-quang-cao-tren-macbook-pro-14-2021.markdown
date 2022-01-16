---
layout: post
title: 'Cách để có thời lượng pin như quảng cáo trên Macbook Pro 14" 2021'
date: 2021-12-18 07:00:00 +0700
categories: macbook, tips
---

Hẳn các bác đã thấy Apple quăng lựu đạn trên keynote như sau

![](Screen%20Shot%202022-01-16%20at%2022.25.52.png)
<center><sup>Có xạo quá không zậy?</sup></center>

Vâng cháu cũng tưởng thật, cho đến khi trên tay thì ối dồi ôi...Pin chắc xài được 5-6 tiếng là max, nghĩ lại có khi còn thua con Zenbook ngày xưa cháu dùng =)) Vậy cháu bắt đầu hành trình đi tìm công thức pin 17 tiếng cho Macbook

# Tắt Siri và Screen Time

Trong lúc nghiên cứu Activity Monitor, cháu nhận thấy một process `corespeechd` chiếm kha khá CPU, hiện tượng này còn rõ rệt hơn khi nghe nhạc qua loa ngoài. Có vẻ như Siri đang có bug gì đó [Link tham khảo](https://tilsupport.wordpress.com/2021/03/27/macbook-pro-2020-high-cpu-caused-by-siri/)

Tiếp theo cháu vô tình đọc được [Screen Time](https://www.reddit.com/r/apple/comments/9wlv0i/has_the_screen_time_battery_drain_been_fixed/) khiến pin tụt trên iOS. Cháu nhận ra tính năng này không hữu ích lắm nên tắt luôn cả điện thoại lẫn máy tính, trên điện thoại có vẻ tắt xong đỡ lag hơn.

# Tắt Boom3D

Không có cái này thì hơi tiếc, vì chỉnh EQ trên Music App không hay lắm.

![](Screen%20Shot%202022-01-16%20at%2022.03.58.png)

# Sử dụng Node 16

Lúc đầu không biết, cứ quen tay cài Node 14 cho tương thích các project đang dùng. Xài M1 thì lên đời Node 16 nhé các bác.

![](Screen%20Shot%202022-01-16%20at%2022.07.27.png)

# Cai nghiện Intellij, đổi sang nVim

Cái này ngày trước thử học Vim được mấy ngày thì bỏ vì mãi không hiểu gì. Chả hiểu sao học Course này tự dưng lại hiểu: [Link Course](https://frontendmasters.com/courses/vim-fundamentals/)

![](Screen%20Shot%202022-01-16%20at%2022.12.31.png)
<center>nVim trông xịn không các bác</center>

# Luôn sử dụng Low Battery Mode

![](Screen%20Shot%202022-01-16%20at%2022.15.22.png)

Hiệu quả tức thì, thời lượng pin tăng tầm 2-3 tiếng. Đổi lại các bác mất ProMotion và CPU sẽ giảm 20% hiệu năng (tương đương M1)

# Giảm tần số quét xuống 47.95Hz

![](Screen%20Shot%202022-01-16%20at%2022.18.46.png)

Tăng pin cực mạnh, thêm hẳn 3 tiếng luôn. Đổi lại, lag thấy rõ :))

# Kết quả

Y như quảng cáo rồi nhá, mà không phải cháu xem Video đâu, ngồi code hẳn luôn. Bình thường code tầm 6 tiếng là hết pin, giờ 15 tiếng là 1.5 ngày không cần sạc rồi :))

![](Screen%20Shot%202022-01-15%20at%2022.13.52.png)