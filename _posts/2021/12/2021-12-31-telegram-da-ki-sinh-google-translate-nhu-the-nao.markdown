---
layout: post
title: "Telegram đã kí sinh trên API của Google Translate như thế nào"
date: 2021-12-31 07:00:00 +0700
categories: telegram, google translate, api
---

Như các bác đã biết, dịch vụ của Google thì miễn phí nhưng API thì xoè tiền ra. Xoè rất nhiều tiền là đằng khác. Theo như API của Google Translate chính thức 
[Cloud Translate API](https://cloud.google.com/translate/docs/reference/rest/v2/translate) nó sẽ có dạng như thế này `/language/translate/v2` và dĩ nhiên tham số truyền lên sẽ bao gồm cả API Key

Tuy nhiên gần đây Telegram có phát hành một tính năng mới gọi là *Message Translation* được implementation một cách đầy ảo thuật [Source Github](https://github.com/DrKLO/Telegram/commit/9e740dfd4d2b1ab6b8ed2b972e0f72fc9b8bd09d#diff-bc405602f072ccdb4e595ac9f577f6bfae72778c6a989bf81021b79cfef28568R1081)

```java
uri = "https://translate.goo";
uri += "gleapis.com/transl";
uri += "ate_a";
uri += "/singl";
uri += "e?client=gtx&sl=" + Uri.encode(fromLanguage) + "&tl=" + Uri.encode(toLanguage) + "&dt=t" + "&ie=UTF-8&oe=UTF-8&otf=1&ssel=0&tsel=0&kc=7&dt=at&dt=bd&dt=ex&dt=ld&dt=md&dt=qca&dt=rw&dt=rm&dt=ss&q=";
uri += Uri.encode(text.toString());
```

Nhìn cái biết ngay dân chơi dùng API private rồi. Đầu tiên dân chơi sử dụng URL `https://translate.googleapis.com/translate_a`, một địa chỉ không chính thức. Ngoài ra, vì sao  Telegram phải tách ra thành từng đoạn, có lẽ để tránh bị quét tự động trên Github.

### Các tham số có thể được giải thích như sau:
- `client` xác định web/app?
- `sl` và `tl`: ngôn ngữ đầu vào - đầu ra
- `ie` và `oe` loại mã hoá đầu vào - đầu ra
- `ssel` và `tsel` biết phát chết liền
- `q` đoạn chữ cần dịch

### Xoay vòng User agent

```java
private String[] userAgents = new String[] {
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.45 Safari/537.36", // 13.5%
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36", // 6.6%
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:94.0) Gecko/20100101 Firefox/94.0", // 6.4%
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:95.0) Gecko/20100101 Firefox/95.0", // 6.2%
  "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36", // 5.2%
  "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.55 Safari/537.36" // 4.8%
};
```

- Một chiêu thức cổ điển để tránh bị phát hiện Spam, Telegram sẽ xoay vòng ngẫu nhiên trong list agent này

```java
connection.setRequestProperty("User-Agent", userAgents[(int) Math.round(Math.random() * (userAgents.length - 1))]);
```

Thời buổi kinh tế khó khăn, lại còn không thu phí người dùng, nên chắc anh em nhà Durov đang tăng xin giảm mua 😀

![](16978756.jpeg)



Bài gốc: [https://danpetrov.xyz/programming/2021/12/30/telegram-google-translate.html](https://danpetrov.xyz/programming/2021/12/30/telegram-google-translate.html)