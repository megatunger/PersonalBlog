---
layout: post
title:  "Code giỏ hàng với Context API + Hook"
date:   2021-01-13 07:00:00 +0700
categories: flutter, javascript, webview
---

### Context API + Hook + Typescript

Context API, Hook thì quá nổi tiếng rồi, chắc trên mạng đầy bài giới thiệu rồi nên cháu lười viết lắm =)). Mặc dù cháu thấy có rất nhiều bài hướng dẫn trên mạng nhưng lúc cháu tìm trên Google tuyệt nhiên không có bài nào viết CartContext bằng Typescript. Mà tính cháu do quen ở công ty cũ, viết JS mà không có Typescript thấy nó ngu người lắm nên tính lên bài này cho các bác nào cần.

Dân chơi đọc Cart phát biết làm gì luôn đúng không các bác, một cái giỏ hàng có thêm sửa xoá sản phẩm (lineItems) trong đấy, thêm linh tinh mấy hàm như tính tổng.

### Bắt đầu nào

Đầu tiên, các bác hình dung sơ qua những hàm sẽ có để viết interface `CartContextState`

```jsx
interface CartContextState {
  lineItems: ILineItem[]; // Tuỳ các bác thiết kế thế nào nha
  addToCart: (item: IProduct) => void;
  removeFromCart: (item: IProduct) => void;
  cartSum: () => number;
  clearCart: () => void;
  cartCount: () => number;
}
```

Sau đó tạo context `CartContext` sử dụng hàm `React.createContext`:

```jsx
const CartContext = React.createContext<CartContextState | undefined>(
  undefined,
);
```

và tạo `hook` cho cái context này, đây sẽ là nơi các bác lấy dữ liệu từ context

```jsx
export function useCart(): CartContextState {
  const context = useContext<any>(CartContext);

  if (!context) {
    console.log('Chưa bọc App vào trong cái CartProvider thì chưa dùng được nha..');
  }
  return context;
}
```

sau đó giờ các bác phải viết Provider để bọc cái App vào trong, lúc đó dữ liệu trong context mới được chia sẻ giữa các components con của nó

```jsx

interface CartProviderProps {
  children: React.ReactNode;
}

export const CartProvider: React.FunctionComponent<CartProviderProps> = ({
  children,
}: CartProviderProps) => {
  const [lineItems, setLineItems] = useState<ILineItem[]>([]);
  
  return (
    <CartContext.Provider
      value={{
        lineItems,
        addToCart, // Cần Implement
        cartSum, // Cần Implement
        clearCart, // Cần Implement
        cartCount, // Cần Implement
        removeFromCart, // Cần Implement
      }}>
      {children}
    </CartContext.Provider>
  );
};

```

Ví dụ cháu sẽ cài đặt các hàm trên như sau

```jsx
  const addToCart = (item: IProduct) => {
    if (lineItems) {
      let newLineItems = [];
      let oldData = lineItems;
      let check = oldData.findIndex((_item) => {
        return item.id === _item.product.id;
      });
      if (check === -1) {
        const newData: ILineItem = {
          product: item,
          quantity: 1,
          timestamp: Date.now(),
        };
        newLineItems = [...lineItems, newData];
      } else {
        newLineItems = [
          ...oldData.filter((_i) => _i.product.id !== item.id),
          {
            product: oldData[check].product,
            quantity: (oldData[check].quantity += 1),
            timestamp: oldData[check].timestamp,
          },
        ];
      }
      setLineItems(
        newLineItems
          .sort(function (a, b) {
            return a.timestamp - b.timestamp;
          })
          .reverse(),
      );
    }
  };

  const clearCart = () => {
    setLineItems([]);
  };

  const cartSum = () => {
    let sum = 0;
    lineItems?.forEach((item) => (sum += item.quantity * item.product.price));
    return sum;
  };

  const cartCount = () => {
    let sum = 0;
    lineItems?.forEach((item) => (sum += item.quantity));
    return sum;
  };
```

Xong rồi ở file App các bác bọc nó kiểu kiểu thế này ha

```jsx
<CartProvider>
  <App />
</CartProvider>
```

Lúc đó tha hồ ở trong các nút con của App, muốn lấy lineItems chỉ việc gọi

```jsx
const { lineItems } = useCart();
```

### Full Code không che:

```jsx
interface CartProviderProps {
  children: React.ReactNode;
}

export interface ILineItem {
  product: IProduct;
  quantity: number;
  timestamp: number;
}

interface CartContextState {
  lineItems?: ILineItem[];
  addToCart: (item: IProduct) => void;
  removeFromCart: (item: IProduct) => void;
  cartSum: () => number;
  clearCart: () => void;
  cartCount: () => void;
}

const CartContext = React.createContext<CartContextState | undefined>(
  undefined,
);

export const CartProvider: React.FunctionComponent<CartProviderProps> = ({
  children,
}: CartProviderProps) => {
  const [lineItems, setLineItems] = useState<ILineItem[]>([]);

  const addToCart = (item: IProduct) => {
    if (lineItems) {
      let newLineItems = [];
      let oldData = lineItems;
      let check = oldData.findIndex((_item) => {
        return item.id === _item.product.id;
      });
      if (check === -1) {
        const newData: ILineItem = {
          product: item,
          quantity: 1,
          timestamp: Date.now(),
        };
        newLineItems = [...lineItems, newData];
      } else {
        newLineItems = [
          ...oldData.filter((_i) => _i.product.id !== item.id),
          {
            product: oldData[check].product,
            quantity: (oldData[check].quantity += 1),
            timestamp: oldData[check].timestamp,
          },
        ];
      }
      setLineItems(
        newLineItems
          .sort(function (a, b) {
            return a.timestamp - b.timestamp;
          })
          .reverse(),
      );
    }
  };

  const removeFromCart = (item: IProduct) => {
    setLineItems(lineItems.filter((_item) => _item.product.id !== item.id));
  };

  const clearCart = () => {
    setLineItems([]);
  };

  const cartSum = () => {
    let sum = 0;
    lineItems?.forEach((item) => (sum += item.quantity * item.product.price));
    return sum;
  };

  const cartCount = () => {
    let sum = 0;
    lineItems?.forEach((item) => (sum += item.quantity));
    return sum;
  };

  return (
    <CartContext.Provider
      value={{
        lineItems,
        addToCart,
        cartSum,
        clearCart,
        cartCount,
        removeFromCart,
      }}>
      {children}
    </CartContext.Provider>
  );
};

export function useCart(): CartContextState {
  const context = useContext<any>(CartContext);

  if (!context) {
    console.log('useCart not wrapped');
  }
  return context;
}
```