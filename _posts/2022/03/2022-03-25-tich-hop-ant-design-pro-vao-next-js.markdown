---
layout: post
title: "Làm thế nào để Ant Design Pro nương tựa vào Next.js"
date: 2022-03-25 07:00:00 +0700
categories: ant design, nextjs, frontend
---

Đường đường nhà trai là Framework React xịn top 1 server, nhà gái cũng là UI Kit nhiều ⭐️ nhất Github, gia thế không có gì ngoài bộ Components cực khủng. Ấy vậy mà 2 anh em này không nương tựa nhau lắm.

# 🌿 Tích hợp Antd lên Next.js

Việc cài đặt lúc đầu cũng chưa có gì khó, chỉ cần gõ `yarn add antd` và thêm dòng
```
// trong file _app.tsx

import "antd/dist/antd.css";
```

OK, thế là tạm lên. Nhưng...

🤔 Giờ sửa màu primary thế nào?<br>
🤔 Ủa sao sửa tí mà phải lằng nhà lằng nhằng trong webpack config vậy?<br>
🤔 Ủa sao Ant Design Pro không có hướng dẫn cho SSR như Next.js vậy?<br>
🤔 Ủa sao lỗi tùm lum vậy?<br>
..........

➡️ Next.js hỗ trợ SASS, SCSS nhưng không hỗ trợ LESS - thứ mà Ant Design đang dùng trong mọi ngóc ngách. Trước đây, bản Next.js 9 cũng hõ trợ đấy nhưng chả rõ vì sao sau bỏ luôn.

➡️ Bản thân Ant Design từ đầu đã có khung "toolchain" khác người rồi. Ant Design sử dụng `UmiJS` (hàng nhà làm) để navigate, quản lí assets, hot reload, build bundle,...

## next-plugin-antd-less

✅ Bước đơn giản đầu tiên để giải quyết mọi vấn đề nhức đầu là dùng package [next-plugin-antd-less](https://www.npmjs.com/package/next-plugin-antd-less).

```
yarn add next-plugin-antd-less
```

Các bác mở next.config.js, copy & paste vào.
```
const withAntdLess = require('next-plugin-antd-less');

module.exports = withAntdLess({
  modifyVars: { '@primary-color': '#04f' }, // neu khong luu ra file
  lessVarsFilePath: './src/styles/variables.less', // neu luu ra file rieng 
  lessVarsFilePathAppendToEndOfContent: false,
  cssLoaderOptions: {
    // ... 
    mode: "local",
    localIdentName: __DEV__ ? "[local]--[hash:base64:4]" : "[hash:base64:8]",
    exportLocalsConvention: "camelCase",
    exportOnlyLocals: false,
    // ...
    getLocalIdent: (context, localIdentName, localName, options) => {
      return "whatever_random_class_name";
    },
  },

  nextjs: {
    localIdentNameFollowDev: true, 
  },
  webpack(config) {
    return config;
  },
});
```

Trong config kia sẽ có đoạn `localIdentName` `getLocalIdent` có thể gây khó hiểu với 1 vài bác. Thật ra vấn đề là do các class CSS sẽ được tự xử lí thành những đoạn mã hashing, mà lúc đấy thì debug nó thuộc component nào chỉ có bằng nhang khói. Cho nên, họ mới đề xuất mình lắp cái tên component gốc vào trước classname để dễ debug trên Dev. 


# 🌿 Tích hợp Antd Pro

Ant Design có một bộ components nữa gọi là Pro Components. Các components này được xây dựng theo "convention" cho 1 resource CRUD bất kì. Các bác có thể tham khảo template chung của 1 app rdùng Ant Design Pro: [Preview](https://preview.pro.ant.design/)

Và để bắt đầu pro hơn, các bác gõ lệnh sau:

```
yarn add @ant-design/pro-form @ant-design/pro-layout \
@ant-design/pro-list @ant-design/pro-skeleton \
@ant-design/pro-table @ant-design/pro-provider
```

## 🆘 Ét ô ét: Cannot use import statement outside a module

![](Screenshot 1.png)

Ngay khi các bác có ý định `import {LoginForm} from '@ant-design/pro-form'` thì lỗi trên sẽ xuất hiện. Vấn đề nó xảy ra ở trong thư viện, họ require một file CSS khi import component này. Mà như các bác đã biết, Next.js không cho phép import 1 file CSS trong component mà phải import từ tầng cao nhất (_app)

![](Screen Shot 2022-03-26 at 17.15.53.png)

### Hành trình đi tìm cách workaround 🙂

Đầu tiên cháu thử sửa kiểu thêm dynamic, vì nó có thể chặn Next render component này ở Server Side

![](container.png)

OK, lên. Nhưng vấn đề là cháu import được ProForm, mà cái cháu đang cần import là LoginForm cơ.

![](Screen Shot 2022-03-25 at 21.33.10.png)

Export kiểu này ai chơi. `export default` ở đây là `ProForm`, và nó cũng được export cùng với những bọn còn lại (named export). Hay thử import thẳng từ code như dưới nhỉ?

![](Screen Shot 2022-03-25 at 21.34.11.png)

![](Screen Shot 2022-03-25 at 21.34.18.png)

Họ lại dùng LoadableComponent làm vỏ (để Lazy Load) khi export component này, dẫn đến import thẳng từ code sẽ không thể sử dụng được

✅ Thật ra cách import đúng sẽ như thế này

![](Screen Shot 2022-03-26 at 17.37.34.png)

```
const LoginForm = dynamic(
  () => import("@ant-design/pro-form").then((module: any) => module.LoginForm),
  {
    ssr: false,
  }
);
```

Sau khi import các bác cần chỉ định module sẽ return trong trường hợp có nhiều đối tượng cùng export. Và kết quả thành công

![](IMG_7E2508E7BF1C-1.jpeg)

Các bác sẽ phải viết như vậy với từng component, kể cả các component nếu họ viết dạng sub module như `ProText.Password`

![](Screen Shot 2022-03-26 at 17.43.05.png)

Thật ra cháu còn một vấn đề nhức nhối nữa là locale chưa thể set được, dẫn đến các nút submit mà họ dùng i18n đều là tiếng trung. Sau một hồi nghiên cứu cách họ export Context, Provider cháu quyết định:

![](Screen Shot 2022-03-27 at 10.23.37.png)

