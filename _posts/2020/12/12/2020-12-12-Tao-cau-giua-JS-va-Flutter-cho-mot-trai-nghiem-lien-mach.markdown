---
layout: post
title:  "Tạo cầu (Bridge) giữa Javascript và Flutter cho một trải nghiệm liền mạch"
date:   2020-12-12 07:00:00 +0700
categories: flutter, javascript, webview
---

#### Luận bàn về sử dụng WebView trong Mobile App

Nhờ có sự phát triển của các framework giúp xây dựng Web App như (React, Angular, Vue,...), kết hợp với sức mạnh ngày càng khủng của các thiết bị di động mà các lập trình viên ngày càng ưa thích việc sử dụng WebView (nhúng trang web lên app điện thoại).

Một ví dụ điển hình là ứng dụng Shopee, các bác có bao giờ thắc mắc tại sao các bác không cập nhật App mà ứng dụng lại có thể "biến hình" mỗi đợt khuyến mãi? Nhưng thật kì lạ rằng các bác vẫn sử dụng ứng dụng một cách liền mạch mà không hề có cảm giác đang vào trang Web đúng không?

[<img src="thinking.gif" width="100"/>](image.png)
 
 Vì các ứng dụng này đã tạo rất nhiều cầu nối (Bridge) giữa Javascript và Native cực kì "điêu luyện". Hôm nay cháu sẽ hướng dẫn cách tạo Bridge để điều hướng màn hình trên Flutter và tạo một trải nghiệm liền mạch giữa WebView và Native App

#### B1: Tạo Router

Bình thường nếu mới học Flutter, các bác sẽ hay viết kiểu:

``` dart
Navigator.push(context, gì gì đó)
```

Tuy nhiên, về lâu dài cháu thấy cách viết này gây ra rất nhiều vấn đề về kiểm soát luồng sử dụng App và quan trọng là không tạo được Bridge để push màn hình.

Vì vậy, cháu khuyến khích mọi người tạo Router và sử dụng `Navigator.pushNamed` để quản lý như thế này

**router.dart**
```dart
class YourAppRouter {
  static Route<dynamic> generateRoute(RouteSettings settings) {
    switch (settings.name) {
      case YourAppRoutes.route1:
        return MaterialPageRoute(builder: (_) => Route1Widget());
      case YourAppRoutes.route2:
        var routeParams = settings.arguments as String;
        return MaterialPageRoute(
            builder: (_) => Route2Widget(routeParams: routeParams));
      default:
        return MaterialPageRoute(
            builder: (_) => Scaffold(
                  appBar: AppBar(),
                  body: Center(
                      child: Text('Fallback')),
                ));
    }
  }
}

class YourAppRoutes {
  static const String route1 = /route1';
  static const String route2 = '/route2';
}

```

và trong **main.dart** các bác config như sau:

```dart
class App extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      onGenerateRoute: YourAppRouter.generateRoute,
      initialRoute: YourAppRouter.route`,
      ....
    );
  }
}
```

#### B2: Tạo Bridge trong WebView

Giả sử các bác có một trang Notifications như sau

và code của WebView đó
```dart
return WebView(
                key: _key,
                initialUrl: this.widget.url,
                javascriptMode: JavascriptMode.unrestricted,
                onPageFinished: (finish) {
                  setState(() {
                    isLoading = false;
                  });
                },
                onPageStarted: (String url) {
                  setState(() {
                    isLoading = true;
                  });
                },
              );

```

giờ hãy thêm `JavascriptChannel`

```dart
                javascriptChannels: Set.from([
                  JavascriptChannel(
                      name: 'PushScreen',
                      onMessageReceived: (JavascriptMessage message) {
                        print(message.message);
                        var pushScreenNotification =
                            PushScreenNotification.fromJson(
                                jsonDecode(message.message));
                        Navigator.pushNamed(
                            context, pushScreenNotification.screen,
                            arguments: pushScreenNotification.params);
                      }),
                ]),
```

WebView này sẽ lắng nghe liên tục tại kênh `PushScreen`, để lấy message ta gọi `message.message` trong Flutter

***Tại phía Web***, cháu sẽ tạo một object có dạng như sau
```js
const screenNeedToPush = {
    screen: '/route2',
    params: '<PARAMS_HERE>`
}
```
và tại button nào đó cháu sẽ gọi
```js
PushScreen.(screenNeedToPush);
```
mục đích là cháu muốn truyền 2 thứ: *Màn hình cần push* + *Tham số cho màn hình này*

#### B3: Kiểm nghiệm

Nhờ việc xây dựng Bridge như này, cháu có thể tạo một tính năng là Push màn hình tương ứng từ trang thông báo (có nhiều loại thông báo khác nhau)

![](ezgif-3-2154c891b910.gif)