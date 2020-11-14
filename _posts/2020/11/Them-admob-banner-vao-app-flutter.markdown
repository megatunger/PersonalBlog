---
layout: post
title:  "Thêm Admob Banner vào app Flutter"
date:   2020-11-14 15:54:00 +0700
categories: admob, flutter, lua lep
---


## Thêm Admob Banner vào app Flutter

Như các bác đã biết, Google Admob là nền tảng để Google hiển thị quảng cáo trên App của các bác, đồng thời giúp các bác có thể kiếm thêm chút tiền cafe hằng tháng. Hôm nay cháu xin giới thiệu cách để tích hợp vào app Flutter của các bác, hiển thị cố định ở cạnh dưới trên App.

Tư liệu thực nghiệm hôm nay là app **Lúa Lếp**. Sản phẩm bài tập lớn môn Xử lí tiếng nói của nhóm cháu. Có gì các bác tải trên [Appstore](https://apps.apple.com/vn/app/lúa-lếp/id1519995438) hoặc [Google Play](https://play.google.com/store/apps/details?id=com.megatunger.lualepapp) ủng hộ cháu nha.

OK, bắt đầu nào.

#### Chuẩn bị

- Đương nhiên là phải có một tài khoản Admob rồi nhá
- Tốt nhất là tài khoản đó nên được duyệt rồi, không là hổng có hiện ra cái gì đâu.

Mở dashboard [Admob](https://apps.admob.com/) lên đầu tiên nè. Sau đó các bác thêm app, thêm các unit quảng cáo giùm cháu nha. Đây là hình ảnh sau khi cháu thêm

![Admob](Screen Shot 2020-11-14 at 16.08.11.png)

Sau đó các bác lên **pub.dev** cài thư viện `firebase_admob` config các thứ cho đúng yêu cầu nha. Docs họ hay thay đổi nên cháu nghĩ không cần thiết viết ở đây.

#### Tạo một class quản lí Ads

Tiếp các bác tạo giúp cháu một file `ad_manager.dart`. Mục đích chính là quản lí các mã ID banner.

``` dart
import 'dart:io';

class AdManager {
  static String get appId {
    if (Platform.isAndroid) {
      return "<YOUR_ANDROID_ADMOB_APP_ID>";
    } else if (Platform.isIOS) {
      return "<YOUR_IOS_ADMOB_APP_ID>";
    } else {
      throw new UnsupportedError("Unsupported platform");
    }
  }

  static String get bannerAdUnitId {
    if (Platform.isAndroid) {
      return "<YOUR_ANDROID_BANNER_AD_UNIT_ID";
    } else if (Platform.isIOS) {
      return "<YOUR_IOS_BANNER_AD_UNIT_ID>";
    } else {
      throw new UnsupportedError("Unsupported platform");
    }
  }
}
```

#### Tạo bloc nhận event Admob

Các hàm trên Admob ở dạng bất đồng bộ nên các bác cần có cơ chế để tiếp nhận các sự kiện admob bắn về. Ở đây cháu dùng kiến trúc `bloc` với `rxDart` để xử lí phần này

**admob_bloc.dart**
``` dart
import 'package:rxdart/rxdart.dart';

class AdmobBloc {
  final BehaviorSubject<bool> _bannerAdInit = BehaviorSubject<bool>();

  showBannerAd({bool state}) async {
    if (state) {
      _bannerAdInit.sink.add(true);
    } else {
      _bannerAdInit.sink.add(false);
    }
  }

  dispose() {
    _bannerAdInit.close();
  }

  BehaviorSubject<bool> get bannerAdInit => _bannerAdInit;
}

final admobBloc = AdmobBloc();

```

#### Tích hợp vào app

Bây giờ cháu muốn banner nó cố định ở cạnh dưới của app ở chỗ này

![](Screen Shot 2020-11-14 at 16.22.14.png)

Nếu `firebase_admob` mà ở dạng Widget flutter thì mọi thứ đã thật đơn giản. Nhưng đời không như vậy các bác ạ, cháu chả hiểu sao họ làm theo kiểu gọi Platform Channel (nôm na là họ tạo một cái view đè lên trên nền app Flutter đang chạy). Vì vậy khá khó chịu khi phải layout cái cục này trên màn hình.

Giờ cháu phải tìm cách để resize toàn bộ App cho nó tạo khoảng trống ở đáy màn hình. Nếu nó init không thành công thì phải đưa cái paddingBottom về 0.

- Đầu tiên các bác chuyển cái main App sang dạng StatefulWidget (để có thể thay đổi được paddingBottom theo state của admob)

![](Screen Shot 2020-11-14 at 16.32.55.png)

- Tiếp đến custom builder cho cái component MaterialApp

**main.dart**
``` dart
return MaterialApp(
      title: 'Lúa Lếp',

      builder: (context, widget) {
        return StreamBuilder(
          stream: admobBloc.bannerAdInit,
          builder: (BuildContext context, AsyncSnapshot<bool> snapshot) {
            if (snapshot.hasData && snapshot.data == true) {
              return Container(
                  color: Colors.black,
                  child: Padding(
                    padding: EdgeInsets.only(bottom: paddingBottom),
                    child: SafeArea(child: widget),
                  ));
            }
            return widget;
          },
        );
      },
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: OnBoarding(),
      navigatorObservers: [
        FirebaseAnalyticsObserver(analytics: analytics),
      ],
    );
  }
```

nhớ tạo một biến (56 là kích thước chiều cao của banner)

```dart
var paddingBottom = 56.0;
```

ở hàm initState các bác khởi tạo bloc

**main.dart**
```dart
  @override
  void initState() {
    admobBloc.showBannerAd(state: false);
    super.initState();
  }
```

Các bác tạo thêm một class để ẩn/hiện, init ad

**ads_helper.dart**

```dart

class AdsHelper {
  static BannerAd _bannerAd;

  static void showPopupAd() {
    if (_popupAd == null) _popupAd = _createPopupAd();
    _popupAd
      ..load()
      ..show();
  }

  static void hidePopupAd() async {
    if (_popupAd != null) {
      await _popupAd.dispose();
      _popupAd = null;
    }
  }

  static BannerAd _createBannerAd() {
    return BannerAd(
      adUnitId: AdManager.bannerAdUnitId,
      size: AdSize.fullBanner,
      listener: (MobileAdEvent event) {
        print("BannerAd event $event: ${event.index}");
        if (event.index == 0) {
          admobBloc.showBannerAd(state: true);
        }
      },
    );
  }

  static void showBannerAd() {
    if (_bannerAd == null) _bannerAd = _createBannerAd();
    _bannerAd
      ..load()
      ..show(anchorType: AnchorType.bottom);
  }

  static void hideBannerAd() async {
    if (_bannerAd != null) {
      await _bannerAd.dispose();
      _bannerAd = null;
    }
  }
}

```

quay trở lại hàm initState của main.dart các bác cho bật ad

**main.dart**
```dart
  @override
  void initState() {
    admobBloc.showBannerAd(state: false);
    super.initState();
    Ads.showBannerAd();
  }
```

Lên rồi nè ahihi

![](Screen Shot 2020-11-14 at 16.48.50.png)

